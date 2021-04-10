---
title:  "[Web] Maven의 Life Cycle"
date: 2021-04-10
excerpt: ""
tags: [til, spring]
categories: [til/spring]
---

Maven의 Life Cycle에 대해 잘 정리해져 있는 글이 있어서 정리합니다.
[https://mangkyu.tistory.com/8](https://mangkyu.tistory.com/8)

---

Maven은 프레임워크이기 때문에 동작 방식이 정해져 있습니다.

즉, 미리 정의된 빌드순서를 라이프사이클이라 하고, 빌드 단계를 Phase라고 합니다.

일반적으로 메이븐은 3개의 표준 라이프사이클을 제공합니다.

- Clean : 빌드 시 생성되었던 Output을 지워준다.
- Default(Build) : 일반적인 빌드 프로세스를 위한 모델이다.
- Site : 프로젝트 문서와 사이트 작성을 수행한다.

![Maven Build LifeCycle](https://t1.daumcdn.net/cfile/tistory/993CC83359FB369723)

## Jenkins에서 프로젝트 빌드 로그 확인

Maven이 어떻게 동작하는지 좀 더 확인하기 위해, 회사 프로젝트 빌드 로그를 봤습니다. (Jenkins 빌드 툴 사용)

### 1. dependency를 가지는 resource download

``` console
Downloaded: xxxxxx/0.1.1-SNAPSHOT/maven-metadata.xml (996 B at 97.3 KB/sec)
Downloaded: xxxxxx/0.1.1-SNAPSHOT/maven-metadata.xml (996 B at 69.5 KB/sec)
```

### 2. `maven-clean`

``` console
[INFO] --- maven-clean-plugin:2.4.1:clean (default-clean) @ ---
[INFO] Deleting xxx/target
[INFO]
```

### 3. `maven-resources`

``` console
[INFO] --- maven-resources-plugin:2.5:resources (default-resources) @ ---
[debug] execute contextualize
[INFO] Using 'UTF-8' encoding to copy filtered resources.
[INFO] Copying 676 resources
[INFO] Copying 36 resources
[INFO]
```

### 4. `maven-compile`

``` console
[INFO] --- maven-compiler-plugin:3.7.0:compile (default-compile) @  ---
[INFO] Changes detected - recompiling the module!
[INFO] Compiling 3388 source files to xxxxx/target/classes
```

### 5. `maven-test`

``` console
[INFO] --- maven-resources-plugin:2.5:testResources (default-testResources) @  ---
[debug] execute contextualize
[INFO] Using 'UTF-8' encoding to copy filtered resources.
[INFO]
[INFO] --- maven-compiler-plugin:3.7.0:testCompile (default-testCompile) @ ---
[INFO] Changes detected - recompiling the module!
[INFO] Compiling 2 source files to xxxx/target/test-classes
[INFO]
[INFO] --- maven-surefire-plugin:2.10:test (default-test) @ ---
[INFO] Surefire report directory: xxxx/target/surefire-reports
```

### 6. `pom.xml`에 기재되어 있던 plugin 수행

`maven-jar-plugin`, `maven-shade-plugin` 수행

``` console
-------------------------------------------------------
 T E S T S
-------------------------------------------------------

Results :

Tests run: 0, Failures: 0, Errors: 0, Skipped: 0

[JENKINS] Recording test results
[INFO]
[INFO] --- maven-jar-plugin:2.3.2:jar (default-jar) @ ---
[INFO] Building jar: project.jar
[INFO]
[INFO] --- maven-shade-plugin:3.2.1:shade (default) @ ---
[INFO] Including xxx:jar:0.0.89-SNAPSHOT in the shaded jar.
[INFO] Including xxxxxx:jar:0.0.6 in the shaded jar.
...
[INFO] --- maven-assembly-plugin:2.5.1:single (default) @ ssg-batch-app ---
```

### 7. `maven-install`

로컬 repository에 package 저장

``` console
[INFO] --- maven-install-plugin:2.3.1:install (default-install) @  ---
```

보통 `package`를 해서 jar, war 등으로 만들어주는 작업을 하는데, 우리 프로젝트는 `maven-assembly` 플러그인으로, 특정 파일들을 디렉터리로 구성해서 같이 패키징을 하도록 설정을 해놨다.

### 8. 후속 처리

빌드 후에 후속 처리로 서버를 UP/DOWN 하거나, 생성된 jar 파일의 이름을 변경하는 등의 작업을 jenkins에서 등록할 수 있다.
