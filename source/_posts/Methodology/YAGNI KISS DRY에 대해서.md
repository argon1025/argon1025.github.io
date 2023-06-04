---
title: YAGNI KISS DRY란
subTitle: 소프트웨어 개발 철학 YAGNI KISS DRY에 대해서
tech: Book
category: 개발방법론
tags:
	- YAGNI
	- KISS
	- DRY
date: 2022-03-01
---

이번 글에서는 YAGNI, KISS, DRY에 대해서 정리하고
해당 개념이 왜 나왔을까에 대해 개인적인 생각을 간단히 기록하고자 합니다

# YAGNI ( You Ain’t Gonna Need It )

---

정말 필요할 때까지 기능을 만들지 말라!

필요하다 판단될 때까지 기능을 추가하지 않는 것이 좋다는 프로그래밍 원칙입니다
이 원칙을 지키지 않는다면 불필요한 구현으로 인해 코드가 길어져서 분석이 어려워지고
개발하는 동안 더 쉬운 방법을 알게 되거나 해당 코드가 필요하지 않은 경우가 발생합니다

# KISS ( Keep it small and simple )

---

단순하게 해라!

코드는 작고 단순하게 유지 해야 한다는 프로그래밍 원칙입니다
코드를 작고 단순하게 작성하기 위해서는 먼저
비즈니스 로직의 기반 배경과 목적을 완벽히 이해했을 때 가능합니다
또한 다른 사람이 코드를 읽을 것을 항상 염두에 두고 바로 이해할 수 있도록 코드를 작성해야 합니다

# DRY ( Don't Repeat Yourself )

---

반복하지 마라!

같은 기능을 가진 코드를 반복하지 않아야 한다는 프로그래밍 원칙입니다.
개발팀 내에서 각 인원이 유일하게 구현한 코드도 있지만 공통으로 사용되는 함수나 코드가 있을 수 있습니다
각 인원이 해당 기능을 따로 구현하지 말고 하나의 컴포넌트로 분리해서 공유해야 합니다

# 무엇을 목표로 하는가?

---

![image](https://user-images.githubusercontent.com/55491354/207637051-ece53b49-246b-47a3-9cde-77103246f2bb.png)

`YAGNI` 원칙은 필요하지 않은 코드를 늘리지 않음으로 복잡도를 줄입니다.  
`KISS` 원칙은 로직을 단순화함으로써 복잡도를 줄입니다.  
`DRY` 원칙은 중복 로직을 없애서 복잡도를 줄입니다.

이렇게 보면 결국 우리가 습관적으로 코드를 청소하고 보이스카우트의 원칙을 지키려 노력하는 것과
유사해 보이는데 왜 이런 개념이 탄생하게 된 걸까요?

## 관념을 개념화 한다

---

> 개념은 여러 관념 중에서 공통적이고 일반적인 요소를 추출하고 종합하여 얻은 보편적인 관념을 말합니다

앞서 말했듯 어려운 개념들은 아닙니다.
좋은 개발자를 추구하고 있다면, 일정에 급히 치이고 있는 게 아니라면 항상 우리가 추구하려는 목표이자 관념이죠

우리는 항상 팀원과 함께 코드를 작성하기 때문에 팀원과 좋은 품질의 코드를 작성하기 위해서는
팀원 간 내 생각과 지식을 공유하는 것이 필수가 되지만 관념은 다른 사람과 공유하기 힘들죠
그래서 팀원 간 지식을 공유하기 위해서는 내 관념을 정확하게 구체화해서 개념화 하는 것이 필요합니다

위 소프트웨어 개발 철학( YAGNI, KISS, DRY )도 우리가 몸으로는 알고 있지만
공유되기 힘든 핵심 가치를 팀원 간에 공유하기 위해서 존재하는 개념인 것 같습니다

# Reference

---

[https://ko.wikipedia.org/wiki/KISS\_원칙](https://ko.wikipedia.org/wiki/KISS_원칙)

[https://ko.wikipedia.org/wiki/YAGNI](https://ko.wikipedia.org/wiki/YAGNI)

[https://www.linkapi.solutions/blog/yagni-kiss-e-dry](https://www.linkapi.solutions/blog/yagni-kiss-e-dry)

[https://hongjinhyeon.tistory.com/136](https://hongjinhyeon.tistory.com/136)
