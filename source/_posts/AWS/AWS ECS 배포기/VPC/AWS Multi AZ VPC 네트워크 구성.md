---
title: AWS Multi AZ VPC 네트워크 구성
subTitle: AWS VPC로 가용성이 보장된 안전한 네트워크 환경 구축
tech: AWS
category: AWS ECS 배포기
tags:
  - AWS
  - VPC
date: 2022-01-06
---

프로젝트에 VPC를 적용하기 위해서 공부한 내용을 정리했습니다


# VPC가 해결하는 문제
---

기존 공용 클라우드, AWS VPC 기본 설정에서는 동일한 네트워크 환경에 엑세스하기 때문에
엔드포인트가 닿는 면적이 넣어 보안에 취약한 문제점이 있었습니다.
VPC(Virtual Private Cloud)는 공용 클라우드 내에서 안전하고 격리된 네트워크 공간을 만들 수 있습니다.


# 키워드 정리
---
VPC를 구성하기 전 필요한 용어에 대해 먼저 정리해보겠습니다.

## Region (리전)

AWS 서비스가 위치하고있는 지리적인 장소입니다.
특정 국가의 가용영역을 묶으면 리전이 됩니다

## Availability Zone (가용영역 : AZ)

가용영역 하나는 물리적인 데이터 센터 하나와 같습니다

## VPC (Virtual Private Cloud)

물리적으로 개방된 네트워크에 **논리적으로 격리된 가상의 네트워크**를 구축할 수 있습니다
각각의 VPC는 리전 하나를 두고 완전히 독립된 서브넷 라우팅 게이트웨이를 구성할 수 있습니다

## Subnet

서브넷은 VPC에서 설정한 IP 대역폭을 잘게 나누어 하나의 가용영역에 연결합니다
서브넷은 주로 하나의 가용영역에 public subnet, private subnet으로 나누며
인터넷과 연결되어 외부에 노출되어 있는 지점을 최소화할 수 있습니다

## Router

VPC안에서 발생한 네트워크 요청에 대해 그에대한 최적의 경로를 지정합니다

## Internet Gateway

VPC와 외부 인터넷 통신을 활성화하기 위한 게이트웨이입니다

## NAT Gateway

private subnet이 외부의 인터넷 연결이 필요할 경우 구성하며
public subnet의 라우팅 테이블을 따라 인터넷 통신을 가능하게 합니다

## ACL

Subnet 단위로 적용되는 방화벽입니다
Subnet은 하나의 ACL만 가질 수 있고 ACL은 여러 개의 Subnet에 연결할 수 있습니다



# 구성 목표
---
해당 포스트에서는 VPC를 보안성을 고려해 public, private으로 나누고
두개의 가용영역을 통해 가용성을 확보하도록 하겠습니다.


![image](https://user-images.githubusercontent.com/55491354/193416652-40261765-a460-4654-958f-32a9a4b0fa44.png)

- 하나의 리전에 VPC를 생성하고 두 개의 가용영역(Multi AZ)에 각각 public, private subnet을 생성합니다
- public subnet은 하나의 라우팅 테이블을 가지고 인터넷 게이트웨이와 연결 합니다
- private subnet은 각각의 라우팅 테이블을 가지며 각 가용영역의 NAT Gateway를 통해서 외부 인터넷과 연결합니다
  - 하나의 가용영역에만 NAT Gateway를 두는 패턴도 많으나 가용성을 유지하려면 해당 배치가 맞다고 판단했습니다
- 각 서브넷에 동일한 프로파일 ACL 설정
- 이후 인스턴스 추가 시 별도의 보안그룹 설정합니다 (인바운드: 서비스 포트만 오픈, 아웃바운드 : 전체 허용)

## CIDR 설정

네트워크 클래스를 대체하는 라우팅 기법으로 IP주소를
여러 네트워크 영역으로 나눌 때 기존방식에 비해 유연하게 할 수 있도록 합니다
프로젝트 VPC구성에 사용될 서브넷은 다음과 같습니다

```
// 각 서브넷의 사이더는 총 27개의 ip 대역폭을 가집니다( 앞4개 뒤1개는 예약됨 )
172.16.0.0/24
public subnet1: 172.16.0.0/27
public subnet2: 172.16.0.32/27
private subnet1 : 172.16.0.64/27
private subnet2: 172.16.0.96/27
미사용 172.16.0.128/27
미사용: 172.16.0.160/27
```



# AWS에서 구성하기
---

## VPC 생성하기

![image](https://user-images.githubusercontent.com/55491354/193438154-3afc9893-b08b-41cd-8531-6b08ecd6e93d.png)
리전, CIDR를 입력하고 VPC를 생성합니다

![image](https://user-images.githubusercontent.com/55491354/193438158-7a04557d-8ab1-4c42-9702-43b572bd6a39.png)
어떤 VPC에 해당 서브넷을 생성할지 선택합니다.


## 서브넷 생성하기

![image](https://user-images.githubusercontent.com/55491354/193438169-c30011f0-478a-4a64-81fd-8b7a0201001b.png)

서브넷 이름을 지정하고
서브넷이 어느 가용영역에 속할것인지 정합니다
이번 프로젝트에서는 2개의 가용영역에 총 4개의 서브넷을 생성합니다

![image](https://user-images.githubusercontent.com/55491354/193438173-45283cb2-bdda-44a6-8f8a-d6ea2915d3e9.png)

해당 서브넷의 ip 대역폭을 설정합니다 VPC 대역폭보다 적어야하며
각 서브넷 관리 아이피( 앞4개 뒤1개는 예약됨 )를 제외하고 한개 이상이어야 합니다
나머지 3개의 서브넷도 위와 같이 설정했습니다.



## Internet Gatway 생성하기

![image](https://user-images.githubusercontent.com/55491354/193438190-5f9cc5dd-824a-41a6-b47e-c7bde37772bf.png)
외부 통신을 위한 Internet Gatway를 생성합니다



## Public subnet Routing

![image](https://user-images.githubusercontent.com/55491354/193438214-c07e0340-410b-436f-bbf6-ccd567f28ecf.png)

퍼블릭 서브넷은 하나의 라우팅 테이블을 통해 내부와 외부로 통신합니다
내부 아이피가 아닌 모든 요청은 인터넷 게이트웨이로 패킷을 전송합니다

![image](https://user-images.githubusercontent.com/55491354/193438219-fe685e41-c115-44da-820e-51de2dd686c5.png)

> 라우팅 테이블 → 작업 → 서브넷 연결 편집

에서 라우팅 테이블을 적용할 서브넷을 클릭하고 저장합니다



## NAT Gateway 생성하기

### NAT에서 사용할 EIP 생성

![image](https://user-images.githubusercontent.com/55491354/193438267-237f374c-c0ad-4f6f-b7d8-3411ccf4b1f6.png)
NAT-gateway에서 사용할 EIP를 생성합니다

### 게이트웨이 생성

![image](https://user-images.githubusercontent.com/55491354/193438274-42e5e9a5-37d1-4227-a798-2d970c21a89f.png)

NAT-Gateway를 생성합니다
퍼블릭 서브넷을 지정하고 생성한 EIP를 할당합니다
각 가용영역마다 NAT-Gateway를 배치하기위해 해당 작업을 두번 진행합니다
**NAT-Gateway 생성은 Internet-Gateway가 라우팅된 퍼블릭 서브넷이 있는 상태에서만 생성할 수 있습니다**

---

## Private subnet Routing

![image](https://user-images.githubusercontent.com/55491354/193438289-90cfb1b1-d799-40a8-b1e6-dbbc68813584.png)

각 가용역영별로 가용성을 보장하기위해 프라이빗 서브넷은 별도의 라우팅 테이블로 내부와 통신합니다
내부 아이피가 아닌 모든 요청은 해당 가용영역의 퍼블릭 서브넷에 존재하는 NAT-Gateway로 전송합니다

![image](https://user-images.githubusercontent.com/55491354/193438294-38c90400-1a8f-41da-b952-5bcf29ee3619.png)

> 라우팅 테이블 → 작업 → 서브넷 연결 편집

에서 적용할 서브넷을 클릭하고 저장합니다



## 퍼블릭 서브넷 아이피 자동할당 설정
---
![image](https://user-images.githubusercontent.com/55491354/193438316-eeef4d17-f180-4665-9c42-3fb8c0bb3d60.png)

인스턴스를 퍼블릭 서브넷에 배치할 경우 퍼블릭 IP주소 할당이 비활성화되어 있기 때문에
라우팅 테이블이 제대로 설정되어 있더라도 외부에서 접근할 수 없습니다
기본 AWS VPC와 같이 퍼블릭 IP를 할당하려면 서브넷 설정에서 자동할당 설정을 활성화 해야 합니다



# ACL
---
ACL은 네트워크 구성단계에서 적용하지 않고 각 인스턴스별 보안그룹만 적용 했습니다.

> AWS 공식문서의 권장 ACL 규칙 문서 [링크](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Scenario2.html#nacl-rules-scenario-2)

```
많은 Linux 커널(Amazon Linux 커널 포함)은 포트 32768-61000을 사용합니다.

Elastic Load Balancing에서 시작된 요청은 포트 1024-65535를 사용합니다.

Windows Server 2003을 통한 Windows 운영 체제는 포트 1025-5000을 사용합니다.

Windows Server 2008 이상 버전은 포트 49152-65535를 사용합니다.

NAT 게이트웨이는 포트 1024-65535를 사용합니다.

AWS Lambda 함수는 포트 1024-65535를 사용합니다.
```

> ACL 구성 참고 문서 [https://docs.aws.amazon.com/vpc/latest/userguide/vpc-network-acls.html#nacl-ephemeral-ports](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-network-acls.html#nacl-ephemeral-ports)



# Reference
---

[VPC와 서브넷 AWS 공식 문서](https://docs.aws.amazon.com/ko_kr/vpc/latest/userguide/VPC_Subnets.html)

[AWS VPC구성](https://www.joinc.co.kr/w/Site/cloud/AWS/VPC)

[AWS 다중 가용영역 패턴에 대해서](https://stackoverflow.com/questions/55067043/aws-private-public-vpc-nat-gateway-multiaz-pattern)

[VPC 기본개념 이해하기](https://jbhs7014.tistory.com/164)
