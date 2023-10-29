---
title: 자주 사용하는 Docker 명령어들
subTitle: 📜 Docker CLI 정리
tech: Docker
category: Docker 사용하기
tags:
  - Docker
date: 2021-02-02
---

Docker(도커)는 컨테이너 기반의 오픈소스 가상화 플랫폼입니다
다양한 프로그램과 실행환경을 YAML 파일로 관리할 수 있으며
서버 관리 및 배포를 단순하게 해주고 있습니다

# Docker 설치
---

도커는 [링크](https://www.docker.com/get-started)를 통해 설치할 수 있습니다

# Docker 버전 확인하기
---

```bash
docker version
```

설치된 docker 서버와 클라이언트 정보를 조회합니다



# Docker pull 명령어로 ubuntu 이미지 설치하기
---

```bash
docker pull ubuntu:latest
```

컨테이너를 실행하기 위한 이미지파일을 다운받을 수 있습니다

`ubuntu:latest` 는 최신 LTS Ubuntu 이미지를 말합니다



# Docker images 명령어로 설치된 이미지 확인하기
---

```bash
docker images
```

```bash
REPOSITORY               TAG       IMAGE ID       CREATED          SIZE
ubuntu                   latest    f63181f19b2f   3 weeks ago      72.9MB
```

`docker images` 명령어로 설치했거나 빌드한 이미지를 확인할 수 있습니다



# Docker run 명령어로 ubuntu 설치해보기
---

```bash
docker run --rm -it ubuntu:latest /bin/bash
```

ubuntu 컨테이너를 실행합니다 `ubuntu:latest` 는 최신 LTS Ubuntu 이미지를 말합니다 이미지에 대한 자세한 사항은 [Dock Hub](https://hub.docker.com/_/ubuntu)에서 보실 수 있습니다

## 자주 사용하는 Docker run 옵션

`-it` (interactive),(tty) 컨테이너 안에서 터미널을 실행해서 명령어를 입력할때 사용하는 옵션
`-p 8080:80` 도커호스트 포트:컨테이너 내부 포트, 옵션을 여러개 줘서 여러 포트를 동시에 열 수 있습니다
`--name` 컨테이너의 이름을 설정합니다
`-v` 호스트 컨테이너와 디렉토리를 마운트 합니다
`-rm` 컨테이너 프로세스가 종료되면 자동으로 컨테이너를 제거합니다
`-d` 백그라운드 모드로 실행합니다



# Docker ps 명령어로 실행된 컨테이너 확인하기
---

```bash
docker ps
```

```bash
CONTAINER ID   IMAGE           COMMAND       CREATED         STATUS         PORTS     NAMES
1ed35f870d27   ubuntu:latest   "/bin/bash"   7 seconds ago   Up 6 seconds             vigilant_mestorf
```

`docker ps` 명령어로 현재 실행되고 있는 컨테이너 리스트를 볼 수 있습니다

```bash
docker ps -a
```

```bash
CONTAINER ID   IMAGE           COMMAND       CREATED          STATUS                     PORTS     NAMES
0795d80d1abd   ubuntu:latest   "/bin/bash"   11 seconds ago   Exited (0) 7 seconds ago             zen_dubinsky
1ed35f870d27   ubuntu:latest   "/bin/bash"   2 minutes ago    Up 2 minutes                         vigilant_mestorf
```

`docker ps -a` 명령어로 실행,종료된 모든 컨테이너의 리스트를 볼 수 있습니다



# Docker stop 명령어로 컨테이너 중지하기
---

```bash
docker stop ${CONTAINER ID}
```

`docker ps -a` 명령어로 `CONTAINER ID` 를 확인한뒤 `docker stop` 를 통해 컨테이너를 중지할 수 있습니다
띄어쓰기로 여러개의 컨테이너를 동시에 종료할수도 있습니다



# Docker rmi 명령어로 이미지 제거하기
---

```bash
docker rmi ${IMAGE ID}
```

```bash
docker rmi f63181f19b2f
Untagged: ubuntu:latest
Untagged: ubuntu@sha256:703218c0465075f4425e58fac086e09e1de5c340b12976ab9eb8ad26615c3715
Deleted: sha256:f63181f19b2fe819156dcb068b3b5bc036820bec7014c5f77277cfa341d4cb5e
Deleted: sha256:0770b7f116f8627ec336a62e65a1f79e344df7ae721eb3e06e11edca85d3d1e7
Deleted: sha256:476e931831a5b24b95ff7587cca09bde9d1d7c0329fbc44ac64793b28fb809d0
Deleted: sha256:9f32931c9d28f10104a8eb1330954ba90e76d92b02c5256521ba864feec14009
```

`docker images` 명령어로 이미지 목록을 조회해서 `IMAGE ID` 를 알아내면
`docker rmi` 명령어를 사용하여 사용하지 않는 이미지를 제거할 수 있습니다
이미지를 제거하려면 반드시 관련 컨테이너가 모두 삭제되어야 합니다



# Docker logs 명령어로 컨테이너 로그 보기
---

```bash
docker logs ${CONTAINER ID}
```

```bash
docker ps
CONTAINER ID   IMAGE                      COMMAND                  CREATED          STATUS          PORTS                  NAMES
d9811a26e5de   argon1025/node-web-app:1   "docker-entrypoint.s…"   59 seconds ago   Up 59 seconds   0.0.0.0:80->8080/tcp   cranky_pascal

docker logs d9811
Running on <http://0.0.0.0:8080>
```

`docker ps` 명령어로 컨테이너 ID를 얻은 다음 해당 컨테이너에 기록된 Log를 `docker logs` 명령어로 열람할 수 있습니다
위는 nodejs 컨테이너를 실행하고 로그를 조회한 결과입니다

## 자주 사용하는 Docker logs 옵션

`-f` 옵션으로 로그를 실시간으로 볼 수 있습니다
`--tail` 옵션으로 마지막 10줄만 출력할 수 있습니다



# Docker exec 명령어로 컨테이너에 명령어 실행하기
---

```bash
docker exec -it ${CONTAINER ID} /bin/bash
```

`docker ps` 명령어로 컨테이너 ID를 얻은 다음 `docker exec` 명령어로 해당 컨테이너에 접속할 수 있습니다
`SSH` 는 권장하지 않고 반드시 이방법으로만 컨테이너에 접속해야합니다
