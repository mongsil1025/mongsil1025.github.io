---
title:  "[Vue.js 퀵스타트] Ch1 Vue.js 시작하기"
date: 2021-03-21
excerpt: ""
tags: [book, vuejs-quickstart]
classes: wide
categories: [vuejs-quickstart]
---

## Vue.js 란?

Vue.js란 Google Creative Lab에서 일하던 에반 유 (Evan you)가 UI를 빠르게 개발하기 위해서 만들었다. 그렇게 떄문에 Vue.js는 웹 화면 작성에 최적화된 프레임워크라고 할 수 있다.


### MVVM 아키텍처
- View : HTML 과 CSS로 작성하는 "화면"
- View Model : View의 실제 논리 및 데이터 흐름을 담당, ViewModel 의 상태 데이터만 변경하면 즉시 View에 반영된다.
- Model : 도메인 특화 데이터


## 개발 환경 설정


Vue.js를 학습하기 위해서는 아래와 같은 개발 환경이 필요하다.


- Node.js : 서버 측 자바스크립트 언어이자 플랫폼
- npm : 앱의 의존성 관리를 위해 사용하는 노드 패키지 관리자
- Visual Studio Code : 코드 편집 도구 (이클립스와 같은 IDE여도 상관없다.)
- **Vue-CLI** : Vue 앱 작성을 위한 기본 틀을 제공하는 도구


여기서 Vue-CLI 만 짚고 넘어가보겠다.

Vue-CLI는 커맨드라인 인터페이스 기반의 스캐폴딩(Scaffolding) 도구이다.

**스캐폴딩**이란 애플리케이션을 개발할 때 처음부터 개발하는 것이 아니라 기본적인 인터페이스와 툴을 제공해주는 것이다.

8장에서 자세히 다루겠지만, 이 도구는 프로젝트를 추가하면 빌드, 테스트 등을 손쉽게 볼수 있고 수행할 수 있는 것으로 보인다.

## Vue-CLI 설치

`node.js`를 설치하게 되면 `npm` 명령어를 사용할 수 있다.
Node.js 설치 공식 사이트 : [https://nodejs.org/ko/](https://nodejs.org/ko/)




## 첫 번째 Vue.js 애플리케이션

``` html
<!DOCTYPE html>
<html>

<head>
    <meta charset="utf-8">
    <title>hello vue.js</title>
    <script src="https://unpkg.com/vue@2.5.16/dist/vue.js"></script>
</head>

<body>

    <!-- 뷰 -->
    <div id="simple">
        <h2>{{message}}</h2>
    </div>

    <script type="text/javascript">
        // 모델 객체
        var model = {
            message: "몽실's 첫 번째 Vue.js 앱입니다.",
        };

        // 뷰모델 객체
        var simple = new Vue({
            el: '#simple', // HTML 요소
            data: model // 모델 객체 참조
        });
    </script>
</body>

</html>
```

여기서 알아야할 점은 뷰에 표현되는 데이터는 모델에 정의한 객체들이며, 모델과 뷰를 연결해주는 뷰모델을 통해 화면이 그려지는 것이다.


데이터가 변경되면 뷰모델 객체는 **즉시 HTML에 반영 된다.**

데이터, UI, 로직 이 세부분이 명시적으로 나뉘어져 있어서 복잡한 화면을 만들 때 굉장히 유용할 것 같다.
{: .notice--info}



## 콧수염 표현식

`<h2>{{message}}</h2>`

뷰.js 에서는 위와 같이 콧수염표현식으로 선언적으로 HTML DOM을 명시한다. 문자열을 덧붙인다는 의미로 보간법이라고도 불린다.
