---
title: JSDoc 사용하기
subTitle: JSDoc 알고쓰자
tech: NodeJS
category: NodeJS 사용하기
tags:
  - JsDoC
  - 개발방법론
date: 2022-02-01
---

JSDoc은 JavaDoc, phpDocumentor와 유사한 Javascript용 API문서 생성기입니다.
코드 자체에 주석을 추가하는 것으로 JSDoc 도구를 실행하면 자동으로 소스코드를 스캔하고 HTML 문서 사이트를 생성합니다

# 사용법

---

```jsx
/**
 * 함수에 대한 설명, 첫번째 줄일때는 생략이 가능하다
 * @author LeeseongRok <argon1025@gmail.com>
 * @version 1.0.0
 * @param categoryID 카테고리 아이디
 * @returns {object} 카테고리의 세부정보를 반환합니다
 * @throws {InvalidArgumentException} 매개변수가 올바르지 않을때 에러가 발생합니다.
 * @todo 리팩터링 필요!
 */
```

# Version을 정할때

---

소프트웨어 버전 할당 규칙에 따라
A.B.C 형식을 가지고 각 알파벳은 다음과같은 형식을 따른다

A → 기존 버전과 호환되지 않을때 숫자를 올린다
B → 기존 버전과 호환되면서 새로운 기능을 추가 했을때 숫자를 올린다
C → 기존 버전의 버그를 수정했을 경우 숫자를 올린다

# Reference

---

> [https://jsdoc.app/index.html](https://jsdoc.app/index.html)

> [https://tttsss77.tistory.com/57](https://tttsss77.tistory.com/57)

> [https://okayoon.tistory.com/entry/JSDoc를-사용해서-Javasript-문서화해보자](https://okayoon.tistory.com/entry/JSDoc를-사용해서-Javasript-문서화해보자)
