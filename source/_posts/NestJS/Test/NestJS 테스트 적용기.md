---
title: 테스트란 무엇이며 어떻게 할까요?
subTitle: NestJS에서 Jest로 테스트코드 작성하기
tech: NestJS
category: 테스트
tags:
  - 테스트
  - TDD
  - Jest
date: 2023-12-01
---

저희 서비스는 NestJS 기반으로 다양한 도메인 서비스와 배치작업을 처리하고 있습니다.
계속 다양한 요구사항에 대응하고 많은 팀원이 코드를 수정하면서 휴먼 에러가 늘어나
테스트 코드의 중요성이 커지게 되었습니다.

오늘은 서비스에 유닛테스트를 도입하게 된 계기와
도입하면서 공부한 내용에 대해 공유하고자 합니다.


# 테스트 유형에 대해
---

서비스 레이어가 Controller, Service, Repository로 나뉘어 있는 애플리케이션은
주로 테스트 범위에 따라 3가지 유형이 존재합니다.
- Unit Test (단위 테스트)
	- Service내에 정의된 메소드, 클래스의 단위 기능을 테스트합니다.
	- 개별 컴포넌트의 동작을 테스트하기 위해 의존성을 격리합니다. (mock)
- Integration Test (통합 테스트)
	- 컴포넌트 간 다양한 상호작용을 테스트합니다.
	- Controller <-> Service, Service <-> Repository (실제 DB 또는 가상 DB 사용)
- E2E Test (종단 테스트)
	- 실제 사용자 관점에서 전체 시스템(UI 포함)이 예상대로 동작하는지 확인합니다. 


# 유닛테스트 도입 배경
---
테스트 코드 없이 많은 사람이 로직을 작성하고 유지보수 할 때 제가 겪은 문제는 다음과 같았습니다.
- 히스토리 부재
	- 요구사항이 변화할 경우 코드가 기존 의도와 달라져 사이드 이펙트가 발생했어요.
- 신뢰성 이슈
	- 코드리뷰와, 개발자의 개인적인 테스트에만 의존했어요.
- 갓 객체의 탄생
	- 동작하는 기능을 빠르게 제공하는 것에만 초점을 두어 하나의 객체가 너무 많은 일을 하게 되었어요.

이러한 기술부채에 맞서기 위해 **유닛 테스트**를 도입하고자 했습니다.


# 유닛테스트를 작성하기 앞서
---
유닛 테스트의 핵심은 빠르게 하나의 기능(메소드)을 **의존성을 격리**해서 테스트하는 것인데요.
본격적으로 유닛테스트를 작성하기 전 다음 두 개의 질문에 부딪히게 됩니다.

- 어떻게 의존성을 격리할 것인지
- 어디까지 격리할 것인지


## 어떻게 격리할까요?
유닛테스트에서는 테스트 더블(Test Double)을 통해
의존성 객체들을 대체함으로써 의존성을 격리하게 됩니다.

테스트 더블(Test Double)이란 기능을 동작하는데 필요한 의존성 객체 대신
테스트에 사용되는 모든 방법을 말합니다.

- Dummy
	- 단순 인스턴스 객체를 리턴하는 객체를 만듭니다.
	- 정상 동작을 보장하지 않습니다.
- Stub
	- 호출 시 사전에 준비된 결과를 리턴합니다.
- Mock
	- 호출 시 기능명세에 따라 완벽히 동작하는 객체를 만들어 결과를 리턴합니다.
- Fake
	- 동작은 하지만 실제 객체처럼 정교하게 동작하지 않는 객체를 만들어 결과를 리턴합니다.
	- *ex) Fake DB*
- Spy
	- Stub 역할을 하면서 호출되었을 때 상태를 저장하는 기능을 가지는 객체를 만들어 결과를 리턴합니다.


## 어디까지 격리할까요?
위에서 테스트 더블을 통해 의존성을 대체하는 방법 대해 알아보았는데요.
그럼 어느 정도까지 의존성을 격리하는 게 좋을까요?
이에 대해서는 두 가지 견해가 있습니다.

### 고전파
하나의 테스트를 기준으로 격리합니다.
테스트 간에 공유되고 결과에 영향을 미칠 수 있는 객체만 테스트 더블 합니다. *ex) DB*

### 런던파
테스트의 의존성(협력자)을 격리합니다.
하나의 테스트가 의존하는 모든 객체를 테스트 더블합니다.

의존성을 테스트 더블 할수록 테스트 단위가 작아진다는 장점이 있지만
의존 모듈을 전부 테스트 더블하기 때문에 테스트 코드가 복잡해지고
리팩토링 내성을 잃게 되는 단점이 있었습니다.


# NestJS에서 Jest로 유닛 테스트 작성하기
---
저희는 먼저 기존에 테스트 코드가 없지만, 중요도가 높은 로직에 테스트 코드를 먼저 적용했습니다.
대표적으로 저희 서비스에서 SQS 메시지를 가져오는 부분을 간단하게 아래에 예시로 작성해 보았습니다.
해당 로직과 NestJS [Jest 문서](https://docs.nestjs.com/fundamentals/testing) 기반으로 테스트 코드를 작성해 보겠습니다.
```typescript
@Cron('*/15 * * * * *', { timeZone: 'Asia/Seoul' })

async batch() {
	
	// 9시 ~ 18시에만 실행
	if (!this.taskService.isWorkTime()) return { success: true, batchMessage: '영업시간아님' };
	
	// SQS에서 메시지를 가져온다.
	const messages: JuiceEvent[] = await this.sqsProvider.getJuiceMessageAndDelete(this.queryUrl);
	if (messages.length === 0) return { success: true, batchMessage: '메시지없음' };
	
	  
	
	const failedMessages: JuiceEvent[] = [];
	for await (const message of messages) {
		const { actCode, contractNum, provider } = message;
		try {
			switch (actCode) {
			case 'NAC': {
				await this.taskService.registerLine(provider, contractNum);
				break;
			}
			case 'CCN': {
				await this.taskService.changeLine(provider, contractNum);
				break;
			}
			default: {
				Logger.error(`[${actCode}][${contractNum}] 알 수 없는 actCode 입니다.`, this.LOG_CONTEXT);
				throw new Error(`알 수 없는 actCode 입니다. [${actCode}]`);
			}
		}
		} catch (error) {
			failedMessages.push(message);
		}
	}
	
	return { success: true, batchMessage: '작업 종료', failedMessages };

}
```
해당 로직은 SQS에서 메시지를 가져오고 메시지의 이벤트 코드를 보고
적절한 비즈니스 로직에 메시지를 전달하는 책임을 가지고 있는 객체입니다.
자세한 요구사항을 항목으로 정리하면 다음과 같습니다.

- 9시 ~ 18시에만 실행됩니다.
- 각 `actCode`에 맞는 비즈니스 로직을 호출합니다.
- 처리에 실패한 메시지는 작업 결과와 함께 반환되어야 합니다.


## 테스트 코드 작성하기

### 클래스 의존성 테스트 더블하기
```typescript
describe('TestBatch', () => {
	let testBatch: TestBatch;
	const taskServiceMock = { registerLine: jest.fn(), changeLine: jest.fn() };
	const SqsV3ProviderMock = { getJuiceMessageAndDelete: jest.fn() };
	
	beforeEach(async () => {
		const moduleRef = await Test.createTestingModule({
		providers: [
			TestBatch,
			{ provide: SqsV3Provider, useValue: SqsV3ProviderMock },
			{ provide: TaskService, useValue: TaskServiceMock },
		],
	}).compile();
	
	
	testBatch = moduleRef.get<TestBatch>(TestBatch);
});

```
`TestBatch` 클래스에서 사용하는 `SqsV3Provider` `TaskService` 메소드를 Jest Mocking 객체로 대체해 주었습니다.


## 영업시간 체크로직 테스트하기
```typescript
it('영업 시간(9시 ~ 18시)이 아니면 프로세스를 종료합니다.', async () => {
	// given
	// 영업 시간이 아니라고 가정
	const fakeNowTime = DateTime.fromISO('2023-01-01T08:00:00Z'); // utc 기준
	jest.useFakeTimers();
	jest.setSystemTime(fakeNowTime.toJSDate());
	
	// when
	const result = await testBatch.batch();
	
	// then
	expect(result).toEqual({ success: true, batchMessage: '영업시간아님' });
});
```
영업시간이 아닐 경우 종료 처리되는 결과를 테스트하기 위한 코드를 작성해주었습니다.

시간에 영향을 받는 로직의 경우 Jest의 `setSystemTime`을 사용하면 의도한 시간대로 시간을 고정할 수 있었어요.


## SQS 메시지 조회 테스트하기
```typescript
it('SQS에서 메시지를 가져오지 못하면 프로세스를 종료합니다.', async () => {
	// given
	SqsV3ProviderMock.getJuiceMessageAndDelete.mockResolvedValue([]);
	
	// when
	const result = await testBatch.batch();
	
	// then
	expect(result).toEqual({ success: true, batchMessage: '메시지없음' });
});
```
다음으로 처리할 메시지가 없으면 로직을 종료해야 하는 요구사항을 테스트하기 위한 코드를 작성했습니다.

메시지를 가져오는 메소드를 `mockResolvedValue` 를 사용해서 `stub` 해주었습니다.


## 처리 불가능한 이벤트 처리 테스트하기
```typescript
it('SQS에서 처리 불가능한 메시지가 있을 경우 failedMessages에 담아서 리턴합니다.', async () => {
	// given
	// 처리 불가능한 임의의 actCode를 가진 메시지
	const failedMessages = [
	{ actCode: 'TEST1', contractNum: '123', provider: 'SKT' },
	{ actCode: 'TEST2', contractNum: '123', provider: 'SKT' },
	];
SqsV3ProviderMock.getJuiceMessageAndDelete.mockResolvedValue(failedMessages);
	
	// when
	const result = await testBatch.batch();
	
	// then
	expect(result).toEqual({ success: true, batchMessage: '작업 종료', failedMessages });
});
```
처리 불가능한 `actCode`가 존재할 경우 작업 종료할 때 해당 메시지들을
`failedMessages`에 담아 리턴하는 요구사항을 테스트하기 위한 코드를 작성했습니다.

## 올바른 로직 호출여부 테스트하기
```typescript
it('NAC 이벤트를 받으면 registerLine(회선 등록)을 호출합니다.', async () => {
	// given
	const messages = [{ actCode: 'NAC', contractNum: '123', provider: 'SKT' }];
	SqsV3ProviderMock.getJuiceMessageAndDelete.mockResolvedValue(messages);

	// when
	const result = await testBatch.batch();

	// then
	expect(result).toEqual({ success: true, batchMessage: '작업 종료', failedMessages: [] });
	expect(taskServiceMock.registerLine).toBeCalledTimes(1);
	expect(taskServiceMock.registerLine).toBeCalledWith('SKT', '123');
});

```
각 `actCode` 이벤트별로 올바른 로직을 호출했는지 테스트하기 위한 코드를 작성했습니다.
해당 비즈니스 로직에서는 어떤 행위를 했는지가 중요하므로 다음과 같이 진행했습니다.

대부분의 경우 로직의 행위보다는 결과에 집중해야 리팩토링 내성을 잃지 않을 수 있음을 명심하고 진행해야 합니다!




# 마치며
---
저희 서비스에서 테스트 코드를 도입하는 과정에서 공부한 내용과 적용과정을 간단하게 정리했습니다!

서비스에 유닛테스트를 적용하면서 생각한 **장점**은 다음과 같았어요

-  객체의 역할과 책임에 집중할 수 있습니다.
	- 객체가 너무 많은 책임을 지고 있는 것이 아닌지 생각해 볼 수 있었어요.
- 코드에 드러나지 않은 요구사항을 명시합니다.
	- 로직적으로 풀어낸 요구사항을 테스트 케이스를 통해 의도한 바를 명확하게 전달할 수 있었어요.
- 코드의 신뢰성을 확보할 수 있습니다.
	- 테스트하기 어려운 환경을 재현하여 테스트할 수 있었어요. *ex) 시간*


제가 느낀 유닛테스트의 **단점**은 다음과 같았어요

- 작업량이 늘어납니다.
- 테스트 코드가 있는 코드는 테스트만 통과하면 코드를 무조건 신뢰하는 경향이 있습니다.
	- 테스트 코드는 완벽하지 않으며 꾸준히 반례를 추가해 나가야 합니다.
- 거의 모든 비즈니스 로직은 공유 의존성을 가지기 때문에 테스트하기 어렵습니다.
	- Repository 계층의 상호작용을 테스트할 경우 Fake DB(In-memory Database)가 필요합니다.
	- 행동 기반(호출 횟수, 매개변수 검증)으로 테스트할 경우 리팩토링 내성을 잃습니다. 




# Reference
---
https://tecoble.techcourse.co.kr/post/2020-09-19-what-is-test-double/
https://martinfowler.com/articles/mocksArentStubs.html
https://jestjs.io/docs/api#beforeallfn-timeout
https://docs.nestjs.com/fundamentals/testing