---
title: OpenAPI SDK로 프론트엔드와 함께 개발하기
subTitle: OpenAPI-Generator로 SDK 생성해 공유하기
category: NestJS 사용하기
tech: NestJS
tags:
	- NestJS
	- TypeScript
	- OpenAPI
date: 2022-04-23
---

OpenAPI sepcification에 대해서 알아보고 요구사항에 맞춰 코드를 작성한 뒤
OpenAPI-Generator를 이용해서 SDK를 생성해 팀 간에 공유하는 법에 대해서 알아보겠습니다

# OpenAPI Specification 이란

---

OpenAPI(OAS)를 준수하면 RESUfulAPI 소스코드, 문서를 보지 않더라도
사람과 컴퓨터 모두가 서비스 인터페이스를 이해할 수 있는 표준 인터페이스를 작성할 수 있습니다
대표적으로 아래의 데이터가 기계가 이해할 수 있는 JSON, YAML 형태로 작성됩니다

- API 주소, 설명
- HTTP metod
- 요청 헤더, 요청 매개변수
- 상황별 응답 데이터
- 요청, 응답 예제 데이터

이후 해당 표준 인터페이스를 이용해서 다양한 문서 생성(Swagger) 및 클라이언트 생성도구(OpenAPI-generator)를 사용할 수 있습니다

# 기존 서비스 문제점

---

TILog-V1에서는 Swagger Document를 사용해서 엔드포인트 문서를 작성은 했지만
정작 OpenAPI 스펙에는 무지한 상태로 임했기에
OpenAPI에서 정의하는 표준에 한참 못미치는 결과물이 나왔습니다

그 결과 백엔드 개발에 있어서는 문서서를 유지하기 위해서 추가적인 리소스를 소모하고
프론트 개발에 있어서는 문서에 나와있는 API 스펙정보가 정확하지 않아서 소스를 보거나 직접 요청을 보내봐야 했고

백엔드에서 API 스펙이 변경되면 구두로 전달하고 프론트에서도 응답에 대한 인터페이스를 무한히 작성, 수정해야하는 과정이 필요했습니다.

# NestJS에서 OpenAPI Specification 맞춰 개발하기

---

그럼 NestJS 환경에서 OpenAPI Specification에 맞춰 개발하고 나아가
OpenAPI를 이용해서 프론트엔드에서 사용할 SDK를 생성하는 것 까지 해보겠습니다

## NestJS CLI Swagger Plugin 적용

기존 NestJS에서 Swagger에 속성에 대한 정보를 제공하기 위해서
`@ApiProperty` 데코레이터를 각 프로퍼티마다 붙여줘야 했었는데요

NestJS CLI Swagger Plugin을 사용하면 해당 과정을 자동으로 진행해 코드라인을 줄이고
DTO와 ApiProperty간 데이터의 일관성을 지키는 데 도움을 받을 수 있습니다

```json
// nest-cli.json
"compilerOptions": {
    "tsConfigPath": "./tsconfig.json",
    "plugins": [{
      "name": "@nestjs/swagger",
      "options": {}
    }]
  }
```

compilerOptions plugins에 `@nestjs/swagger` 플러그인을 추가하는 것만으로 자동으로 활성화되며
_.dto.ts,_ .entity.ts 파일 대상으로 다음과 같은 행동을 자동으로 수행합니다

- 프로퍼티에 `@ApiProperty` 데코레이터를 자동으로 추가합니다
- 프로퍼티가 옵셔널일 경우 자동으로 `required` 속성이 추가됩니다
- type에 따라서 OpenAPI에 맞는 타입 속성을 설정합니다
- default 할당된 기본값이 있으면 속성으로 반영합니다
- class-validator 데코레이터를 사용 중이라면 자동으로 유효성 검사 규칙을 설정합니다
- 주석을 기반으로 요청, 응답 설명을 생성합니다

이외 자세한 사항은 [NestJS 공식문서](https://docs.nestjs.com/openapi/cli-plugin)를 참고해 주세요

## Request, Response DTO 정확하게 선언하기

```tsx
// get-user-info.dto.ts

// 요청
export class GetUserInfoRequestDto {
  @Type(() => Number)
  @IsNumber()
  @IsNotEmpty()
  userId: users["id"]; // Prisma interface type
}

// 응답
export class GetUserInfoResponseDto {
  userId: users["id"]; // Prisma interface type
  name: users["userName"]; // Prisma interface type
  mailAddress: users["mailAddress"]; // Prisma interface type

  constructor(required: Required<GetUserInfoResponseDto>) {
    Object.assign(this, required);
  }
}
```

NestJS-CLI에 만족하는 파일 형식으로 요청, 응답에 사용하는 DTO를 정의하고 간단한 엔드포인트를 생성했습니다

## OpenAPI-Generator로 SDK 생성하기

### OpenAPI-Generator 란

OpenAPI-Specification을 준수해서 개발했다면
OpenAPI-Generator를 사용해서 API 클라이언트 라이브러리(SDK)를 생성할 수 있습니다

SDK를 생성할 경우 우리가 백엔드에서 열심히 작성한 DTO 요청, 응답 인터페이스를 공유할 수 있게됩니다
또한 API 스펙이 변경되면 SDK 함수도 함께 변경되기 때문에 사이드 이펙트를 에디터 레벨에서 확인할 수 있는 장점이 있습니다

다양한 언어와 API 라이브러리를 사용하는 SDK를 생성할 수 있습니다
자세한 지원사항은 [여기 공식 저장소 문서](https://github.com/OpenAPITools/openapi-generator/blob/master/docs/generators.md)에 기재되어 있습니다

### OpenAPI 문서 JSON으로 저장하기

먼저 OpenAPI-Generator를 사용하기 위해서는 OpenAPI 형식으로 작성된 JSON, YAML 파일이 필요한데요
Swagger Document를 사용하고 있다면 쉽게 OpenAPI 문서를 저장할 수 있습니다

```tsx
const document = SwaggerModule.createDocument(app, config);

if (process.env.NODE_ENV === "local") {
  fs.writeFileSync("library/swagger/openapi-spec.json", JSON.stringify(document));
}
```

로컬 환경에서 서버 애플리케이션을 실행할 경우 자동으로 OpenAPI 문서를 저장하도록 코드를 작성했습니다

### 저장된 OpenAPI 문서로 SDK 생성하기

OpenAPI-Generator는 다양한 [설치옵션(Docker, jar, brew, npm package)](https://github.com/OpenAPITools/openapi-generator#1---installation)을 제공하고 있는데요

그중에서 저는 프로젝트 스크립트로 개발팀원과 해당 내용을 공유하기 위해서 npm package로 작성된 OpenAPI-Generator인 `openapi-generator-cli` 를 사용해서 SDK를 생성하고자 합니다

```tsx
$ yarn @openapitools/openapi-generator-cli --dev
```

먼저 개발팀 전체가 SDK를 생성할 수 있도록 하기 위해 devDependency로 패키지를 추가했습니다

```tsx
"script":{
  "openapi:generate": "openapi-generator-cli generate -i library/swagger/openapi-spec.json -g typescript-axios -o openapi/tilog-api",
}
```

다음 Package.json 스크립트에 SDK를 생성하는 명령어를 추가했습니다
프론트엔드에서 가장 많이 사용하는 조합인 typescript-axios로 SDK를 생성했습니다
OpenAPI-Generator로 생성할 수 있는 언어, 클라이언트 조합은 [이곳 공식문서](https://github.com/OpenAPITools/openapi-generator/blob/master/docs/generators.md)를 참고해 주세요

## SDK 사용하기

생성된 SDK를 사용하면 어디서든지 백엔드와 같은 인터페이스를 공유할 수 있습니다

![image](https://user-images.githubusercontent.com/55491354/207649056-0f27b993-99c6-4fce-9c9c-28205ba7e5aa.png)
모든 엔드포인트가 함수로 작성된 모습입니다
아까 작성한 GetInfo를 사용해 보겠습니다

![image](https://user-images.githubusercontent.com/55491354/207649085-5a7f06b4-4799-463e-9482-8c84f58f149c.png)
해당 엔드포인트를 요청하는데 필요한 매개변수 프로퍼티와 타입을 참조할 수 있습니다

![image](https://user-images.githubusercontent.com/55491354/207649119-07f0ca5e-51fe-4d41-a469-b4aa8e6c8736.png)
응답 데이터에 대한 프로퍼티와 타입을 참조할 수 있습니다

## SDK 공유하기

SDK를 공유하는 방법으로는 대표적으로 두 가지가 있는데요

- OpenAPI JSON 파일을 공유 후 프론트에서 직접 OpenAPI-Generator를 사용하는 방법
- 백엔드에서 OpenAPI-Generator 후 패키지 형식으로 공유 하는방법

TILog-v2에서는 후자의 방식으로 SDK를 공유하고 있습니다
링크 → [https://www.npmjs.com/package/@til-log.lab/tilog-api](https://www.npmjs.com/package/@til-log.lab/tilog-api)

# OpenAPI sepcification을 지켜서 얻은것

OpenAPI Sepcification을 준수하면서 SDK 까지 프로젝트에 적용해 보았습니다
개인적으로 좋았던 점은 다음과 같습니다

- 요청, 응답에 대한 인터페이스가 백엔드, 프론트간 공유된다
- 인터페이스가 공유되기 때문에 객체 프로퍼티 오타 및 타입 불일치 문제가 해결된다
- API 스펙이 변경될 때 OpenAPI-Generator로 생성된 SDK 함수도 변경되기 때문에 사이드 이펙트를 에디터 레벨에서 확인할 수 있다

# Reference

---

OpenAPI Specification으로 타입-세이프하게 API 개발하기: 희망편 VS 절망편

[https://www.youtube.com/watch?v=J4JHLESAiFk](https://www.youtube.com/watch?v=J4JHLESAiFk)

OpenAPI Specification 3.0

[https://swagger.io/specification/](https://swagger.io/specification/)

OpenAPI-generator 생성 타입 리스트 문서

[https://github.com/OpenAPITools/openapi-generator/blob/master/docs/generators.md](https://github.com/OpenAPITools/openapi-generator/blob/master/docs/generators.md)
