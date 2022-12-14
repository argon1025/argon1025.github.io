---
title: 그런 Branch 관리 전략으로 괜찮은가
subTitle: 체계적이고 빠르게 개발하는 방법에 대해
category: 개발방법론
tags:
	- Git
tech: git
techColor: orange
date: 2022-04-06
---

# 브랜치 관리 전략
---
버전 관리 시스템에는 대표적으로 SVN, Git 두 가지가 존재합니다
최근엔 Git을 주로 사용하는 것이 추세고 Git에서 제공하는 대표적인 기능인 브랜치(Branch) 기능을 통해
많은 사람과 체계적으로 개발할 수 있게 되었습니다.

그럼 Branch를 어떤 식으로 관리해야 많은 사람이 코드 베이스에 참여하더라도
체계적이고 빠르게 개발할 수 있을지
대표적은 브랜치 관리 전략에 대해 알아보고 이전엔 어떤 식으로 했는지 되돌아보는 시간을 가지고자 합니다



# Git flow
---
![image](https://user-images.githubusercontent.com/55491354/207642801-942fab1d-7519-468e-8671-b605c5df6589.png)
가장 최초로 제안된 브랜치 관리 전략 → [A successful Git branching model](https://nvie.com/posts/a-successful-git-branching-model/)
정기적으로 배포되는 대규모 프로젝트 관리에 적합합니다.
배포하기까지 많은 branch를 거쳐야 하므로 CI/CD를 지향하는 프로젝트에서는 사용하기 힘들다는 단점이 있습니다

## Master(main)
최종적으로 배포되는 브랜치 입니다

## Develop
개발을 진행하는 중심 브랜치 입니다
기능개발 브랜치( Feature )에서 개발을 진행하고 Develop 브랜치에 merge 합니다

## Feature
Develop 브랜치에서 생성되며 한 기능 단위를 구현하는 브랜치 입니다
기능 구현이 완료되면 develop 브랜치로 merge 됩니다

## Release
개발한 내용을 Master(main)에 배포 하기위해 준비(테스트, 버그수정)하는 브랜치 입니다

## HotFix
Master(main)에서 생성되는 브랜치로 배포된 소스에서 버그가 발생하면 생성하는 브랜치 입니다




# Github Flow
---
![image](https://user-images.githubusercontent.com/55491354/207642829-10f1d5b9-0e40-45ec-a68c-e54414fbdc79.png)
GitFlow를 개선하기 위해 제안된 방식으로
CI/CD를 염두하고 고안된 개념으로 PR을 적극적으로 사용합니다

## Master(main)
메인 브랜치 이며 기능, 오류 수정 모두 이곳에 merge되어 최신 상태를 유지합니다
따라서 Merge 전에 충분한 테스트 과정을 거쳐야 합니다

## 자유로운 브랜치 생성
GitFlow와 다르게 develop 브랜치가 없어 자유롭게 Master(main)에서 브랜치를 생성하고 작업합니다
단 작업이 종료되었을 경우 PR 및 코드리뷰를 거쳐야 합니다


# GitLab Flow
---
![image](https://user-images.githubusercontent.com/55491354/207642854-0a963674-119d-49b0-9b63-66ab5ab2f150.png)
GitFlow처럼 복잡하지 않으면서 GithubFlow처럼 너무 단순하지 않고 효율을 높이고자 생긴 브랜치 전략입니다

## Master(main)
GirFlow의 develop 브랜치와 동일한 역할을 합니다

## Feature
Master(main)에서 생성되는 브랜치로
모든 기능 개발을 진행합니다
기능 구현이 완료되면 PR후 Master(main)브랜치로 merge합니다

## Pre-production
Master(main) 와 Production 사이에 Staging을 위한 브랜치입니다
안정성과 배포시기 조절에 대한 부분을 이곳에서 조율합니다

## Production
Pre-production 브랜치에서 일방적으로 deploy 하는 브랜치로
이곳에서 코드가 배포됩니다




# 팀에 맞는 전략을 세우자
---
어떤 브랜치 관리 전략에 좋은가 라는 답은 없습니다
팀, 배포 목표에 따라 적합한 관리 전략을 선택하거나 새로운 전략을 수립해야 합니다
또한 언제든지 기존 전략에서 자기 팀에 맞는 전략으로 변경할 수 있습니다

[우아한형제들에서 git 브랜치 전략을 변경하게 된 글](https://woowabros.github.io/experience/2017/10/30/baemin-mobile-git-branch-strategy.html)




# Reference

---

[https://blog.gangnamunni.com/post/understanding_git_flow/](https://blog.gangnamunni.com/post/understanding_git_flow/)

[https://nvie.com/posts/a-successful-git-branching-model/](https://nvie.com/posts/a-successful-git-branching-model/)

[https://tecoble.techcourse.co.kr/post/2021-07-15-git-branch/](https://tecoble.techcourse.co.kr/post/2021-07-15-git-branch/)