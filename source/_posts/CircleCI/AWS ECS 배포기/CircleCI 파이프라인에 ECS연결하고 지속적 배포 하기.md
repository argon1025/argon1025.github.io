---
title: CircleCI로 ECR, ECS연동
category: AWS ECS 배포기
tech: linux
techColor: gray
subTitle: CircleCI 파이프라인에 ECS연결하고 지속적 배포 하기
date: 2022-01-18
tags: 
	- CircleCI
	- CI/CD
	- AWS ECR
	- AWS ECS
---



CI/CD Tool CircleCI를 이용해서 CircleCI 파이프라인에 ECS를 연결하며 공부한 내용을

정리하고자 합니다.



---



# CICD는 어떤 문제를 해결하는가

기존에 최신 코드를 배포하기위해서
프로덕션에 맞게 환경설정 작성 → 도커 빌드 → ECR Push → ECS Task 개정 생성 → 서비스 업데이트 개정 생성
과정을 거쳐야 했고 Github 통계서버, 블로그 서버, 프론트 서버 모두 이런 과정을 직접 해야하는 문제가 있었습니다
하지만 CICD는 자동화와 모니터링을 통해 지속적인 통합, 지속적인 배포를 할 수 있게 하고
배포 시 마다 개발 코드를 프로덕션 환경에 맞게 코드를 수정, 개선 하기 위해 오랜 시간 작업해야 하는
**integration Hell** 문제를 해결할 수 있습니다



---


# CICD 키워드 정리

![image](https://user-images.githubusercontent.com/55491354/193439506-69046637-4e88-479a-93a1-998614d793eb.png)

> [https://www.redhat.com/ko/topics/devops/what-is-ci-cd](https://www.redhat.com/ko/topics/devops/what-is-ci-cd)

## DevOps
Development + Operatorions 개발과 운영의 경계를 허물고
개발 테스트 배포 운영을 통합해 개발 속도를 높이는 것을 목표로 하는 문화이고
DevOps를 구현한 결과가 바로 CI/CD 파이프라인입니다

## CI (지속적 통합 : Continuous Integration)
다수의 개발자가 작성하고 수정한 소스 코드가 자동화된 Test( 단위, 통합 테스트 ), Build를 상시로 실시하는 것

## CD (지속적 전달 : Continuous Delivery )
코드의 변경사항을 github 또는 컨테이너 레지스트리에 자동으로 업로드되고 이후 프로덕션 환경으로 릴리스 하는것

## CD ( 지속적 배포 : Continuous Deployment )
지속적 전달(Continuous Delivery)과 달리 코드의 변경사항을 프로덕션 환경까지 자동으로 릴리스하는 것



---




# CircleCI 선택과 이유
-   무료입니다
-   무료플랜에서 Private 저장소를 사용할 수 있습니다
-   Orb를 통해 추가 쉘스크립트 작성 없이 AWS 서비스에 쉽게 연결하고 배포할 수 있습니다
-   알림( 메일, 슬랙, JIRA )을 빠르게 구성할 수 있습니다

[이곳](https://ichi.pro/ko/hyeonjae-sayong-ganeunghan-choegoui-ci-cd-dogu-27gaji-194611649728144)에서 다른 다양한 CICD도구의 특징에 대해서 자세히 알아볼 수 있습니다.



---


# CircleCI 키워드

## Orb
실행환경, 환경변수, 작업, 명령어가 미리 정의된 공유가능한 오픈소스 패키지 입니다
환경변수는 매개변수화되어 사용할 때 작업의 하위 옵션으로 전달해줄 수 있습니다

## Pipeline
파이프라인은 프로젝트의 전체 .circleci/config.yml 파일을 포함하는 작업의 최상위 단위입니다
파이프라인은 .circleci/config.yml 구성파일이 포함된 프로젝트의 변경사항이 푸시될 때 시작됩니다
파이프라인에는 워크플로우가 포함됩니다

## Workflow
작업 모음과 실행순서를 정의하기위한 규칙의 집합입니다

## Job
작업은 스탭의 모음입니다

## step
스탭은 실행가능한 명령어를 정의합니다


---


# 프로젝트 CICD Pipeline 구성

![image](https://user-images.githubusercontent.com/55491354/193439554-2b57d20b-725a-4b92-84c2-a304b2157c08.png)

버전관리 : `Github`
CI/CD : `CircleCI`
빌드 이미지 저장소: `AWS ECR`
컨테이너 오케스트레이션 : `AWS ECS`
컨테이너 인스턴스: `AWS EC2`


---


# Dockerfile
프로젝트 앱의 도커파일을 작성합니다
도커파일에는 프로젝트의 구성환경, 빌드 명령어가 명시되어 있습니다

```yaml
# 베이스 이미지를 지정합니다
FROM node:16
# MAINTAINER argon1025@gmail.com

# 각명령어의 디렉토리 위치는 매번 초기화되기 때문에
# 기본 디렉토리를 생성하고 해당 디렉토리로 위치를 고정 합니다
WORKDIR /usr/src/app

# 프로젝트를 복사합니다
COPY ./ ./

# 모듈을 설치합니다
RUN npm install

# 프로젝트를 빌드합니다
RUN npm run build

# 포트를 오픈합니다
EXPOSE 80

# 앱을 시작합니다
CMD [ "node", "dist/main.js"]
```



---

# CircleCI config.yml
CircleCI config.yml 파일에는 푸시가 되었을때 실행되는 파이프라인에대한 설정을 포함하고 있습니다

```yml
version: 2.1

# 오브를 추가합니다
# 오브엔 실행환경, 환경변수 작업, 명령이 미리 구성되어 있습니다
orbs:
  aws-ecr: circleci/aws-ecr@7.3.0
  aws-ecs: circleci/aws-ecs@2.2.1

# 워크플로우에서 사용할 작업을 정의합니다
# <https://circleci.com/docs/2.0/configuration-reference/#jobs>
jobs:
  # nestJS 빌드를 진행하는 작업입니다
  BuildTest:
    docker:
      - image: cimg/node:lts-browsers
    steps:
			# 프로젝트 폴더를 복사합니다
      - checkout
			# 명령어를 실행합니다
      - run:
          name: 'install npm module'
          command: |
            cd src
            npm i
      - run:
          name: 'build project'
          command: |
            cd src
            npm run build
  # CircleCI 컨텍스트를 활용해 .ENV 파일을 프로젝트 폴더에 생성해주는 작업입니다
  createEnv:
    machine: true
    steps:
      - checkout
      - run:
          name: 'Create .env file'
          command: |
            ./.circleci/scripts/generate-env-files.sh
            ls -al
            cat .env
      - persist_to_workspace:
          root: .
          paths:
            - .

# 워크플로우를 정의합니다
workflows:
  # 빌드 테스트
  # 작업 조건 : TIL 로 시작하는 브랜치
  Test-build:
    jobs:
      - BuildTest:
          filters:
            branches:
              only: /^TIL-.*/
  # 이미지 푸시
  ECR-push-ECS-deploy:
    jobs:
      # CircleCI 컨텍스트를 활용해 .ENV 파일을 프로젝트 폴더에 생성해주는 작업
      # 작업 조건 : release 브랜치 커밋
      - createEnv:
          context:
            - TILOG_STATS_BACKEND_PROD
          filters:
            branches:
              only:
                - release
      # Dockerfile을 빌드하고 ECR에 푸시하는 작업
      # 작업 조건 : 상위 createEnv 작업 종료, release 브랜치 커밋
      - aws-ecr/build-and-push-image:
          # 이전 작업영역을 이어받습니다
          attach-workspace: true
          checkout: false
          # AWS 액세스 키
          aws-access-key-id: AWS_ACCESS_KEY_ID
          # AWS 시크릿 액세스 키
          aws-secret-access-key: AWS_SECRET_ACCESS_KEY
          # ECR 저장소 링크
          account-url: AWS_ECR_ACCOUNT_URL
          # ECR 저장소가 존재하지 않으면 생성합니다
          create-repo: true
          # dockerfile 명
          dockerfile: Dockerfile
          # 서비스 리전
          region: AWS_DEFAULT_REGION
          # 레포지토리 명
          repo: '${AWS_ECR_NAME}'
          # 이미지 태그
          tag: '$CIRCLE_SHA1'
					# 선행되어야하는 작업
          requires:
            - createEnv
					# 특정 브랜치에 푸시된 커밋만 진행합니다
          filters:
            branches:
              only:
                - release
      # ECR에 푸시된 컨테이너 이미지를 사용해 ECS 태스크 서비스를 업데이트 하는 작업
      # 작업 조건 : 상위 aws-ecr/build-and-push-image 작업 종료, release 브랜치 커밋
      - aws-ecs/deploy-service-update:
          # 컨테이너:태그 container:tag
          container-image-name-updates: 'container=${AWS_ECR_NAME},tag=${CIRCLE_SHA1}'
          # ECS 클러스터 이름
          cluster-name: '${AWS_ECR_NAME}-ECS-Cluster'
          # 업데이트할 서비스명
          service-name: '${AWS_ECR_NAME}-service'
          # 개정할 작업 정의 명
          family: '${AWS_ECR_NAME}'
					# 선행되어야하는 작업
          requires:
            - aws-ecr/build-and-push-image
					# 특정 브랜치에 푸시된 커밋만 진행합니다
          filters:
            branches:
              only:
                - release 
```



---



# 적용 결과
## 깃허브 PR 결과확인

![image](https://user-images.githubusercontent.com/55491354/193439574-44bc4422-a53e-449a-8b48-0b6994a4ed1f.png)![image](https://user-images.githubusercontent.com/55491354/193439580-286cb835-ca67-4586-9ed5-2c705783e6f1.png)
테스트 빌드가 실패, 성공했을경우



![image](https://user-images.githubusercontent.com/55491354/193439602-191deaec-c7d3-48b6-8647-a198d5962c8f.png)![image](https://user-images.githubusercontent.com/55491354/193439587-25c52414-c7ca-46ff-a815-28fc03e45004.png)

배포가 성공하면
ECS에서 새로운 태스크가 개정되고 서비스가 업데이트됩니다



---



# Else
## CircleCI에서 Slack으로 메시지 전송하기

Orb를 사용하면 Slack에 쉽게 사용자정의 메시지를 전송할 수 있습니다

[공식문서](https://circleci.com/developer/orbs/orb/circleci/slack)



---



# Reference

[CircleCI 공식문서](https://circleci.com/docs/2.0/getting-started/)

[DEVOPS에 대해서](https://www.redhat.com/ko/topics/devops/what-is-continuous-delivery)

[애자일 방법론이란?](https://www.redhat.com/ko/devops/what-is-agile-methodology)

[인테그레이션 헬](https://www.solutionsiq.com/agile-glossary/integration-hell/)