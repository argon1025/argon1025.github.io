---
title: Npm Helmet 패키지 알고쓰기
category: NodeJS 보안구성
tags:
	- NodeJS
	- NestJS
	- Helmet
	- Npm
tech: node-js
techColor: lime
subTitle: NodeJS Helmet 패키지의 구성에 대해
date: 2021-01-04
---

Helmet은 15개의 보안관련 HTTP 헤더의 설정을 도와주는 미들웨어 기능 모음 모듈입니다.


# 설치
---
```bash
npm i --save helmet
```



# 미들웨어 적용
---
```tsx
import * as helmet from 'helmet';

// This...
app.use(helmet());

// ...is equivalent to this:
app.use(helmet.contentSecurityPolicy());
app.use(helmet.dnsPrefetchControl());
app.use(helmet.expectCt());
app.use(helmet.frameguard());
app.use(helmet.hidePoweredBy());
app.use(helmet.hsts());
app.use(helmet.ieNoOpen());
app.use(helmet.noSniff());
app.use(helmet.permittedCrossDomainPolicies());
app.use(helmet.referrerPolicy());
app.use(helmet.xssFilter());
```

# 보안 헤더 설정
---
총 15개의 설정이 있고 11개의 메서드만 기본적으로 포함되어 있으며
문서 하단 '*' 표시된 메서드는 `helmet()` 에 포함되어 있지 않아 명시적으로 활성화 해야합니다.


## helmet.contentSecurityPolicy
`Content-Security-Policy` 헤더를 설정합니다
자신의 웹사이트에 존재하는 리소스만을 허용하는 헤더를 추가합니다
추가 설정을 통해 관련 허용하는 리소스 URL을 추가할 수 있습니다


## helmet.expectCt
`Expect-CT` 헤더를 설정합니다
헤더가 있을 경우 브라우저가 CT logs를 확인해서 발급된 SSL, TLS 인증서의 유효성을 확인합니다.


## helmet.referrerPolicy
`Referrer-Policy` 헤더를 설정합니다
이 헤더의 설정에 따라 A 사이트에서 B 사이트로 이동할 경우 방문객이 어떤 사이트를 통해 방문했는지 알 수 있습니다


## helmet.hsts
`Strict-Transport-Security` 헤더를 설정합니다
HTTP 대신 HTTPS만을 사용해야한다고 브라우저에 요청합니다


## helmet.noSniff
`X-Content-Type-Options` 헤더를 설정합니다
반드시 정해진 mime type으로 문서를 랜더링 해야한다고 브라우저에 요청합니다


## helmet.dnsPrefetchControl
`X-DNS-Prefetch-Control` 헤더를 설정합니다
DNS 프리패치를 제어해서 서비스의 반응성을 개선할 수 있습니다
기본적으로 비활성화 되어 있습니다


## helmet.ieNoOpen
`X-Download-Options` 헤더를 설정합니다 인터넷 익스프롤러 8에서만 해당됩니다
안전하지 않은 다운로드 파일 이라고 판단되면 브라우저에서 바로 실행을 막고 사용자가 직접 실행하게 합니다.


## helmet.frameguard
`X-Frame-Options` 헤더를 설정합니다
클릭재킹 공격을 막기위해 다른 페이지에서 frame 태그로 우리 웹사이트를 로드하는것을 막습니다


## helmet.permittedCrossDomainPolicies
`X-Permitted-Cross-Domain-Policies` 헤더를 설정합니다
도메인 간 요청에 대해 PDF 및 Flash 문서를 허용하는데 사용됩니다.


## helmet.hidePoweredBy
`X-Powered-By` 헤더를 제거합니다
서버 정보, 서버 버전정보를 숨겨 공격자에게 최소한의 정보만 주도록 합니다


## helmet.xssFilter
`X-XSS-Protection` 헤더를 설정합니다
XSS 공격을 감지할 때 브라우저에서 페이지 로드를 중지시킬 수 있습니다


## helmet.crossOriginEmbedderPolicy() *
`Cross-Origin-Embedder-Policy` 헤더를 설정합니다
CEOP 헤더는 문서가 교차출처 리소스를 로드하지 못하도록 합니다


## helmet.crossOriginOpenerPolicy *
`Cross-Origin-Opener-Policy` 헤더를 설정합니다
최상위 문서가 원본 문서와 탐색 컨텍스트 그룹을 공유하지 않도록 할 수 있습니다


## helmet.crossOriginResourcePolicy *
`Cross-Origin-Resource-Policy` 헤더를 설정합니다
브라우저가 리소스에 대한 CORS 요청을 차단하게합니다


## helmet.originAgentCluster *
`Origin-Agent-Cluster` 헤더를 설정합니다
동일한 사이트간의 동기 스크립트 액세스를 방지하도록 브라우저에 요청합니다


# Reference
---
[https://helmetjs.github.io/](https://helmetjs.github.io/)