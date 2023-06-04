---
title: AWS ECS로 서비스 배포하기
subTitle: AWS 컨테이너 오케스트레이션 서비스 알아보기
tech: AWS
category: AWS ECS 배포기
tags: 
	- AWS
	- AWS ECS
date: 2022-01-13
---

Docker-compose로 환경을 구성 및 배포해오다
처음 컨테이너 오케스트레이션 서비스로 배포하기 위해
공부한 내용을 정리하고자 합니다.

---

# 컨테이너 오케스트레이션은 어떤 문제를 해결하는가

## 컨테이너 오케스트레이션을 사용하기 전에는..

Docker-compose를 활용해서 개발환경을 쉽고 빠르게 설정할 수 있지만
서비스를 배포할 때 Docker-compose로 부하 분산 서비스를 구현하기 위해서
Docker-compose 내에서 리버스 프록시, 애플리케이션 컨테이너 설정을 모두 명시해야 했고
그로 인해 Docker-compose가 실행된 상태에서
업데이트, 확장, 장애 회복 등 모든 상황에 들어가서 직접 작업해야 했습니다
하지만 컨테이너 오케스트레이션을 사용함으로써 다음 영역을 자동화할 수 있었습니다

- 배포 자동화 (무중단 배포)
- 자동 로드밸런싱
  - 업, 다운 스케일링 대응 트래픽 라우팅
- 컨테이너 상태 모니터링 및 가용성 제공
  - 컨테이너 업/다운 스케일링
  - 컨테이너 스케일링 전략
- 인스턴스 자원관리, 리소스 할당 및 가용성 제공
  - 인스턴스의 업/다운 스케일링
  - 인스턴스 스케일링 전략
  - 컨테이너의 배치 전략

---

# 왜 ECS를 선택했는지

최근 가장 많이 사용되는 EKS를 적용하려 했지만
저렴한 인스턴스를 사용하지 못해 추가 비용이 발생할 여지가 있어 선택지에서 제외했습니다.

- 러닝 커브가 다른 컨테이너 오케스트레이션 툴에 비해 비교적 낮다
- 자체적인 클러스터 관리를 지원한다
- API 호출, aws cli 로 모든 관리를 할 수 있다
- 클라우드 와치를 통한 모니터링
- EC2 인스턴스를 통해 ECS를 사용할 경우 비용 절감 (Pay as you go)
- ECR을 통해 ECS를 구성하기에 추후에 EKS로 전환이 용이합니다

---

# AWS 컨테이너 오케스트레이션 서비스 구성

![image](https://user-images.githubusercontent.com/55491354/193438641-5919ada6-8ce5-439b-aa64-62cc22076e12.png)

## Image Registry

### ECR(Amazon Elastic Container Registry)

Docker Private Repository를 구축해 컨테이너 이미지를 저장, 공유, 배포할 수 있는 이미지 레지스트리 서비스입니다
S3 버킷에 이미지를 저장하고 IAM을 통해 이미지 Push Pull 인가를 관리합니다

## Management

### ECS(Amazon Elastic Container Service)

컨테이너 이미지의 손쉬운 배포, 관리, 스케일링을 지원하는 완전관리형 컨테이너 오케스트레이션 서비스

### EKS(Amazon Elastic Kubernetes Service)

AWS에서 오픈소스 프로젝트인 컨테이너 이미지 자동 배포, 스케일링을 제공하는 쿠버네티스를 사용할 수 있도록 하는 서비스

## Hosting

### Fargate

서버리스 컴퓨팅 엔진으로 EC2 인스턴스를 관리할 필요 없이 컨테이너를 실행하기 위해 ECS EKS에서 사용할 수 있는 기술

### EC2

AWS EC2 인스턴스를 이용해 컨테이너를 실행하는 기본적인 기술

---

# ECS 는 어떤식으로 구성되어 있는가?

ECS 서비스를 작은 개념부터 하나씩 살펴보겠습니다

## 클러스터 인스턴스 ( 컨테이너 인스턴스 )

![image](https://user-images.githubusercontent.com/55491354/193438721-a8a30c17-ea09-4dde-9a5d-9384821a0df0.png)

도커 컨테이너 구동을 위한 물리적 인스턴스를 말합니다.

## 클러스터

![image](https://user-images.githubusercontent.com/55491354/193438727-fbf8aca1-f165-4a67-b4fc-be730e528a6c.png)

클러스터 인스턴스의 묶음으로 논리적인 단위입니다

## 태스크

![image](https://user-images.githubusercontent.com/55491354/193438731-25ef1380-a75f-4316-ae63-665447dc82aa.png)

클러스터 위에서 구동되는 도커 컨테이너를 말합니다

## 태스크 디피니션

![](https://tilog-file-service-s3.s3.ap-northeast-2.amazonaws.com/20us8j71zc0rj2022-01-14%2014%3A41%3A43.png)

태스크에서 구동되는 도커 컨테이너의 설정 템플릿 입니다

## 서비스

![image](https://user-images.githubusercontent.com/55491354/193438739-35104417-131a-4dc3-9d32-88c617725aa4.png)

태스크(작업)를 관리하는 단위입니다

- 서비스에서 정의한 대로 태스크가 실행되는 상태를 유지합니다
- 서비스에서 정의한 대로 서비스를 스케일 인 아웃 합니다
- 클러스터 인스턴스의 ASG 정책을 ECS Capacity Provider를 통해 관리하여 인스턴스를 스케일 인 아웃 합니다
  - 이후 새로 시작하는 태스크가 있다면 ECS가 태스크가 적은 인스턴스에 태스크를 분배합니다
- 스케일 인 아웃에 맞춰 ALB 설정을 최신화합니다

## ECS 오토스케일링

ECS에서 하는 오토 스케일링은 두 가지로 나눌 수 있습니다

### 서비스 오토 스케일링

ECS는 **Scaling policy** 설정을 통해 서비스를 스케일 인 아웃(태스크 수 증가 및 감소)하게 설정할 수 있습니다.
이때 증가 및 감소한 태스크와의 연결은 ALB가 담당합니다

### 클러스터 오토 스케일링

ECS는 **Capacity Provider**를 통해 클러스터 인스턴스의 스케일 인 아웃(인스턴스 추가 삭제)하게 설정할 수 있습니다
EC2 인스턴스를 사용할 경우에만 적용됩니다

---

# 직접 ECS 생성 하고 설정하기

ECS에 작업을 할당하기 위해서는 ECR에 이미지를 푸시하는 과정이 필요하지만
해당 작업은 완료했다 가정하고 진행하겠습니다

## ECS 클러스터, 클러스터 인스턴스 생성

![image](https://user-images.githubusercontent.com/55491354/193438833-21999d30-7547-4a6c-86d5-58f190d54b07.png)
클러스터의 이름을 입력합니다

![image](https://user-images.githubusercontent.com/55491354/193438837-8ff93781-ca97-4229-9d8b-c5e7690436cd.png)
클러스터 인스턴스 생성을 위한 정보를 입력합니다
제가 진행하는 서비스에서는 인스턴스가 많이 필요하지 않아 인스턴스 수를 1개로 생성할 예정입니다
다음에 추가 인스턴스가 필요할 경우 클러스터 설정을 변경할 수 있습니다

![image](https://user-images.githubusercontent.com/55491354/193438840-c6520bec-acec-4ab1-b46e-00f3c481873f.png)
인스턴스가 사용할 VPC를 구성합니다
프로젝트를 위해 구성한 VPC를 사용하고 오토스케일링 가용성을 위해 다른 가용영역의 서브넷을 두 개 할당했습니다

![image](https://user-images.githubusercontent.com/55491354/193438844-80712ccc-d9fb-44d1-8ea2-8f936a67ac0c.png)
인스턴스에 IAM정책을 부여합니다
해당 정책을 통해 ECS가 컨테이너 인스턴스의 활성화 여부, 자원관리 및 태스크를 관리할 수 있습니다

![image](https://user-images.githubusercontent.com/55491354/193438847-7ee53661-1117-49f6-92a3-2cd366ae6e8b.png)
각 태스크에서 발생하는 로그 모니터링 솔루션입니다
ECS 클러스터 오토 스케일링에서 CloudWach 지표를 사용함으로 체크해주세요

---

## CloudWatch 로깅을 위한 인스턴스 IAM 설정

![image](https://user-images.githubusercontent.com/55491354/193438870-6dae7166-fd83-4886-bf94-2eee6b3de6ab.png)

컨테이너 로그를 송출하기위해 컨테이너 인스턴스 IAM역할에 CloudWatchLogsFullAccess 권한을 부여합니다

---

## ECS Task 추가

ECS에서 배포및 관리할 도커 컨테이너를 설정(태스크 디피니션)해 작업(Task)로 만드는 과정입니다

![image](https://user-images.githubusercontent.com/55491354/193438938-217ad8df-f2f7-4184-9889-49730c8e3d2b.png)

태스크 이름을 정의합니다 나머지 값은 기본값으로 설정합니다

### 로깅 컨테이너 추가

![image](https://user-images.githubusercontent.com/55491354/193438943-62828deb-fde5-4ad8-9144-d24baa1beeab.png)

컨테이너 로깅을 위해 로그 라우터 컨테이너를 구성합니다

로그 기록용으로만 사용하는 경우 Fluent Bit 유형을 선택해야 합니다

유형을 선택한 뒤 적용 버튼을 클릭할 경우 로깅 컨테이너가 작업에 추가됩니다

![image](https://user-images.githubusercontent.com/55491354/193438961-890b76a3-2626-4c05-a04e-52a5cb3909e4.png)

log_router 컨테이너 이름을 클릭해 컨테이너 편집 창으로 이동합니다

![image](https://user-images.githubusercontent.com/55491354/193439022-b8f2f9f4-9df8-414f-8b6a-ee97aff0455d.png)

스토리지 및 로깅 설정에서 다음과 같이 설정합니다

```
awslogs-group → firelens-container
awslogs-region → 로그 기록 리전( 서울 : ap-northeast-2 )
awslogs-stream-prefix → firelens
awslogs-create-group → true
```

입력후 업데이트 버튼을 눌러 반영합니다

### 앱 컨테이너 추가

실제로 배포할 서비스 앱 컨테이너를 추가해야합니다

![image](https://user-images.githubusercontent.com/55491354/193438986-c6c69004-d199-49af-b920-44477d0e2c6a.png)

컨테이너 추가 버튼을 눌러

컨테이너 이름과 도커 이미지 링크, 메모리 제한, 포트매핑을 설정했습니다

> 호스트 포스를 0으로 설정하면 다이나믹 포트로 ECS가 자동으로 포트를 관리합니다. 무중단 배포를 활성화하기 위해서는 꼭 호스트포트를 다이나믹 포트로 설정해야 합니다

![image](https://user-images.githubusercontent.com/55491354/193439046-bc4b9f9a-82bf-49c0-acd4-21ec6c384ff3.png)

로그 라우터 컨테이너와 연결하기위해 고급 구성에서 로그 구성을 설정합니다

```
name -> cloudwatch
log_group_name -> 로그 그룹명( ecs-tilog-log )
log_stream_prefix -> from-fluent-bit
region -> 로그 기록 리전( 서울 : ap-northeast-2 )
auto_create_group -> true
```

---

## ALB 생성

방금 정의한 Task를 배포하기 위해서는 ECS에서 서비스를 정의해야 합니다 하지만 그전에
서비스를 연결해줄 ALB를 먼저 생성해야 빠르게 ECS 서비스를 구성할 수 있습니다

![image](https://user-images.githubusercontent.com/55491354/193439067-8fec0c1e-3ad0-4a96-95e5-c629d317aaea.png)![image](https://user-images.githubusercontent.com/55491354/193439072-250a151f-5146-4b4d-bf2b-c8def535bb43.png)
로드밸런서 이름을 입력하고
외부의 접속에 대응하는 Internet-facing 옵션을 선택합니다

![image](https://user-images.githubusercontent.com/55491354/193439077-3a8d5612-7b46-4cd6-8284-0690d8ebd97c.png)
가용영역과 서브넷을 선택했습니다

![image](https://user-images.githubusercontent.com/55491354/193439082-5b5e15b9-dc9e-4296-bc78-3f33c48db5c9.png)
Listener는 클라이언트와 로드밸런서간 연결이며
Target은 로드밸런서와 서비스간 연결을 정의합니다
해당 프로토콜과 포트로 접근하는 클라이언트에 대해서 어느 서비스로 연결할지 정의하기위해
create target group을 클릭해 이동합니다

![image](https://user-images.githubusercontent.com/55491354/193439100-74cbe370-989a-4d55-b438-7c6a7161b1b8.png)
알맞은 VPC를 선택하고 target group을 생성합니다

![image](https://user-images.githubusercontent.com/55491354/193439106-0102a08a-5c6f-40c8-8ece-97b2460061ad.png)
생성한 Target group과 리스너를 매치시키고 로드 밸런서를 생성합니다

---

## ECS 서비스 정의

서비스를 통해 지정된 수의 작업(Task)를 실행하고 관리할 수 있습니다

![image](https://user-images.githubusercontent.com/55491354/193439143-87ef8dac-949e-4d4e-a60f-b1872e5b78c7.png)

배포할 태스크를 선택하고 서비스 생성을 선택합니다

![image](https://user-images.githubusercontent.com/55491354/193439150-544186ca-9aff-4d05-867e-1f4cb96bdb52.png)

서비스에서 작업을 얼만큼 생성하고 관리할지 정의합니다

태스크 개수는 무중단 배포를 위해서 최소 2개로 최대 4개까지 늘어날 수 있도록 설정했습니다

![image](https://user-images.githubusercontent.com/55491354/193439154-282fa39e-c600-436e-ad7e-e8f7d48a0849.png)

다음을 눌러 로드 밸런싱 설정에 진입합니다

이전 과정에서 생성했던 ALB를 연결합니다

![image](https://user-images.githubusercontent.com/55491354/193439165-dc49f19b-aa35-4d2c-b534-d93ef2b6b8e9.png)

컨테이너에서 오픈할 포트를 지정하고 로드 밸런서에 추가합니다

![image](https://user-images.githubusercontent.com/55491354/193439172-702cbcb0-f013-4098-85c8-d58548d2cbb8.png)

해당 컨테이너와 타깃그룹을 연결합니다 이후 동일한 태스크가 늘어나도 ECS에서 자동으로 타깃 그룹에 추가, 삭제합니다

![image](https://user-images.githubusercontent.com/55491354/193439177-803bdebc-4fca-4d79-a276-a33c9d67aeca.png)

클러스터 인스턴스 오토 스케일을 사용하지 않는 상태이기때문에 비활성화 한 상태로 서비스를 생성했습니다

# 결과

---

![image](https://user-images.githubusercontent.com/55491354/193439197-36b978a2-0574-4064-9b4f-204e5e889567.png)

- 서비스가 실행되면 클러스터 인스턴스에 태스크를 분배합니다
- 태스크가 실행되면 자동으로 ALB 타겟그룹에 추가되고 로드밸런싱됩니다

---

# Reference

[AWS ECS 밋업 공식문서](http://ecs.catsdogs.kr.s3-website.ap-northeast-2.amazonaws.com/ko/1.-intro/)

[AWS ECS로 시작하는 컨테이너 오케스트레이션](https://www.44bits.io/ko/post/container-orchestration-101-with-docker-and-aws-elastic-container-service#%EB%93%A4%EC%96%B4%EA%B0%80%EB%A9%B0)

[ECS를 시작하기전 알았으면 좋았을 것들](https://medium.com/harrythegreat/ecs%EB%A5%BC-%EC%8B%9C%EC%9E%91%ED%95%98%EA%B8%B0%EC%A0%84-%EC%95%8C%EC%95%98%EC%9C%BC%EB%A9%B4-%EC%A2%8B%EC%95%98%EC%9D%84-%EA%B2%83%EB%93%A4-1-%EC%9A%A9%EC%96%B4-%EC%84%A4%EB%AA%85-92dbfb9d59f7)
