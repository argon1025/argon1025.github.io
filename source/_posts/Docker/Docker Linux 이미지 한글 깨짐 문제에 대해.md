---
title: Docker Linux 이미지 한글 깨짐 문제
category: Docker와 인프라, 미로 탐험기
tags:
	- Docker
	- MySQL
tech: Docker
techColor: blue
subTitle: 베이스 이미지 환경변수 설정
date: 2021-03-01
---

# 문제 원인
Docker를 사용할 때 MySQL character set 은 UTF-8로 설정되어 있음에도 불구하고
한글 데이터가 깨지는 현상이 간혹 발생하는데요

대부분 Docker의 베이스 이미지 환경변수를 설정해주면 해결됩니다


---



# 해결 방안

```yaml
Docker-compose.yml

Database_MySQL:
    image: mysql:5.7
    container_name: Database_MySQL
    environment:
      - LC_ALL=C.UTF-8 # LC_ALL 환경변수를 설정합니다.
```

`Docker-compose` 파일에서 LC_ALL 환경변수로 UTF-8 로케일을 설정합니다.


---



# Reference

> [# 도커(Docker) 컨테이너 로케일 설정](https://www.44bits.io/ko/post/setup_linux_locale_on_ubuntu_and_debian_container)