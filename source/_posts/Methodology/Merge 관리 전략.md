---
title: Merge 관리 전략 종류
subTitle: 대표적인 Merge 관리 전략에 대해
tech: Git
category: 개발방법론
tags:
	- Git
	- Merge
date: 2022-03-01
---

# 어떤 merge 관리 전략이 있나

---

대표적으로 Non-linear History 와 Linear History가 있습니다

## Merge `Non-linear History`

![image](https://user-images.githubusercontent.com/55491354/207640752-1d03a3a3-ae9b-4a57-82ca-1cd07df2f942.png)

a, b, c 커밋 브랜치를 유지하고 하나의 머지 커밋을 대상 브랜치에 추가합니다
커밋이 병합되어 하나의 머지 커밋으로 생성됩니다

## Squash and Merge `Linear History`

![image](https://user-images.githubusercontent.com/55491354/207640763-4ab5f6d7-1358-4de9-ac57-a2a6525f799f.png)

a, b, c를 합쳐서 새로운 커밋을 생성 후 대상 브랜치에 추가하고 기존 a, b, c 커밋 브랜치를 삭제합니다
새로 생성된 커밋의 Body에 기존 커밋 메시지 전체 기록을 저장합니다

## Rebase and Merge `Linear History`

![image](https://user-images.githubusercontent.com/55491354/207640790-ed1186ed-fe70-4ec8-82e8-c077080a281e.png)

a, b, c 커밋을 전체를 대상 브랜치에 추가하고 기존 a, b, c 커밋 브랜치 삭제합니다

# 결론

---

Linear History는 Non-Linear History와 달리 시행하는데 추가적인 리소스가 필요하지만
개발 커밋에 대한 내용을 차례대로 쉽게 추적할 수 있다는 장점이 있었습니다.

하지만 결국 상황에 따라서 각 브랜치 마다 merge 전략을 달리해야 하는데
대표적인 상황을 다음과 같이 상정했습니다.

- 개발에 참여하는 인원의 숙련도가 부족한 경우
  - Merge 사용
- 개발에 투자하는 자원과 시간이 한정적인 경우
  - Merge 사용
- 커밋 내역이 매우 많지만 히스토리에 모두 기록하지 않아도 될경우
  - Squash Merge 사용
- 커밋 내역을 히스토리에 보존해야하는 경우
  - Merge 또는 Rebase Merge 사용

위와 같이 다양한 상황에 따라서
Linear History, Non-Linear History를 사용할지 선택하는 것이 좋습니다

# Reference

---

[https://www.bitsnbites.eu/a-tidy-linear-git-history/](https://www.bitsnbites.eu/a-tidy-linear-git-history/)

[https://stackoverflow.com/questions/20348629/what-are-advantages-of-keeping-linear-history-in-git](https://stackoverflow.com/questions/20348629/what-are-advantages-of-keeping-linear-history-in-git)

[https://meetup.toast.com/posts/122](https://meetup.toast.com/posts/122)

[https://moonsupport.oopy.io/post/1](https://moonsupport.oopy.io/post/1)
