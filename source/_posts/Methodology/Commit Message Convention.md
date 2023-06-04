---
title: Commit Message Convention
subTitle: 사람과 기계가 모두 식별할 수 있는 커밋 메시지
tech: Git
category: 개발방법론
tags:
	- Git
date: 2022-03-01
---

# Commit Message를 작성하는 이유

---

우리는 기능 구현을 위해서 브랜치를 만들고
브랜치 안에서 작은 단위의 코드 블록을 작성하고 Commit을 하게 됩니다
Commit Message는 이후 해당 코드블럭에 대해 이해를 돕는 시야와
당시의 설계 의도 ( 마인드셋 )을 파악하는 중요한 정보를 제공하는 역할을 하게 됩니다.

# Conventional Commits이란 무엇인가

---

Commit Message 컨벤션 중 하나로 명확한 커밋 히스토리를 작성하기 위한 규칙을 정의합니다
통일된 컨벤션을 통해서 서로가 작성한 커밋의 의도를 명확하게 파악할 수 있게 됩니다.

## Commit Message 구조

```
<타입>[적용 범위(선택 사항)]: <설명>

[본문(선택 사항)]

[꼬리말(선택 사항)]
```

## Commit Message 타입

`fix` → 코드 베이스 버그 패치
`feat` → 코드 베이스 기능 추가
`BREAKING CHANGE` → 메이저 업데이트, 이전 API와 호환되지 않을 경우
`style` → 코드 베이스 스타일 변경
`docs` → 문서 변경

기타

```
build: 시스템 또는 외부 종속성에 영향을 미치는 변경사항 (npm, gulp, yarn 레벨)
ci: ci구성파일 및 스크립트 변경
chore: 패키지 매니저 설정할 경우, 코드 수정 없이 설정을 변경
perf: 성능 개선
refactor: 버그를 수정하거나 기능을 추가하지 않는 코드 변경, 리팩토링
test: 누락된 테스트 추가 또는 기존 테스트 수정
revert: 작업 되돌리기
```

# Conventional Commits 적용하자

---

![image](https://user-images.githubusercontent.com/55491354/207640134-c436ac42-9a6d-4bc4-a6c8-b260fad0bbf9.png)

이전 커밋 메시지에 대해서 문제점을 파악하고
다음과 같은 목표를 설정하고 커밋 메시지 컨벤션을 통일했습니다

- 하나의 커밋은 하나의 기능에 대응한다
- 커밋은 누가 보더라도 이해하기 쉽게, 아토믹(Atomic) 해야한다
- 커밋 메시지는 다른 사람이 보더라도 해당 작업내용을 파악할 수 있게 작성한다
- Conventional Commits 을 지킨다.

# 나는 저자고 미래의 나와 팀원은 독자다

---

클린코드 첫 장에서 `우리는 코드를 적는 저자`라고 소개합니다
그리고 저자는 독자와 잘 소통할 책임을 진다는 것을 기억해야 한다 말합니다.

우리가 코드를 읽는 과정에서 들이는 시간과 노력을 조금이라도 덜기 위해서
이런 세세한 부분에서 복잡도를 줄이고 지속해서 개선해 나가야 한다고 생각합니다

# Reference

---

[https://www.conventionalcommits.org/ko/v1.0.0/](https://www.conventionalcommits.org/ko/v1.0.0/)

[https://medium.com/hdackorea/commit-history를-효과적으로-관리하기-위한-규약-conventional-commits-67b2114ac8e4](https://medium.com/hdackorea/commit-history를-효과적으로-관리하기-위한-규약-conventional-commits-67b2114ac8e4)
