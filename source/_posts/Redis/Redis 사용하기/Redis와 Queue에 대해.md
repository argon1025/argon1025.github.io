---
title: Redis와 Queue에 대해
subTitle: Redis로 NestJS에서 Queue 구현하기
tech: Redis
category: Redis 사용하기
tags:
	- Redis
	- NestJS
	- Queue
date: 2022-01-05
---

기존 프로젝트에서 중요한 에러 알림은 슬랙으로 모니터링 중이었습니다
Slack WebHook lateLimit의 영향으로 초당 1개 이상의 슬랙 메시지가 봇을 통해 발생할 경우
에러 메시지 수신이 일정 시간 동안 중단되는 상황이 발생하게 되는데요

이번 포스트를 통해 큐란 무엇인가에 대해 간단히 정리하고
슬랙의 요청 제한에 대응하는 메시지 큐 서비스를 구현해 보겠습니다

---

# 메시지 큐 종류

메시지 큐는 대표적으로 `Kafka` `RabbitMQ` `Amazon MQ` 등이 있습니다
Redis는 기본적으로 in-memory 스토리지지만
Stream이나 pub/sub 컬렉션을 지원하기에 레디스에서도 메시지 큐를 사용할 수 있습니다
프로젝트에서 메시지큐로 레디스를 사용하기로 했고 이유는 다음과 같습니다

- 이미 레디스를 프로젝트에서 사용하고 있습니다
- 작은 프로젝트이기 때문에 추가로 다른 인스턴스를 할당하기가 어렵습니다
- 메시지의 크기가 작고, 메시지를 저장해서 재사용할 필요가 없습니다
- FIFO 구조의 작업 큐 형태만 필요합니다

> Kafka vs RabbitMQ 비교 문서 [https://blog.logrocket.com/kafka-vs-rabbitmq-comparing-node-js-message-brokers/](https://blog.logrocket.com/kafka-vs-rabbitmq-comparing-node-js-message-brokers/)

> Kafka vs Redis 비교문서 [https://logz.io/blog/kafka-vs-redis/](https://logz.io/blog/kafka-vs-redis/)

---

# Redis Bull

Bull은 Redis 기반 큐 시스템을 빠르게 사용할 수 있도록 만들어진 노드 라이브러리입니다.
그냥 Redis command를 통해서도 구현할 수 있지만
Bull을 사용할 경우 Redis command를 직접적으로 다루지 않고도
쉽게 처리할 수 있도록 간략화된 API를 제공합니다

## WorkFlow

![image](https://user-images.githubusercontent.com/55491354/193415820-835ca506-91bb-414a-9db4-f46d89f86273.png)

Bull을 사용할 경우 다음과 같이 작업이 추가되고 처리됩니다

---

# NestJS에서 구현하기

### AppModule 모듈 등록

```typescript
BullModule.forRootAsync({
      imports: [ConfigModule],
      useFactory: async (configService: ConfigService) => {
        const REDIS_HOST = configService.get<string>('REDIS_HOST', 'localhost');
        const REDIS_PORT = configService.get<number>('REDIS_PORT', 6379);
        return { redis: { host: REDIS_HOST, port: REDIS_PORT } };
      },
      inject: [ConfigService],
    }),
```

설정 옵션은 `interface QueueOptions` 인터페이스를 참조하세요

[Bull Docs](https://github.com/OptimalBits/bull/blob/master/REFERENCE.md#queue)

### task-manager.module 작업 큐 생성하기

```typescript
@Module({
  imports: [
    BullModule.registerQueue({
      name: 'log',
      limiter: {
        max: 1,
        duration: 1000,
      },
    }),
    HttpModule,
    ConfigModule,
  ],
  providers: [TaskManagerService, LogConsumer],
  exports: [TaskManagerService],
})
```

log라는 하나의 큰 작업 대기열을 생성하고 `limiter` 옵션을 적용했습니다
해당 옵션은 전역설정이기 때문에 log 대기열에 대해 10명의 소비자가 있더라도
1초에 1개의 작업만 처리됩니다

> [대기열 설정 https://docs.bullmq.io/guide/rate-limiting](https://docs.bullmq.io/guide/rate-limiting)

그리고 다른 서비스에서 해당 작업 큐를 사용하기 위해 밑에서 생성한
TaskManagerService, LogConsumer를 프로바이더로 등록했습니다

### task-manager.service Producer 작업 생산자 생성

대기열에 작업을 추가하는 서비스를 구현합니다

```jsx
constructor(@InjectQueue('log') private readonly loggingQueue: Queue) {}

  /**
   * 에러 로그 전송작업을 생성합니다
   * @param requestData
   * @returns
   */
  public async sendError(requestData: { location: string; developerComment: string; errorCode: number; errorObjectCode: string; message: object }) {
    const jobInfo = await this.loggingQueue.add(
      'send',
      { data: requestData },
      {
        timeout: 3000,
        removeOnComplete: true,
        removeOnFail: 30,
        attempts: 5,
        backoff: { type: 'fixed', delay: 3000 }, // 3s
      },
    );
    return jobInfo.id;
  }
```

log 작업 대기열에 send 작업을 추가합니다
작업을 추가할 때 작업 큐 생성 시 설정한 데이터를 오버라이드 할 수 있습니다
작업이 성공하면 스토리지에서 삭제, 실패할 경우 최근 30개는 삭제하지 않도록 했고
실패했을 때 3초 간격으로 5회 다시 시도하도록 설정했습니다

> 재시도 설정 [https://docs.bullmq.io/guide/retrying-failing-jobs](https://docs.bullmq.io/guide/retrying-failing-jobs)

### log.processor 소비자 생성

작업을 처리하는 서비스를 구현합니다

```jsx
@Processor('log')
export class LogConsumer {
  constructor(private httpService: HttpService, private configService: ConfigService) {}

  @Process('send')
  async sendError(job) {
    // 작업 데이터 로드
    const ERROR_DATA = job?.data?.data;
    const MESSAGE: object = {
      blocks: [
        {
          type: 'header',
          text: {
            type: 'plain_text',
            text: '에러 이벤트 발생',
            emoji: true,
          },
        },
        {
...;
    const WEB_HOOK_URL: string | undefined = this.configService.get<string>('ERROR_SLACK_WEBHOOK_URL', undefined);

    try {
      if (!!WEB_HOOK_URL) {
        await firstValueFrom(this.httpService.post(WEB_HOOK_URL, MESSAGE));
      }
      return 'ok';
    } catch (error) {
      throw new Error('log.send job fail');
    }
  }
}
```

log 작업 대기열의 send 작업을 처리하는 소비자를 생성했습니다
인수로 전달받는 job 객체는 작업 생산자가 전달한 데이터와 작업의 상태를 관리하는 여러 메서드가 있습니다

### ExceptionFilter에 DI하기

```jsx
// ExceptionFilter
const taskManagerService = app.get < TaskManagerService > TaskManagerService;
app.useGlobalFilters(new HttpExceptionFilter(taskManagerService));
```

이전 에러 처리 리팩터링 때 추가한 개발자 코멘트 객체를 웹훅에 전달하기위해서
글로벌 익셉션 필터에서 taskManagerService(작업 생산자 서비스)를 DI 했습니다.

### ExceptionFilter에서 작업 생성하기

```typescript
export class HttpExceptionFilter implements ExceptionFilter {
  constructor(private taskManagerService: TaskManagerService) {}
...
// Task-Manager-Service
    // 전체 에러에 대한 로깅
    this.taskManagerService.sendError({
      location: requestLocation,
      developerComment: devDescription,
      errorCode: status,
      errorObjectCode: errorObjectCode,
      message: message,
    });
```

ExceptionFilter에서 오류가 생성될 때 마다 log 의 send 작업을 생성하게 작성했습니다

---

# 결과

![image](https://user-images.githubusercontent.com/55491354/193415934-b67e0237-1b18-4c3e-8e10-9c9f78d87ae1.png)

급격한 로깅 증가에도 `Slack WebHook lateLimit`에 맞춰 메시지를 받을 수 있게되었습니다
