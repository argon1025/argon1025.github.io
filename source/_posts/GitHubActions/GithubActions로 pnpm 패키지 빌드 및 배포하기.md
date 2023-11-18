---
title: Github Actions로 패키지 배포 자동화 하기
subTitle: Github Actions로 pnpm을 사용하여 openAPI SDK 패키지 빌드 및 배포를 자동화 해보자
tech: Github
category: CI/CD
tags:
  - pnpm
  - Github
  - GithubActions
  - CI/CD
date: 2023-11-18
---

새로운 사이드 프로젝트를 진행하면서 각각의 마이크로 서비스를 BFF에서 연동하기 쉽도록
[OpenApiTools](https://github.com/OpenAPITools/openapi-generator)를 통해 SDK를 생성하고 패키지를 배포까지 진행 했는데요

하지만 서비스에 새로운 기능이 추가될 때 마다 새로운 SDK를 빌드하고
npm에 퍼블리싱하는 번거로움이 있었기 때문에
Github Actions으로 자동화하여 프로세스를 개선하고자 했어요.

Github Actions를 사용하면서 알게된 사실에 대해 정리하고 공유하고자 합니다.




# CircleCI vs Github Actions
---
Github Actions가 나오기 이전에는 저는 CI/CD Tool로 `CircleCI`를 사용했었는데요.
Github 공개 레포를 사용하고 있는 상황에서는 `CircleCI`를 사용할 이유가 없어보였습니다.

- 가격
	- CircleCI -> 월 6000분 무료, 팀 추가가능 5명
	- Github Actions -> 공개 레포인경우 무료
- 사용 권한
	- CircleCI -> 기본적으로 파이프라인 진행사항은 비공개
	  무료 플랜은 최대 5명까지만 팀으로 초대 가능
	- Github Actions -> 초대 불필요





# Github Actions에 대해
---
Github Actions는 CI/CD 플랫폼으로
빌드, 테스트, 배포 파이프라인을 자동화할 수 있는 워크플로를 생성할 수 있습니다.
워크플로는 레포의 `.github/workflows`  폴더 내 yml 형식으로 하나 이상 정의할 수 있습니다.
 


## 워크플로의 구성요소
---
![워크플로 기본](https://docs.github.com/assets/cb-25535/mw-1440/images/help/actions/overview-actions-simple.webp)

- 이벤트 (event)
	- 워크플로를 트리거해서 워크플로가 실행될 수 있도록 합니다.
- 러너 (Runner)
	- 워크플로를 실행하는 컨테이너 서버 입니다.
- 작업 (job)
	- 하나의 작업은 여러 개의 Step으로 이루어집니다.
	- Step은 실행될 쉘 스크립트와 실제 수행할 작업들의 정보를 명시합니다.
	- 각 작업은 독립적인 러너에서 실행됩니다.
	- 작업은 기본적으로 병렬로 실행되나 실행순서를 제어할 수 있습니다. [(공식문서)](https://docs.github.com/ko/actions/using-workflows/workflow-syntax-for-github-actions#jobsjob_idneeds)
- 액션 (Actions)
	- 작업을 미리 정의해둔 일종의 플러그인입니다.
	- 마켓 플레이스에서 액션을 찾을 수 있습니다.
	- CircleCI의 orb와 같은 기능을 합니다.


## 워크플로 작성해보기
---
위에서 간단하게 Github Actions의 워크플로가 어떤 식으로 구성되는지 알아보았습니다.
위 정보를 바탕으로 패키지를 빌드하고 NPM에 업로드 하는 과정을 소개하겠습니다.

### NPM 토큰 발급하기
---
![image](https://github.com/HotteokGroup/PPOTTO_USER_SDK/assets/55491354/972f9fbb-771c-4b96-b948-24e21d56a511)
먼저 NPM 토큰을 발급을 진행했습니다.
NPM의 2차 인증을 패스하기 위해서 반드시 토큰 유형을 `Authmation`으로 선택 후 진행해주세요.


### 워크플로에서 사용할 환경변수 정의
---
Github에서는 워크플로 작성 전 먼저 노출되어서는 안 되는 정보(*인증정보*)나
수시로 변경될 수 있는 값을 저장할 수 있도록 지원하고 있어요.

![image](https://github.com/HotteokGroup/PPOTTO_USER_SDK/assets/55491354/31366f14-ca23-437a-8551-1e0fb1c04c5c)
NPM 패키지를 업로드 하기 위해서는 토큰이 필요하기 때문에
워크플로 작성 전 먼저 노출되어서는 안 되는 NPM 토큰 정보를
`Repository secrets`에 등록했습니다.

![image](https://github.com/HotteokGroup/PPOTTO_USER_SDK/assets/55491354/eeda5c7a-c8da-4b2a-b48d-e75eb537fc7f)
다음으로는 빌드에 사용할 노드 버전을 `Repository variables`에 등록했습니다.
`Repository secrets` 과 달리 저장 이후에도 값을 보거나 수정이 가능합니다.


### 워크플로 이름 및 이벤트 정의
---
```yml
# .github/workflows/npm-publish.yml

name: PPOTTO Social SDK NPM Publish

# main 브랜치에 push되었을 경우 실행한다.
on:
	push:
		branches:
			- main
```

프로젝트에 `.github/workflows/npm-publish.yml` 파일을 먼저 생성했어요.

- `name` : 워크플로 이름, 작성하지 않을 경우 파일명이 기본값으로 사용됩니다.
- `on` : 워크플로의 트리거를 지정합니다.
	- 트리거 가능한 이벤트 정보는 [공식 문서](https://docs.github.com/ko/actions/using-workflows/events-that-trigger-workflows)에서 확인할 수 있었습니다.

### 러너와 작업명 정의
---
```yml
# .github/workflows/npm-publish.yml

name: PPOTTO Social SDK NPM Publish

# main 브랜치에 push되었을 경우 실행한다.
on:
	push:
		branches:
			- main

jobs:
	{작업 명}:
		runs-on: ubuntu-latest
```
이어서 러너와 작업을 명시해 주도록 하겠습니다.

- `jobs` : 워크플로에서 실행되는 모든 작업을 그룹화합니다.
  작업은 여러 개 선언될 수 있습니다.
- `{작업 명}` : 작업을 정의합니다 영어로만 가능합니다. 
  ex) *build_and_publish*
- `runs-on` : 작업을 수행할 환경을 정의합니다
  기본적으로 리눅스, 윈도우, 맥 환경을 지원합니다. [공식문서](https://docs.github.com/ko/actions/using-workflows/workflow-syntax-for-github-actions#choosing-github-hosted-runners)

### 스텝 정의
---
```yml
# .github/workflows/npm-publish.yml

name: PPOTTO Social SDK NPM Publish

# main 브랜치에 push되었을 경우 실행한다.
on:
	push:
		branches:
			- main

jobs:
	{작업 명}:
		runs-on: ubuntu-latest
		steps:
		- name: 프로젝트 로드
		uses: actions/checkout@v4

		- name: PNPM 사용
			uses: pnpm/action-setup@v2
			with:
				version: latest
		
		- name: NodeJS 설정
			uses: actions/setup-node@v3
			with:
				node-version: ${{ vars.NODE_VERSION }}
				registry-url: "https://registry.npmjs.org"
				cache: "pnpm"
		
		- name: 의존성 설치
			run: pnpm i
		

		- name: openAPI SDK 생성
			run: pnpm run open-api:generate
		
		- name: 패키지 빌드
			run: pnpm run build
		
		- name: Git 프로파일 등록
			run: |
				git config --global user.email "github-action@ppotto.io"
				git config --global user.name "github-action-ppotto"
		
		- name: 패키지 퍼블리싱
			run: pnpm run patch
			env:
				NODE_AUTH_TOKEN: ${{ secrets.NPM_TOKEN }}
		
		- name: 패키지 태그 업데이트
			run: git push --follow-tags origin main
```
`{작업 명}`에서 수행할 작업의 스텝을 명시해 주도록 하겠습니다.

- `name` : 스텝에 설명을 추가합니다.
- `uses` : 미리 정의된 액션을 사용합니다. `with`로 액션별 환경설정을 전달합니다.
	- `actions/checkout` : 깃허브 레포에 접근할 수 있도록 합니다.
	- `pnpm/action-setup` : pnpm을 가상환경에서 사용가능하도록 구성합니다.
		- `version` : 사용할 pnpm 버전을 명시합니다.
	- `actions/setup-node` : nodeJS를 가상환경에서 사용가능하도록 구성합니다.
		- `node-version` : 사용할 노드 버전을 명시합니다.
		- `registry-url` : npm 레지스트리 URL 설정 (패키지 배포를 위해 명시)
		- `cache` : 패키지를 캐싱합니다.  패키지 매니저인 pnpm을 명시합니다.
- `run` : 명령을 실행합니다.
	- `env` : 명령을 실행할 때 환경변수를 지정합니다.
	- `${{ context }}` : 컨텍스트를 통해 사전에 정의한 환경변수에 액세스합니다.
	  사전에 정의된 환경변수 외 다양한 정보에 접근이 가능합니다. [공식문서](https://docs.github.com/en/actions/learn-github-actions/contexts)


# 결과
---
![image](https://github.com/HotteokGroup/PPOTTO_USER_SDK/assets/55491354/a7ba1f03-4925-41ee-ac8f-18f63d481ce4)

main 브랜치에 push 이벤트가 발생할 경우
정의한대로 openAPI SDK를 빌드 후 NPM에 퍼블리싱까지 진행할 수 있게 되었습니다.


# Reference
---
깃허브 공식 문서
https://docs.github.com/ko/actions/using-workflows/about-workflows
