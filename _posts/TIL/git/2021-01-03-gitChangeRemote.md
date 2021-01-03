---
title:  "[GIT] UserName 변경 후 Git repository URL 변경"
date: 2021-01-03
excerpt: ""
tags: [til, git]
categories: [til/git]
---

## Git Username 변경

처음에 GIT 계정 생성 시 아무생각 없이 생성했던 계정이름이 마음에 들지 않아서 변경했다.

> github.com 접속 - Settings - Account - Change username

![git캡쳐](https://i.imgur.com/YG29Da2.png)

Change usernmae 버튼을 클릭하면 아래와 같은 경고문이 나온다.
- 프로필 페이지에 대한 리다이렉션을 생성하지 않는다.
- Page Site 에 대한 리다이렉션을 생성하지 않는다.

`github pages` 를 이용해서 블로그를 운영하고 있다면 기본적으로 username을 따서 해당 url로 블로그를 운영하고 있을 것이다. 계정이름을 변경하면 기존에 존재했던 블로그에 대한 redirect를 해주지 않는다.

글쓴이는 기존에 운영하고 있던 github page 블로그가 있어서, 해당 repository도 username과 동일하게 rename을 해주었다!

![Screenshot 2021-01-03 at 15.38.16](https://i.imgur.com/dsF7mN9.jpg)

## 로컬에서 작업하고 있던 Git repository URL 변경

username을 변경했으니, git url 경로 또한 당연히 변경됬을 것이다.

```
asis-user-name/repository -> tobe-user-name/repository
```

따라서 로컬에서 작업하고 있던 git url을 변경해줘야 한다.
[Changing a remote's URL](https://docs.github.com/en/free-pro-team@latest/github/using-git/changing-a-remotes-url)

`git bash`를 열고 로컬에서 작업하고 있는 경로로 이동
`git remote -v` 명령어 실행해서 현재 mounting 된 git URL 확인
![화면 캡처 2021-01-03 154306](https://i.imgur.com/HYxRbHX.png)

`git remote set-url` 명령어 실행, 아래 두가지 parameter 가 필요하다.
- An existing remote name. For example, `origin` or `upstream` are two common choices. (현재 remote name)
- A new URL for the remote (신규 remote URL)


![화면 캡처 2021-01-03 154428](https://i.imgur.com/h3suFIA.png)

---
참고문서
https://docs.github.com/en/free-pro-team@latest/github/using-git/changing-a-remotes-url
