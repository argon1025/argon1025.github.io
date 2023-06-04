---
title: NodeJS 패키지 매니저에 대해
subTitle: NPM 과 Yarn에 대해
tech: NodeJS
category: NodeJS 사용하기
tags:
	- NodeJS
	- NPM
	- Yarn
date: 2022-04-05
---

어떤 패키지 매니저를 사용할지 NPM과 Yarn의 차이점에 대해서 알아보고  
처음 패키지가 설치될 때 어떤 버전이 설치되는지,

모든 팀원이 언제든지 같은 환경에서 개발할 수 있도록
패키지 매니저에서 어떤 식으로 동작하는지를 알아보겠습니다

# NPM vs Yarn

---

> 항상 우리가 마주하는 고민

NodeJS 진영의 대표적인 패키지 매니저로서 NPM, Yarn 두 가지가 존재하고 항상 무엇을 사용할지 고민합니다  
2022년 시점으로 NPM과 Yarn을 있는 그대로 사용할 때 큰 차이는 없습니다  
같은 기능은 간단히 정리하고 Yarn에서 제공하는 특별한 기능에 관해서 알아보겠습니다

## NPM Yarn, 동일한 기능들

### 보안

- NPM : 버전 6 부터 보안 검사를 수행하고 SHA-512으로 패키지를 검증합니다
- Yarn : 라이센스 정보를 통해 종속성 충돌을 예방하고 체크섬을 사용해 패키지를 검증합니다

### 종속성 트리

- NPM : package-lock.json 파일을 생성해서 종속성 버전을 명시하고 동일한 환경을 보장합니다
  - npm-shrinkwrap을 통해서 모든 사용자가 동일한 버전의 버전을 설치하도록 고정합니다
- Yarn : yarn.lock 파일을 생성해서 종속성 버전을 명시하고 동일한 환경을 보장합니다

## NPM 대비 Yarn의 장점

### 속도

Yarn은 패키지를 병렬로 설치하기 때문에 NPM보다 속도가 빠릅니다  
하지만 Yarn 도큐먼트에 의하면 그것은 상대적이며 일시적인 현상으로  
프로젝트가 추구하는 목표가 아니라고 언급합니다

```jsx
Speed is relative and a temporary state. Processes, roadmaps and core values are what stick.

속도는 상대적이며 일시적입니다, 프로젝트 로드맵 및 핵심가치에따라 얼마든지 바뀔 수 있습니다
```

[https://yarnpkg.com/getting-started/qa/#is-yarn-faster-than-other-package-managers](https://yarnpkg.com/getting-started/qa/#is-yarn-faster-than-other-package-managers)

### PnP, 새로운 방식의 의존성 관리

NPM, yarn_v1 은 node_modules 이라는 파일 트리를 사용해 의존성을 관리합니다  
이때 공간을 아끼기 위해서 중복되는 의존성 파일을 끌어 올리는 호이스팅(Hoisting) 기법을 사용하는데  
직접적인 의존성이 없음에도 불구하고 사용이 가능한 유령 의존성 문제(Phantom Dependency)가 발생합니다  
이후 패키지를 업데이트, 삭제할 때 어떤 사이드 이펙트가 발생할지 몰라서 라이브러리가 필요 없음에도  
쉽게 삭제하지 못하는 상황이 발생합니다  
Yarn은 PnP 라는 개념을 통해 새롭게 이 문제를 해결합니다

해당 기능에 대해서 정리한 좋은 글이 있습니다

[node_modules로부터 우리를 구원해 줄 Yarn Berry](https://toss.tech/article/node-modules-and-yarn-berry)

# 언제 어디서든지 동일한 환경에서 개발하기

---

> 버전관리 제대로 알기

이전 섹션에서 지금 시점에서 Yarn을 사용하는 이유에 대해서 알아보았다면  
패키지 매니저가 패키지 버전을 어떤 식으로 명시하고 관리하는지 알아보고  
더 나아가 다른 컴퓨터에서 동일한 환경을 어떻게 보장하는지를 알아보겠습니다

## Package.json에서 패키지 버전을 명시하는 방법

Package.json에서는 프로젝트에 관련된 각종 메타 데이터가 저장됩니다  
`npm install` `yarn install` 을 처음 할 때 패키지 매니저가 Package.json 파일을 참고해서 어떤 패키지 버전을 설치할지 결정합니다

이때 어떤 규칙을 바탕으로 패키지 버전이 명시되어있고 우리가 패키지를 설치할 때 어떤 버전이 설치되는지를 간단히 알아보겠습니다

### 유의적 버전 ( Semantic Versioning )과 노드의 패키지 관리

유의적 버전이란 버전 번호를 어떻게 정하고 올려야 하는지를 명시하는 규칙과 요구사항을 말합니다
버전을 명시하는 방법은 다음과 같습니다

```jsx
X.Y.Z

X -> 메이저 업데이트 : 하위 호환성을 보장하지 않는 변경
Y -> 마이너 업데이트 : 하위 버전에 대해서 호환성을 보장하는 변경
Z -> 패치 : API에 영향이 없는 단순 수정

기존 버전과 호환되지 않게 API가 바뀌면 “주(X) 버전”을 올리고,
기존 버전과 호환되면서 새로운 기능을 추가할 때는 “부(Y) 버전”을 올리고,
기존 버전과 호환되면서 버그를 수정한 것이라면 “수(Z) 버전”을 올린다.
```

버전을 올리는 규칙에 대해서는 [공식문서](https://semver.org/lang/ko/)에서 자세히 다루고 있습니다
현재 말하고자 하는 주제에 벗어나기 때문에 자세한 내용은 위 공식문서를 참고해 주세요

### 패키지 설치 범위를 결정하는 틸드~, 캐럿^ 알기

NPM과 Yarn에서는 NodeJS 패키지가 유의적 버전 ( Semantic Versioning )
규칙을 충실히 따른다는것을 가정합니다

이 가정을 통해서 “어떤 버전 이상의 패키지를 사용할 것이다”를 Package.json에 명시하게 되는데
이것이 버전 앞에 붙는 틸드`~` 캐럿`^` 입니다

`~` 틸드

```jsx
"dependencies": {
    "testPackage": "~1.1.0",
}
```

X.Y.Z 버전 중 Z에 해당하는 ‘패치’ 버전에 대해서만 설치를 허용합니다
위 testPackage의 경우 1.1.0 ~ 1.1.9까지의 버전이 설치되는 것을 허용합니다

`^` 캐럿

```jsx
"dependencies": {
    "testPackage": "^1.1.0",
}
```

X.Y.Z 중 Y.Z에 해당하는 마이너 업데이트에 대한 설치를 허용합니다
위 testPackage의 경우 1.1.0 ~ 1.9.9까지의 설치를 허용합니다

## 처음 생성되는 package-lock, yarn.lock 파일에 대해 알기

npm과 Yarn은 `package.json`에 정의된 버전 범위에 따라 패키지를 설치합니다
하지만 이는 install을 하는 시점, 환경 마다 설치되는 패키지의 버전이 달라질 수 있습니다

npm과 yarn은 이 문제를 해결하기 위해 모든 패키지 설치가 성공한 시점에
설치된 패키지의 버전과 의존성 트리를 그대로 다른 환경에서 재현할 수 있도록
`package-lock.json` `yarn.lock` 파일을 프로젝트 폴더에 생성하고
이후 install시 해당 파일이 있을 경우 `package.json`보다 우선적으로 `lock` 파일을 참고해서 패키지를 설치합니다

이때 생성되는 lock 파일에 대해서는 지켜야 하는 중요한 규칙이 두 가지 있습니다.

- 해당 파일은 Git 저장소를 통해 공유되어야 합니다
- 절대로 직접 수정되어선 안 되며 패키지 매니저 명령어를 통해서 수정되어야 합니다

## 패키지 매니저를 통해서 패키지 관리하기

각 파일의 역할과 중요성을 알았으니 패키지 매니저를 통해서 패키지를 관리할 차례입니다  
Yarn을 기준으로 설명하겠습니다

### 모든 의존성 패키지 설치하기

```
yarn install // NODE_ENV 환경에 따라 개발 의존성 패키지를 설치할지 결정합니다
yarn install —production=true // devDependencies 미설치
yarn install —production=false // devDependencies 설치
```

모든 의존성 패키지를 설치합니다

### 프로젝트에 패키지 추가하기

```bash
yarn add <package-name> // 가장 최신 패키지를 설치합니다
yarn add <package-name>@^4.0.0 // 버전 범위를 지정합니다
yarn add <package-name>@^4.0.0 --dev // devDependencies에 등록되며 프로덕션에서 설치되지 않습니다
```

### 패키지 업데이트

```bash
// 허용 버전 범위 내에서 업데이트
yarn upgrade // 전체 패키지 업데이트
yarn upgrade <package-name> // 특정 패키지 업데이트

// 최신버전으로 업데이트
yarn upgrade --latest // 전체 패키지 최신버전으로 업데이트
yarn upgrade <package-name> --latest // 특정 패키지 최신버전으로 업데이트

// CLI 대화형 패키지 업데이트
yarn upgrade-interactive
yarn upgrade-interactive --latest
```

yarn upgrade로 package.json에서 정의된 버전 범위 (~, ^)범위 내에서 업데이트하고 lock 파일에 기록합니다
이때 package.json은 변경되지 않습니다

`—latest` 태그를 사용하면 package.json에서 정의된 버전 범위를 무시하고 최신버전으로 업데이트합니다
package.json, lock 파일 전부 변경됩니다

대화형 패키지 업데이트 매니저는 콘솔에서 GUI로 쉽게 업데이트할 수 있는 기능을 제공합니다
NPM에선 `npm-check` 패키지를 통해 제공하고 yarn에선 `yarn upgrade-interactive` 명령어를 통해서 제공됩니다

# 마치며

---

모른다는 이유만으로 warn 출력되는 의존성 경고를 무시하고 install만을 외쳤던 과거의 나를 되돌아보는 시간이 되었습니다  
여러분도 과거 자신의 패키지 관리에 대해서 돌아볼 수 있는 시간이 되었으면 합니다

# Reference

---

[https://www.npmjs.com/package/npm-check](https://www.npmjs.com/package/npm-check)

[https://classic.yarnpkg.com/en/docs/cli/upgrade-interactive/](https://classic.yarnpkg.com/en/docs/cli/upgrade-interactive/)

[https://classic.yarnpkg.com/en/](https://classic.yarnpkg.com/en/)

[https://docs.npmjs.com/cli/v8/configuring-npm/package-lock-json](https://docs.npmjs.com/cli/v8/configuring-npm/package-lock-json)

[https://yarnpkg.com/features/pnp](https://yarnpkg.com/features/pnp)

[https://classic.yarnpkg.com/en/docs/cli/](https://classic.yarnpkg.com/en/docs/cli/)
