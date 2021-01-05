---
title:  "[Docker] 도커 명령어"
date: 2021-01-05
excerpt: ""
tags: [til, docker]
categories: [til/docker]
---

## Docker Pull

`docker pull <image name>` : docker hub으로부터 image를 다운받는다.
```
docker pull mongo
docker pull eses/mongodb_exporter
```

## images (image 목록 보기)

`docker images` : 현제 host 서버에 다운 받아져있는 image들을 출력한다.
``` console
docker images
REPOSITORY               TAG                 IMAGE ID            CREATED             SIZE
mongodb_exporter         latest              fdddd5dde0d3        7 hours ago         15.8MB
<none>                   <none>              b87580cdafbd        7 hours ago         481MB
golang                   alpine              53efefffaa70        6 days ago          299MB
rabbitmq1_rabbitmq       latest              927b3ab46bee        5 months ago        257MB
rabbitmq_rabbitmq        latest              927b3ab46bee        5 months ago        257MB
rabbitmq                 latest              4c8cb17c3ab5        9 months ago        151MB
ubuntu                   latest              72300a873c2c        9 months ago        64.2MB
test-consumer            latest              d731ca487638        13 months ago       1.3GB
nodejs-pm2-application   latest              f0eade772333        13 months ago       1.3GB
crwaler-test             latest              6a0021cf1d10        14 months ago       1.27GB
mongo                    latest              58477a771fb4        14 months ago       361MB
rabbitmq                 management          4af5c5534d00        14 months ago       180MB
centos                   latest              0f3e07c0138f        14 months ago       220MB
node                     10                  636ef87129d6        15 months ago       903MB
alpine                   3.4                 b7c5ffe56db7        22 months ago       4.81MB
rnbwd/sinopia            latest              14038a0d48a2        3 years ago         114MB
```

## run (컨테이너 생성과 동시에 컨테이너로 접속)

`docker run <옵션><이미지이름><실행할파일>` : 단순 image 안의 파일을 실행하 목적으로 생성된것이기 때문에 **메인으로 실행되는 파일이 종료되면 컨테이너도 같이 종료된다. 따라서, 계속해서 컨테이너를 유지하고 싶다면 `-d` 옵션을 이용해야 한다**

- 옵션
  - i (interactive) : 사용자가 입출력을 할 수 있는 상태로 한다   
  - t : 가상 터미널 환경을 에뮬레이션 하겠다는 말   
  - **d : 컨테이너를 일반 프로세스가 아닌 데몬프로세스 형태로 실행하여 프로세스가 끝나도 유지되도록 한다**

- 몽고 이미지 빌드

    ``` console
    docker run --name mongodb-server -v /etc/mongod.conf:/etc/config/mongod.conf -d mongo --config /etc/config/mongod.conf
    ```

    [몽고 서버 띄울 때..] 이렇게하면 안된다.. `--config` 를 지우면 띄어 지는데,, 왜 안되는 지 모르겠다.
    {: .notice--warning}

## exec

만들어진 이미지 서버에, 접속한다.
``` console
docker exec -it mongodb_server bash
```

## ps (컨테이너 확인(실행중인 image 확인))

- `docker ps`
  - 실행중인 컨테이너의 목록을 확인한다

- `docker ps -a`
  - 이전에 종료되었던 컨테이너들을 포함한 컨테이너의 목록을 확인한다

## logs (컨테이너 로그를 확인한다.)
``` console
$ docker logs mysql
Initializing database
2019-01-14T10:18:41.333808Z 0 [Warning] TIMESTAMP with implicit DEFAULT value is deprecated. Please use --explicit_defaults_for_timestamp server option (see documentation for more details).
```

---
[참고자료]

- [https://captcha.tistory.com/49](https://captcha.tistory.com/49)
- [https://elfinlas.github.io/2019/02/11/docker-on-mongo/](https://elfinlas.github.io/2019/02/11/docker-on-mongo/)
- [컨테이너실행](https://subicura.com/2017/01/19/docker-guide-for-beginners-2.html)
- [도커기본정보](https://subicura.com/2017/01/19/docker-guide-for-beginners-2.html)
