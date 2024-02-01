---
title: NestJS + Prisma + Jest 환경에서 Test DB를 구성하여 테스트 및 자동화 하기
subTitle: 이제 NestJS & Prisma & Jest & Docker & Github Actions 를 곁들인
tech: NestJS
category: 테스트
tags:
  - 테스트
  - TDD
  - Jest
  - NestJS
  - GithubActions
date: 2024-01-30
---

기존에는 유닛 테스트를 적용하면서 각 테스트의 의존성 중
공유 의존성 들에 대해서는 MockClass를 만들어서 테스트 더블을 진행했는데요

대표적인 공유 객체 중 하나인 데이터베이스의 경우
비즈니스 요구사항이 복잡해지자 MockClass도 함께 복잡성이 증가되어
MockClass를 유지하는 데에도 비용이 많이 발생하게 되었고
실제 시스템의 동작을 잘 반영하지 못하는 경우 유의미한 테스트를 진행할 수 없는 경우가 있었습니다.

그러므로  `NestJS` `Prisma` `Jest` 환경에서 `Docker`를 사용해서 테스트 DB를 구성하여
테스트 환경을 구축하고 추가로 자동화까지 진행한 내용을 소개해 드리고자 합니다.



# 테스트 DB & Prisma를 사용하는 테스트 코드 작성하기
---

## 1. Docker-compose 로 테스트 DB 구성
```yml
# docker-compose.yml
services:
	muzi-db:
		image: postgres:16
		restart: always
		environment:
			POSTGRES_USER: muzi
			POSTGRES_PASSWORD: muzi
			POSTGRES_DB: Muzi
		ports:
			- 5432:5432
		volumes:
			- ./environments/docker/postgres/data:/var/lib/postgresql/data
		networks:
			- muzi
networks:
	muzi:
		name: muziNetwork
		driver: bridge
```

프로젝트 루트에 docker-compose 파일을 생성합니다.
저는 postgres 16 버전을 사용했습니다.


## 2. Script 추가

```json
// package.json
"scripts": {
	"docker:up": "docker-compose up -d",
	"docker:down": "docker-compose down",
	"prisma:migrate:local": "dotenv -e environments/.env.local -- pnpm exec prisma migrate dev",
	"test:local": "pnpm docker:up && pnpm prisma:migrate:local && dotenv -e environments/.env.local -- jest --passWithNoTests && pnpm docker:down",
}
```

테스트 구동을 위한 스크립트를 추가합니다.
각 스크립트는 다음과 같은 역할을 합니다.

### docker:up
`docker-compose up -d` 를 실행하여 Docker-compose에 정의된 컨테이너를 백그라운드 모드로 실행합니다.

### docker:down
Docker-compose에 정의된 서비스를 중지하고 컨테이너를 삭제합니다.

### prisma:migrate:local
명령어를 실행하기 전 `dotenv -e environments/.env.local` 환경변수를 로드하고
`prisma migrate dev` 명령어를 통해 테스트 DB와 Prisma 스키마 간에 동기화 합니다.

### test:local
위 명령어를 조합하여 아래 순서대로 테스트를 시작합니다.
1. `docker:up` : 테스트 DB를 실행합니다.
2. `prisma:migrate:local`: 테스트 DB와 Prisma 스키마 간에 동기화합니다.
3. `dotenv -e environments/.env.local` Jest 테스트 전 환경변수를 로드합니다.
4. `jest --passWithNoTests` 테스트를 시작합니다.
5. `docker:down` 테스트 DB를 중지하고 정리합니다.


## 3. 테스트 코드 작성하기
테스트 DB를 연결한 테스트 코드를 작성해 보겠습니다.
`PrismaService`의 경우 dotenv 명령어로 로드한 환경변수 파일의 `DATABASE_URL`를 통해 데이터베이스에 연결합니다.

```TypeScript
// campaign.service.spec.ts

describe('CampaignService', () => {
	let campaignService: CampaignService;
	let prismaService: PrismaService;

	// 테스트 시작 전 한 번 실행됩니다.
	beforeAll(async () => {
		// NestJS에서 제공하는 createTestingModule을 사용해 의존성을 DI합니다.
		// 이경우 CampaignRepository 가 PrismaService를 의존합니다.
		const module: TestingModule = await Test.createTestingModule({
		providers: [
			CampaignService,
			{ provide: CAMPAIGN_REPOSITORY, useClass: CampaignRepository },
			PrismaService,
			],
		}).compile();
	
		campaignService = module.get<CampaignService>(CampaignService);
		prismaService = module.get<PrismaService>(PrismaService);
	
	});

	// 각각의 테스트가 종료될 때마다 실행됩니다.
	afterEach(async () => {
	// 테스트 DB의 정보를 초기화합니다.
	await prismaService.$transaction([prismaService.userCampaign.deleteMany(), prismaService.campaign.deleteMany(), prismaService.user.deleteMany()]);
	// jest mock을 초기화 합니다.
	jest.clearAllMocks();
	});

	// 모든 테스트 종료 후 한 번 실행됩니다.
	afterAll(async () => {
	// 테스트 DB 연결을 종료합니다.
		await prismaService.$disconnect();
	});

});

```

위처럼 구성할 경우 다음과 같은 수명주기를 따르게 됩니다.
- 테스트가 실행될 때마다 DB에 연결됩니다.
- 각각의 테스트 케이스가 종료되면 지정된 DB 테이블도 초기화됩니다.
- 모든 테스트가 종료되면 DB연결을 종료합니다.


## 4. 테스트 케이스 추가하기
```TypeScript
// campaign.service.spec.ts

describe('CampaignService', () => {
	// ...위와 동일

	describe('findMany', () => {
		it('삭제된 캠페인은 목록에서 제외한다', async () => {
			// given
			// 삭제된 캠페인 데이터 1건 등록
			const campaignList = [{
				id: 1,
				title: 'title1',
				deletedAt: new Date(),
				updatedAt: new Date(),
				createdAt: new Date(),
			}];
			await prismaService.campaign.create(campaignList);
			
			// when
			const result = await campaignService.findMany({ size: 10, page: 1 });
			// then
			expect(result).toEqual({ list: [], total: 0 });
		});
	});

});

```

`findMany` 에 대한 테스트 케이스 하나를 Given-When-Then 패턴으로 작성했습니다.

- Given(상태) : 삭제된 캠페인이 한 건 등록되어 있다.
- When(행동) : 캠페인 리스트를 조회했다.
- Then(예상되는 결과) : 빈 리스트를 반환한다.

## 5. 테스트 코드 실행하기

![image](https://github.com/argon1025/muzi-backend/assets/55491354/88900d84-1ef6-4f10-815a-7549c6340a72)

`pnpm test:local` 명령어를 통해 테스트를 실행한 결과
테스트 DB를 구성하고 테스트를 진행한 후 테스트 DB 컨테이너를 삭제하는 것까지 확인할 수 있었습니다.

이로써 복잡한 MockClass를 대신해서 테스트 DB로 테스트를 진행할 수 있게 되었습니다.




# Github Actions를 사용해 PR시 테스트 및 리포트 작성하기
---
현재 프로젝트에서는 기능별 브랜치를 나누어 개발한 뒤
PR을 통해서 메인 브랜치에 병합하는 프로세스로 개발을 진행하고 있습니다.
PR을 등록하게 되면 자동으로 코드 테스트 및 테스트 리포트를 작성하도록 구성해 보겠습니다.


## 1. Jest-junit 설치
```bash
pnpm i jest-junit -D
```

Github Actions 의 [Test-reporter](https://github.com/marketplace/actions/test-reporter) 를 사용해서 리포트를 작성하려고 합니다.
해당 액션의 경우 JUnit을 통해서 리포트를 생성하기 때문에
Jest에서 JUnit 리포트를 생성할 수 있도록 해당 의존성을 설치합니다.

## 2. Script 추가
```json
// package.json
"scripts": {
	"test:ci": "pnpm docker:up && pnpm prisma:migrate:test && dotenv -e environments/.env.local -- jest --passWithNoTests --ci --reporters=default --reporters=jest-junit && pnpm docker:down",
}
```

`test:ci` 명령어의 경우 위 섹션에서 추가했던 `test:local` 와 플로우 상 동일하며
테스트 후 Junit 리포트를 작성하도록 Jest 파라미터를 추가했습니다.

![image](https://github.com/argon1025/muzi-backend/assets/55491354/505f481e-0418-48d2-a833-c7892ba81630)
해당 명령어를 실행하면 프로젝트 루트에 `junit.xml`이 생성되는것을 확인할 수 있었습니다.

## 3. Github Action YML 생성
```yaml
name: pull_request_test

# Pull Request가 생성될 경우 실행됩니다.
on:
	pull_request:
		branches:
			- '**'

jobs:
	testAndReport:
		runs-on: ubuntu-latest
		steps:
			# 코드에 접근합니다.
			- name : Checkout
				uses: actions/checkout@v2

			# pnpm 패키지 매니저 설치
			- name: PNPM 사용
				uses: pnpm/action-setup@v2
				with:
				version: latest

			# NodeJS 설치, 프로젝트 루트 .nvmrc 정보를 참조하여 버전을 선택
			- name: NodeJS 설정
				uses: actions/setup-node@v4
				with:
				node-version-file: '.nvmrc'
				cache: 'pnpm'

			- name: 의존성 설치
				run: pnpm i

			# 위에서 정의한 테스트 스크립트 실행
			- name: 테스트 및 결과 저장
				run: pnpm test:ci

			# 테스트 후 생성된 junit.xml를 기반으로 리포트 생성
			# secrets.GITHUB_TOKEN 는 액션 시작 시 자동 할당
			- name: 리포트 생성
				uses: dorny/test-reporter@v1
				if: success() || failure()
				with:
					name: "테스트 결과 리포트"
					token: ${{ secrets.GITHUB_TOKEN }}
					path: junit.xml
					reporter: jest-junit
					fail-on-error: 'true'
```

다음과 같이 액션을 정의하게 되면 PR에서 리포트를 확인하실 수 있게 됩니다.

![image](https://github.com/argon1025/muzi-backend/assets/55491354/c069753d-33c2-4674-bed3-914e68163057)
![image](https://github.com/argon1025/muzi-backend/assets/55491354/88602c7a-d807-47de-9c66-fb01a1bb786d)




# Reference
---
https://www.prisma.io/docs/orm/prisma-client/testing/integration-testing
https://www.prisma.io/docs/orm/prisma-client/testing/unit-testing
https://jojoldu.tistory.com/602