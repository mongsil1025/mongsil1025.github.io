---
title:  "[GIT] Access denied"
date: 2020-11-16
excerpt: ""
tags: [til, git]
categories: [til/git]
---

IntelliJ에서 gitlab을 연동해서 사용하던 중, 회사에서 주기적으로 비밀번호를 바꿔야해서 바꿨더니 `git push` 도 안되고 `git pull`도 안됬다.

<!--more-->

```
*** 로 마스킹 처리

오후 4:11	Update failed in ***
				remote: HTTP Basic: Access denied
				Authentication failed for '***'
				remote: HTTP Basic: Access denied
				Authentication failed for '***'

오후 4:11	Update canceled
```

<br/>
## 1. Password 탭 확인

<div class="code-example" markdown="1">
  - Go to Settings >> Appearance & Behavior >> System Settings >>Passwords
  - Change the setting to not store passwords at all
  - Invalidate and restart IntelliJ
  - Go to Settings >> Version Control >> Git >> SSH executable: Build-in
  - Do a fetch/pull operation
  - Enter the password when prompted
  - Again go to Settings>>Appearance & Behavior>>System Settings>>Passwords
  This time select store passwords on disk(protected with master password)
</div>

<div class="card mb-3">
    <img class="card-img-top" src="https://i.imgur.com/TnA5rGM.png"/>
    <!-- <div class="card-body bg-light">
        <div class="card-text">![blog_main]()
            The Peak District on a mosty morning.
        </div>
    </div> -->
</div>

위와 같이 Do not save 로 되어있어도 비밀번호가 업데이트가 안되었다 ;;  
<br/>

## 2. 제어판 > 사용자 계정 < 자격 증명 관리자에서 비밀번호 업데이트

![20201116_2](https://i.imgur.com/NYMVoyB.png)

해당 화면에서 편집을 눌러 비밀번호를 업데이트 해주면 된다.

---

[참고] [https://stackoverflow.com/questions/28142361/change-remote-repository-credentials-authentication-on-intellij-idea-14](https://stackoverflow.com/questions/28142361/change-remote-repository-credentials-authentication-on-intellij-idea-14)
