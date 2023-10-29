---
title: SEO에 대해서
subTitle: 2023년 SEO 구성 가이드
tech: Book
category: SEO
tags:
  - SEO
date: 2023-06-10
---

Hexo를 사용한 블로그 스킨을 만들면서 SEO에 대해 검토하고
이를 적용하는 시간을 가진 경험을 공유하고자 합니다.

# SEO
---
SEO(search engine optimization)는 **검색엔진 최적화**의 의미로
검색엔진으로부터 웹사이트나 웹페이지에 대한 웹사이트 트래픽의 품질과 양을 개선하는 과정입니다.

SEO에 친화적일수록 검색 결과 상위에 노출되고 이는 더 많은 트래픽 유입을 유도하게 됩니다.


# SEO 트랜드는 변화합니다
---
대표적으로 구글은 검색엔진 알고리즘을 수시로 변경합니다([Google 9년간 업데이트 기록](https://moz.com/google-algorithm-change)) 이로인해
검색 결과 노출 순서가 수시로 변경되는 현상이 발생하죠

그렇기에 상위노출이 중요한 사이트를 운영 중이라면 SEO최신 트랜드와
검색엔진 회사(Google, Naver)에서 제공하는 가이드라인을 꾸준히 모니터링하고
해당 조건을 만족하는 것이 중요합니다

다음은 주요 검색엔진의 SEO 가이드 문서 링크입니다
- [Google 검색 센터](https://developers.google.com/search?hl=ko)
- [NAVER 웹 마스터 가이드](https://searchadvisor.naver.com/)
- [Bing SEO 문서](https://www.bing.com/webmasters/help/webmasters-guidelines-30fba23a)



# SEO 체크리스트
---
현재 주로 사용되고 효과적인 SEO 전략들에 대해 간단히 살펴보겠습니다


## 단순한 URL 구조
---
```text
// 간단하면서도 구체적인 단어 사용
https://myblog.com/post/postname 👍
https://myblog.com/1q2w3e4r/hotteok 👎

// 영어가 아닌 언어는 UTF-8 인코딩 사용
https://myblog.com/post/%D9%86%D8%B9 👍
https://myblog.com/post/نعنا 👎

// 단어를 하이픈으로 분리 (for Google)
https://myblog.com/post/post-name 👍 ( 하이픈 - )
https://myblog.com/post/post_name 👎 ( 밑줄 _ )
https://myblog.com/post/postname 👎 ( 단어 통합 )
```
> https 프로토콜 사용은 필수

사이트 URL 구조는 단순 해야합니다
URL 구조는 논리적이고 이해하기 쉬운 방식으로 구성하세요


## 크롤링 가능한 링크
---
```text
// 적절한 <a>태그 사용
<a href="https://myblog.com/post"> 👍
<a href="/post/%D9%86%D8%B9"> 👍

// Bad case
<a routerLink="/post/%D9%86%D8%B9"> 👎 ( href 지향 )
<span href="/post/%D9%86%D8%B9"> 👎 ( 다른 태그에 링크설정 지양 )
<span onClick="javascript:goto(A)">Link</span> 👎 ( JS 동적 생성 지양 )
```

Google, Naver 검색엔진의 경우 `<a>`태그를 올바르게 사용할 경우만 링크를 크롤링할 수 있습니다



## robots.txt 설정
---
`robots.txt`는 검색엔진에 사이트를 수집할 수 있도록 허용하거나 제한하도록하는 국제 권고안입니다.

```text
// 모든 검색엔진 허용, 내 사이트 모든 URL 수집 허용
User-agent: *
Allow: /

// 모든 검색엔진 비허용, 내 사이트 모든 URL 수집 비허용
User-agent: *
Disallow: /
```
> robots.txt 파일은 루트 디렉토리에서 접근 가능해야 하며, UTF-8로 인코딩되어 텍스트 파일 (text/plain)로 제공되어야 합니다

robots.txt는 더 다양한 접근 규칙을 설정할 수 있습니다
상세 구성은 아래 문서를 참고하세요

[Google robots.txt 파일 작성 및 제출방법](https://developers.google.com/search/docs/crawling-indexing/robots/create-robots-txt?hl=ko#create_rules)
[Naver robots.txt 설정하기](https://searchadvisor.naver.com/guide/seo-basic-robots)

## Sitemap 설정
---
> 사이트맵을 자동으로 생성해주는 사이트들이 있습니다 [여기](https://www.google.com/search?q=generate+sitemap&hl=ko)를 참고하세요

사이트맵은 사이트에 있는 모든 리소스에 관한 정보를 제공하는 파일입니다
해당 정보를 토대로 검색엔진이 사이트를 조금 더 빠르게 파악할 수 있도록 합니다

### 사이트맵이 필요한 경우
- 크기가 큰 사이트
- 콘텐츠가 많은 사이트
- 새로 만든 지 얼마 안 된 사이트

### 사이트맵이 불필요한 경우
- 크기가 작은 사이트
- 내부적으로 잘 연결된 사이트

사이트맵은 `XML`, `RSS, mRSS, Atom 1.0`, `plain text` 3가지 형식을 지원합니다
주로 XML 방식을 사용하며 표준에 대한 문서 링크는 [여기](https://www.sitemaps.org/protocol.html)를 참고하세요


### Sitemap 파일 검색엔진에 알리기
Sitemap을 작성했다고 해서 검색엔진이 사이트맵을 사용한다고 보장하지않습니다.
Sitemap을 작성했다면 검색엔진에 알려주세요

```text
// Google (핑 도구 사용해서 사이트맵 제출)
https://www.google.com/ping?sitemap=FULL_URL_OF_SITEMAP
```


## 페이지 Header 구성
---
HTML 표준에 따라 `<head>` 내 유효한 요소를 사용해야합니다
먼저 유효한 요소, 유효하지 않은 요소에 대해 간단히 알아본 다음 자세한 표기 방법에 대해 알아보겠습니다.

**유효한 요소**
-   title
-   meta
-   link
-   script
-   style
-   base
-   noscript
-   templat

**유효하지 않은 요소**
> 유효하지 않은 요소를 반드시 사용해야한다면 `<head>` 태그 마지막에 선언하세요
-   iframe
-   img


그럼 유효한 요소 중 검색엔진이 주로 보는 요소 구성 방법에 대해 알아보겠습니다

### Title
```
// 제목 (페이지에 대한 키워드 정보 포함)
<title>blog - 포스트 제목</title>
```

### Meta
#### 대표 URL
```
// 대표 URL
<link rel="canonical" href="http://myblog.com/">
```

#### 간단 설명
```
// 페이지 설명
<meta name="description" content="A description of the page">
```

#### 반응형 웹
```
// 해당 웹은 반응형입니다
<meta name="viewport" content="width=device-width, initial-scale=1.0">
```
반응형 페이지일 경우 페이지의 확대 축소 없이 웹 콘텐츠를 표시할 수 있다고 알립니다
별도의 모바일 페이지가 존재할 경우 [별도의 모바일 URL제공](https://searchadvisor.naver.com/guide/markup-mobile) 페이지를 참고하세요

#### Favicon
```
// 파비콘(웹 아이콘)설정 택 1
<link rel="icon" href="/path/to/favicon.ico">

<link rel="apple-touch-icon" href="/path/to/favicon.ico">
<link rel="apple-touch-icon-precomposed" href="/path/to/favicon.ico">
<link rel="shortcut icon" href="/path/to/favicon.ico">
```


#### 오픈그래프
```
// 기본적으로 사용되는 오픈그래프
<meta property="og:type" content="website"> // 페이지 타입 (video, move)
<meta property="og:title" content="페이지 제목">
<meta property="og:description" content="페이지 설명">
<meta property="og:image" content="http://www.mysite.com/myimage.jpg"> // 대표 이미지
<meta property="og:url" content="http://www.mysite.com"> // 페이지 메인 주소



// 다른 소셜미디어에서 추가로 사용하는 오픈그래프 정보 (Optional)
<meta name="twitter:card" content="트위터 카드 타입(요약정보, 사진, 비디오)"> 
<meta name="twitter:title" content="콘텐츠 제목"> 
<meta name="twitter:description" content="웹페이지 설명"> 
<meta name="twitter:image" content="표시되는 이미지">

```
사이트가 소셜 미디어에 공유될 때 우선적으로 활용되는 정보입니다
자세한 정보는 [오픈그래프 프로토폴](https://ogp.me/) 사이트를 참고하세요
다른 소셜 미디어에서 추가로 사용하는 오픈그래프 정보는 [링크](https://metatags.io/)에서 쉽게 테스트할 수 있습니다.


## 이미지 최적화
---
검색엔진은 웹페이지의 로딩 속도를 체크합니다

이미지가 많은 페이지인 경우 이미지의 크기를 줄여 로딩 속도를 개선하는 것이 중요합니다
이미지 최적화는 내부 모듈 또는 웹에서 쉽게 찾을 수 있는 이미지 최적화 도구를 사용할 수 있습니다


# 테스트
---
SEO 구성을 마쳤다면 내가 올바르게 구성한 것인지
간단하게 검증해볼 수 있는 페이지들이 존재합니다
그중 몇 가지를 소개하겠습니다.

- [모바일 친화성 테스트](https://search.google.com/test/mobile-friendly?hl=ko)
	- 얼마나 모바일 친화적인지, 언제 Google검색엔진이 색인했는지 확인할 수 있습니다.
- [네이버 사이트 간단 체크](https://searchadvisor.naver.com/tools/sitecheck)
	- 사이트 제목, 설명문, 로봇 차단여부 등 기본적인 정보를 확인할 수 있습니다.
- [Google Page Speed Insights](https://pagespeed.web.dev/)
	- 사이트 성능 측정 및 개선방향을 확인할 수 있습니다.


# 마치며
---
현 시점에 모든 검색엔진에서 통상적으로 사용하고있는 SEO 구성방법에 대해서 알아보았습니다.

페이지의 운영 목표가 높은 방문율이라면 위 설정에 끝나지 않고
검색엔진별로 권장되는 설정에 대해 조금 더 깊이 알아볼 필요가 있습니다.
또한 각 검색엔진의 사이트 관리 페이지에서
운영중인 페이지가 어떻게 노출되고 있는지 확인하시길 바랍니다.

- [NAVER 웹마스터 - 사이트 관리](https://searchadvisor.naver.com/console/board)
- [Google - Search Console](https://search.google.com/search-console/welcome?hl=ko)


# Reference
---
[위키백과 SEO](https://ko.wikipedia.org/wiki/%EA%B2%80%EC%83%89_%EC%97%94%EC%A7%84_%EC%B5%9C%EC%A0%81%ED%99%94)

[SEO 가이드북](https://www.hedleyonline.com/ko/blog/seo-guide/)

[Google 검색 센터](https://developers.google.com/search?hl=ko)

[NAVER 웹 마스터 가이드](https://searchadvisor.naver.com/)

[Bing SEO 문서](https://www.bing.com/webmasters/help/webmasters-guidelines-30fba23a)