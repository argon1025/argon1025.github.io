---
title: NGINX 로드밸런싱 서버 환경 구성
subTitle: NGINX로 nodeJS 로드밸런싱 구성하기
tech: NGINX
category: NGINX 사용하기
tags:
	- NGINX
	- Docker
	- NodeJS
date: 2021-08-06
---

# 용어 정리

## Nginx

이벤트 방식으로 응답하는 동시접속에 특화된 웹서버입니다
기본적인 서버로서의 역할도 할 수 있지만 [리버스 프록시](https://ko.wikipedia.org/wiki/%EB%A6%AC%EB%B2%84%EC%8A%A4_%ED%94%84%EB%A1%9D%EC%8B%9C)의 기능도 수행할 수 있습니다

## Load balancing

여러 서버 중 하나가 다운되더라도 다른 서버가 온라인을 유지할 수 있습니다
여러 서버에 사용자를 고르게 분산시키는 프로세스입니다

# 구현

![시스템 구성도](https://user-images.githubusercontent.com/55491354/207363007-1d2d77fb-8f9b-4a9e-84d7-f2055bf29179.png)

## 프로젝트 적용

Nginx 리버스 프록시 기능을 통해 NodeJS 컨테이너에 사용자를 나누어 요청을 보내
부하를 분산할 수 있도록 했습니다
Nginx에서 지속적인 Health check를 통해서 NodeJS 컨테이너가 종료되었을 경우
Backup 컨테이너로 요청을 보내도록 합니다

## Docker-compose.yml

```yml
version: "3"

services:
  Nginx:
    image: nginx:latest
    container_name: proxy
    volumes:
      - ./docker/nginx/nginx.conf:/etc/nginx/nginx.conf
    ports:
      - "80:80"
  express_1:
    build:
      context: ./app
    container_name: express_1
    environment:
      - NodeJS_PORT=8001
      - MYSQL_ROOT_PASSWORD=dbrootPassword
      - MYSQL_DATABASE=database
      - MYSQL_USER=dbuser
      - MYSQL_PASSWORD=dbuserPassword
    expose:
      - "8001"
  express_2:
    build:
      context: ./app
    container_name: express_2
    environment:
      - NodeJS_PORT=8002
      - MYSQL_ROOT_PASSWORD=dbrootPassword
      - MYSQL_DATABASE=database
      - MYSQL_USER=dbuser
      - MYSQL_PASSWORD=dbuserPassword
    expose:
      - "8002"
  express_3:
    build:
      context: ./app
    container_name: express_3
    environment:
      - NodeJS_PORT=8003
      - MYSQL_ROOT_PASSWORD=dbrootPassword
      - MYSQL_DATABASE=database
      - MYSQL_USER=dbuser
      - MYSQL_PASSWORD=dbuserPassword
    expose:
      - "8003"
  db:
    image: mysql:5.7
    container_name: Database
    environment:
      - MYSQL_ROOT_PASSWORD=dbrootPassword
      - MYSQL_DATABASE=database
      - MYSQL_USER=dbuser
      - MYSQL_PASSWORD=dbuserPassword
    volumes:
      - "./docker/mysql/mysql-data:/var/lib/mysql"
    expose:
      - "3306"
```

다중 컨테이너 애플리케이션을 정의하기 위해 Docker-Compose를 사용합니다
이 문서에서는 Nginx, NodeJS(Node1, Node2, backup_Node), MySQL 컨테이너를 정의했습니다.
NodeJS의 포트는 expose로 내부 컨테이너 끼리만 네트워크 공유가 가능하게 설정되었습니다

## nginx.conf 작성

```conf
worker_processes 4;
events { worker_connections 1024; }

http {

  upstream node-app {
    # 커넥션이 가장 적은 서버를 할당
    least_conn;

    # 클라이언트 IP 를 hash 해서 특정 클라이언트는 특정 서버로 연결하는 설정
    # ession clustering 이 구성되지 않은 경우
    # ip_hash;

    # weight
    # 업스트림 서버의 비중을 설정합니다 값을 2로 설정할경우 두배더 자주 선택됩니다

    # max_fails
    # n번 실패할경우 서버가 죽은것으로 간주합니다

    # fail_timeout
    # n초만큼 응답하지 않으면 죽은것으로 간주합니다
    server express_1:8001 max_fails=3 fail_timeout=5s;
    server express_2:8002 max_fails=3 fail_timeout=5s;

    # 1,2번 노드가 죽은 경우에 사용됩니다
    server express_3:8003 backup;
  }

  server {
    listen 80;

    location / {
      proxy_pass <http://node-app>;
      proxy_http_version 1.1;
      proxy_set_header Upgrade $http_upgrade;
      proxy_set_header Connection 'upgrade';
      proxy_set_header Host $host;
      proxy_cache_bypass $http_upgrade;
    }
  }
}
```

nginx 볼륨 설정으로 할당한 `./docker/nginx/` 경로에 nginx.conf 파일을 생성하고
위와같이 설정합니다

# 실제 서비스에 적용전에

무료 nginx에서는 로드밸런싱만 가능하고 헬스체크는 비활성화되어
문제가 있는 컨테이너를 서비스에서 제외하는 기능만 수행할 수 있습니다.
해결 방법에는 아래의 솔루션이 있습니다

- 돈을 지불하고 nginx plus를 사용할 수 있습니다.
- nginx 오픈소스 헬스체크모듈을 사용합니다. [**nginx_upstream_check_module**](https://github.com/yaoweibin/nginx_upstream_check_module)
- nginx 대신 AWS ELB를 사용합니다.

# Reference

---

Docker-compose
[https://docs.docker.com/compose/compose-file/compose-file-v3/](https://docs.docker.com/compose/compose-file/compose-file-v3/)

Docker-compose 에서 MySQL 초기화 하기
[https://pythonq.com/so/mysql/1520092](https://pythonq.com/so/mysql/1520092)

nginX 헬스체크
[https://blog.naver.com/sehyunfa/221694064329](https://blog.naver.com/sehyunfa/221694064329)

nginX 헬스체크 오픈소스 모듈
[https://github.com/yaoweibin/nginx_upstream_check_module](https://github.com/yaoweibin/nginx_upstream_check_module)
