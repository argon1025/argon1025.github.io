---
title: 그런 프로젝트 설정으로 괜찮은가?
subTitle: NestJS에서 더 나은 개발 환경 구성하기
tech: NestJS
category: NestJS 사용하기
tags:
  - NestJS
  - TypeScript
date: 2021-12-02
---

# 들어가기에 앞서

---

신입 개발자 회고에서 기록 했던 글의 일부입니다

> 사소한 곳에서 발휘하는 정직은 사소하지 않다

> 프로젝트를 빨리 완성하는 왕도는 없으니 세세한 것에 신경 쓰고 팀원 간에 핵심가치(Insight)를 공유하면서 누가 봐도 좋은 아키텍처, 코드 컨벤션을 가진 읽기 쉬운 코드를 만들자

이후 저는 TILog 서비스를 마이그레이션 하기로 정했고  
우리가 쉽게 외면했던 기초적인 프로젝트 설정 내 고려해야 하는 부분에 대해서  
왜 이런 선택을 했고, 어떻게 했는지 기록하고자 합니다  
간단히 자신의 프로젝트에 대해서도 돌아보면서 부족한 점이 있다면 서로 점검해 나가는 시간이 되었으면 합니다

# Package.json 제대로 설정하기

---

Package.json에서는 프로젝트에 관련된 각종 메타 데이터가 저장됩니다
공식 문서를 기반으로 Package.json 파일을 작성했습니다.

## 패키지 기본 설정

```json
// Package.json
}
"name": "tilog-server-node-v2",
  "version": "0.0.1",
  "description": "blog platform for developers",
  "homepage": "<https://github.com/TIL-Log-lab>",
  "repository": { "type": "github", "url": "<https://github.com/TIL-Log-lab/Tilog-server-node-v2>" },
  "author": {
    "name": "argon1025",
    "email": "argon1025@gmail.com",
    "url": "<https://github.com/argon1025>"
  },
  "contributors": [
    { "name": "Daphne-dev", "email": "daphne01215@gmail.com", "url": "<https://github.com/TIL-Log-lab>" }
  ],
  "bugs": "argon1025@gmail.com",
  "private": true,
  "license": "UNLICENSED",
}
```

[yarn package.json 문서](https://classic.yarnpkg.com/lang/en/docs/package-json/)를 참고해서 작성하고 누락된 내용을 추가했습니다
각 부분에 대한 해설은 다음과 같습니다

`version` → 현재 패키지의 버전
`description` → 패키지 설명
`homepage` → 패키지 홈페이지
`repository` → 실제 코드가 있는 위치
`author` → 패키지 작성자 정보
`contributors` → 패키지 기여자 정보
`bugs` → 패키지 문제를 보낼 수 있는 위치
`private` → 패키지를 개시하지 않을 경우 true
`license` → 패키지 라이센스

## nodeJS, yarn 패키지 환경 지정

```json
// Package.json
}
"engines": {
    "node": "^16.14.2",
    "yarn": "^1.22.0"
  },
}
```

해당 애플리케이션을 실행하는데 필요한 NodeJS, Yarn 버전을 지정했습니다

> 2022.04 기준 LTS NodeJS 버전과 최신 yarn 패키지 매니저를 사용하도록 명시했습니다.

## NodeJS 버전 관리 매니저 사용하기

위 섹션에서 프로젝트 실행에 필요한 노드 버전을 고정했습니다
다른 프로젝트에서는 다른 버전이 필요하게 되면 NodeJS를 다시 설치해야하는 번거로움이 존재합니다.
이 상황에서 유용하게 사용할 수 있는 노드 버전 관리 매니저가 있습니다
대표적으로 두가지 버전 매니저가 존재합니다

[nvm](https://github.com/nvm-sh/nvm) → Node Version Manager
[fnm](https://github.com/Schniz/fnm) → Fase Node Manager

# tsconfig.json 제대로 설정하기

---

tsconfig.json은 Typescript를 작성 및 컴파일 하는 과정에서 참조되는 파일 입니다.
프로젝트를 시작하기 전에 Strict 모드 및 공식 문서를 참고해서 필요한 옵션을 더 추가했습니다.

```json
{
  "compilerOptions": {
    /* 기존 설정 */
    "module": "commonjs", // import 문법 설정
    "declaration": true, // .d.ts 파일을 생성할껀지에 대한 유무
    "removeComments": true, // 컴파일시 주석을 제거할것인지 유무
    "emitDecoratorMetadata": true, // 데코레이터 시험 기능 활성화 유무 NestJS 활성화
    "experimentalDecorators": true, // 데코레이터 시험 기능 활성화 유무 NestJS 활성화
    "allowSyntheticDefaultImports": true, // import * as 구문 허용
    "target": "es2017", // 어떤 자바스크립트로 빌드할것인지
    "sourceMap": true, // 디버그용 sourceMap 생성여부
    "outDir": "./dist", // 컴파일된 파일의 위치
    "baseUrl": "./", // 루트폴더를 정의합니다
    "incremental": true, // 마지막 빌드 정보를 저장합니다
    "skipLibCheck": true, // 선언 파일 타입 확인 스킾

    /* 추가된 설정 */
    "strict": true, // 엄격한 타입 체크
    "forceConsistentCasingInFileNames": true, // 일관된 대소문자 검사
    "noFallthroughCasesInSwitch": true, // switch문 엄격한 오류 체크
    "noUnusedParameters": true, // 사용되지 않은 파라미터 오류 출력
    "noUncheckedIndexedAccess": true // 객체 유형이 명확하게 선언되지않았을 경우 undefined 타입을 추가합니다
  }
}
```

[공식문서](https://www.typescriptlang.org/tsconfig)에서 설정의 자세한 설명을 확인할 수 있습니다

# ESLint 제대로 설정하기

---

ESLint는 코드에서 발견된 문제 패턴을 식별하기 위한 코드 분석 도구입니다.
또한 프로젝트에 코드 스타일 가이드 규칙을 정의하고, 규칙에 따라 검사 및 수정을 지원합니다
코드 스타일 가이드에 대해서 알아보고 어떤 코드 스타일 가이드를 선택해 프로젝트에 적용하겠습니다

## 코드 스타일 가이드란

코드 스타일 가이드란 코드 작성 스타일 형식을 규정한 표준입니다
코드 스타일 가이드중에 가장 유명한것은 [airbnb](https://airbnb.io/javascript/), [google](https://google.github.io/styleguide/jsguide.html), [standard](https://standardjs.com/) 입니다
코드 스타일 가이드를 준수하면서 코드를 작성하면 여러 개발자가 동일한 코드 베이스에서 작업하더라도
일관된 스타일로 코드를 빠르게 이해하고 생산성을 높일 수 있습니다
하지만 코드 스타일도 복잡하고 외우기 힘들어서 때문에 ESLint 코드 분석을 통해서 스타일 가이드에 맞게 코드를 작성할 수 있도록 지원하고 있습니다

## ESLint에 코드 스타일 가이드 플러그인 추가하기

TILog 사이드 프로젝트에서는 프론트, 백엔드에서 모두 사용이 가능한 Airbnb 코드 스타일을 따르기로 했습니다

## Airbnb 코드 스타일

Airbnb 코드 스타일 플러그인을 ESLint에 적용하기 위해 다음 패키지를 설치합니다.

```json
/* 리액트를 사용하지 않는 프로젝트인 경우 */
yarn add eslint-config-airbnb-base eslint-config-airbnb-typescript --dev

// .eslintrc.js
"extends": ['airbnb-base','airbnb-typescript/base']
```

리액트를 사용하지 않는 경우 경량버전인 base 버전을 설치합니다

```json

/* 리액트를 사용하는 프로젝트인 경우 */
yarn add eslint-config-airbnb eslint-config-airbnb-typescript --dev

// .eslintrc.js
"extends": ['airbnb','airbnb-typescript']
```

리액트를 사용하는 경우 기본 버전을 설치합니다
리액트 사용 여부에 따라 설치 패키지가 달라지는 이유는 [airbnb 공식 문서를](https://github.com/iamturns/eslint-config-airbnb-typescript#1-setup-regular-airbnb-config) 참고하세요
NestJS에서 ESLint output console 에러가 날 경우 이 [이슈](https://github.com/microsoft/vscode-eslint/issues/696#issuecomment-560799933)를 참고하세요
또한 VSC를 사용하는 경우 단일 워크스페이스를 사용해야 ESLint 에러를 방지할 수 있습니다

# 마치며

---

쉽게 지나칠 수 있는 것을 제대로 한다면
팀원과 함께 깔끔하고 통일된 코드를 작성하기 편한 환경을 만들어나갈 수 있다는 것을
이번 회고를 작성하면서 다시금 느끼는 계기가 되었습니다

# Reference

---

[https://www.typescriptlang.org/ko/docs/handbook/tsconfig-json.html](https://www.typescriptlang.org/ko/docs/handbook/tsconfig-json.html)

[https://eslint.org/](https://eslint.org/)

[https://jbee.io/essay/code-review-goal/](https://jbee.io/essay/code-review-goal/)

[https://opensource.com/article/18/6/anatomy-perfect-pull-request](https://opensource.com/article/18/6/anatomy-perfect-pull-request)

[https://github.blog/2016-02-17-issue-and-pull-request-templates/](https://github.blog/2016-02-17-issue-and-pull-request-templates/)
