---
title:  "[서버] 리눅스(unix)와 윈도우(cmd) 명령어 모음"
date: 2021-01-13
excerpt: ""
tags: [til, server]
categories: [til/server]
---

서버 개발자인데, 서버에 대한 명령어를 참 많이도 모른다.

회사에서 리눅스 명령어를 조금 익히니 이제 `cmd` 명령어가 익숙치 않다.

## 프로세스 관련

### 리눅스

- `ps -ef | grep mongo` : `mongo`로 수행중인 프로세스 찾기
- `kill -9 "taskId"` : "taskId" 프로세스 죽이기

### cmd

- `tasklist /fi "imagename eq 서비스이름"` : 서비스 이름으로 수행중인 프로세스 찾기
	- `tasklist /?` 를 하면 여러가지 옵션을 볼 수 있다.
	- ![화면 캡처 2021-01-13 224735](https://i.imgur.com/kY1zz5m.png)

- `taskkill /f /pid 프로세스 아이디` : 프로세스 아이디 죽이기
