---
title: 성능테스트란 무엇이며 어떻게 할까요?
subTitle: Artillery를 사용한 부하 테스트 진행하기
tech: NodeJS
category: 부하테스트
tags:
  - 부하테스트
  - Artillery
date: 2023-11-30
---


기존 서비스에서는 개인화 메시지라는 기능을 제공하고 있었습니다.
주문, 배송, 회선, 고객 정보를 활용해서 회원에게 맞는 메시지를 생성해 주는 API 인데요.
거의 모든 도메인 서비스를 전부 조회하기 때문에 부하가 심했고
트래픽이 증가하면 서비스가 다운되는 상황이 발생했어요. 

저희는 원인이 되는 로직을 비활성화하고 해당 서비스로직 개선을 진행했고
개선 후 운영배포에 앞서 성능을 측정하고자 성능 테스트를 도입하게 되면서
공부한 내용에 대해 간단히 공유해 드리고자 합니다.




# 성능 테스트란?
---
성능 테스트란 서비스의 속도, 반응성, 안정성을 측정하고 평가하는 과정입니다.
시스템의 어떤 측면을 테스트하느냐에 따라 다양한 부하 조건에서 테스트를 진행하게 되는데요.
많은 테스트 유형이 있지만 주로 사용되는 성능 테스트 유형에 대해 알아보겠습니다.


## 성능 테스트 유형
![image](https://github.com/HotteokGroup/PPOTTO_USER/assets/55491354/db3e7d08-2be8-4648-be4b-f5a86fdfa2ce)
[출처](https://medium.com/singapore-gds/web-performance-testing-dcubes-practices-fbbc20606000)

### Load Test
정해진 부하를 처리할 수 있는지 검증하는 테스트입니다.
- 부하 수준 : 피크 시간대 유저 수
- 부하 지속 시간 : 통상 한 시간


### Stress Test
매우 높은 부하를 처리할 수 있는지 검증하는 테스트입니다.
시스템을 사용할 수 있는지 부하가 줄었을 때 정상적으로 시스템이 복구되는지 확인합니다.
- 부하 수준 : 피크 시간대 유저 수의 두 배
- 부하 지속 시간 : 통상 한 시간

### Endurance Test
장시간에 걸쳐 시스템 안정성을 검증하는 테스트입니다.
- 부하 수준 : 통상 시간대 유저 수
- 부하 지속 시간 : 8시간 이상

### Spike Test
사용량이 급격히 증가하는 시나리오를 상정하고 시스템의 안정성을 확인하는 테스트입니다.
- 부하 수준 : 통상 시간대 유저 수 ~ 피크 시간대 유저 수의 두배
- 부하 지속 시간 : 통상 한 시간

### Breakpoint Test
동시 사용자 수를 점진적으로 증가시켜 시스템 장애 지점을 확인하는 테스트입니다.
- 부하 수준 : 부하가 발생할 때까지 점진적으로 증가
- 부하 지속 시간 : 장애 발생 지점 까지

## 부하 수준을 설정할 때 고려해야 하는 것
서비스의 성장률과 중요도에 따라 부하 수준을 다르게 설정하는 것이 좋습니다.

- 성장률 : 예상 성장률을 부하 수준에 반영
- 중요도 : 중요도가 높은 서비스라면 더 높은 부하 수준 반영


## 테스트 통과 기준을 정할 때 고려해야 하는 것
- 중요도가 높은 서비스인 경우 응답 성공률도 기준에 포함하는 것이 좋습니다.
- 응답 여부가 통과 기준이 아닌 응답 시간을 염두 해야합니다.
> 웹서비스가 1초 느려지면 매출이 3% 떨어진다는 통계가 있습니다.



# Artillery로 성능 테스트하기
---
> [공식문서 링크](https://www.artillery.io/docs/get-started/get-artillery)

Artillery는 HTTP, [Socket.io](http://socket.io/), WebSockets, gRPC, Kafka등 다양한 프로토콜을 지원하며 플러그인을 통해 모든 종류의 시스템을 테스트할 수 있도록 설계된 부하테스트 도구입니다.

## 테스트 시나리오 구성하기
저희는 먼저 개선한 기능에 대해 얼마만큼 사용자를 감당할 수 있는지 확인하기 위해
Breakpoint Test를 진행하기로 했고 API요청 시나리오는 다음과 같습니다.

1. 서비스에 로그인합니다.
2. 개인화 메시지 기능을 호출합니다.

## 테스트 스크립트 작성하기
시나리오 요구사항을 반영한 스크립트는 다음과 같습니다.
```yaml
config:
  target: "타깃 주소"
  phases:
    - duration: 30m # 30분간 테스트
      arrivalRate: 1 # 시작할 유저 수
      rampTo: 100 # 30분 동안 늘릴 최대 사용자 수, 마지막엔 초당 100명의 사용자가 요청
      name: break point

before:
  flow:
    - log: "Get Auth Token"
    - post:
        url: "/auth/login/email"
        json:
          email: "test@test.com"
          password: "1234qwer"
          isAppLogin: false
        capture:
          json: "$.accessToken"
          as: "accessToken"
    - log: "Get Auth Token: {{ accessToken }}"

scenarios:
  - name: "개인화 메시지 테스트"
    flow:
      - get:
          url: "/mvno/personalMessage"
          cookie:
            PDS_ACTK: "{{ accessToken }}"
```
테스트 파일은 기본적으로 `config` `before` `after` `scenarios` 로 구성되어 있습니다.
- `config` : 테스트 대상 호스트 설정 및 테스트 요청을 어떤 식으로 보낼지 정의합니다.
- `before` / `after` : `scenarios`가 호출되기 전, 후에 한 번 실행되는 섹션입니다.
- `scenarios` : 어떤 방식으로 요청할지 시퀀스를 정의합니다.

> 구문에 대한 설명은 [공식문서](https://www.artillery.io/docs/reference/test-script) 를 참고해 주세요.


## 테스트 시작하기
```bash
artillery run get-personal-message.artillery.yaml -o ./result.json
```

다음 명령어를 통해 테스트를 시작할 수 있습니다.

- `run <fileName>` : 테스트를 실행합니다. 
- `-o <fileName>` : 결과를 저장합니다.


## 테스트 리포트 생성
![image](https://github.com/HotteokGroup/PPOTTO_SOCIAL/assets/55491354/9b1f29a9-d215-45e0-baf2-d508442c1ddd)
```yaml
artillery report result.json
```

다음 명령어를 통해 결과를 시각적으로도 확인 할 수 있는 옵션을 제공했습니다.
`Breakpoint Test` 특성상 요청 수가 선형적으로 증가한 것을 확인할 수 있었습니다.



# Reference
---
https://medium.com/singapore-gds/web-performance-testing-dcubes-practices-fbbc20606000

https://www.artillery.io