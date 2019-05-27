---
layout: post
comments: true
title: Vuejs 입문
feature-img: "assets/img/jeju-bottom.jpg"
categories : [Javascript]
tags : [html, frontend, vuejs]
---

# Vue.js 입문

> Vue.js 의 초점은 더 많은 사람들이 쉽게 웹앱을 만드는데 있다.

## 역사

### 어쩌다?

- Evan You 가 개인 프로젝트로 시작
- Anguler.js 가 어려워서 단순하게 만들 만한 프레임워크 프로젝트 만들었다고 함.
- 이후 다른 여러곳에서 채택하면서 커짐.

### 왜?

- 프론트엔드 라이브러리가 MVC등등의 프레임워크의 크기로 제공되게 됨.
- 위에서 말한 Anguler.js, React 가 대표적인데 너무 어려움.
- Vue.js 는 일단 Anguler.js, React의 쉬운버전을 만들고자 하는 욕구가 출발이라고 함.

## 설계 사상

### 목표

- 처음에 인용구 처럼 더 많은 사람들이 쉽게 웹앱을 만드는게 목표.
- 엄청 간단하게 구현하기 위해서는 한 페이지에 몰아서 쉽게 짜도 되게끔 하는 부분부터, 복잡한 웹앱을 지원하기 위한 단계까지 가능하게끔 보여줌
- 유연성
  - MVVM패턴에서 영감을 받았지만 여러방법으로 개발 가능. 
  - 이게 progressive 구조로 확장됨.

### Progressive Programing

- 점진적 프로그래밍?
- 단계적 프로그래밍?

![Vue.js Progressiv FrameWork]({{site.url}}/assets/img/vue-progressive-diagram.png )

#### 단계

##### 1. 선언적 렌더링

- dom 렌더링, jQuery처럼 라이브러리 삽입.  mvvm에서는 View에 해당함.

```javascript
<script src="https://cdn.jsdelivr.net/npm/vue"></script>

<div id="app">
  {% raw %}{{ message }}{% endraw %}
</div>

var app = new Vue({
  el: '#app',
  data: {
    message: '안녕하세요 Vue!'
  }
})
```

##### 2. 컴포넌트 시스템

- 특정 dom 엘리먼트들의 구조를 컴포넌트화 해서 재사용 가능하게 함.
- 모든 컴포넌트의 본질은 확장된 Vue 인스턴스

```javascript
Vue.component('my-component', {
  template: '<span>{{ message }}</span>',
  data: {
    message: 'hello'
  }
})
```

##### 3. 클라이언트 사이드 라우팅

- url을 새로 받아오면서 서버에서 새로 렌더링 된 페이지들을 불러오는게 아닌, 하나의 페이지에서 동적으로 데이터를 변경하고 이동하는 작업들을 클라이언트에서 history에 저장해 뒀다가 옮겨오면서 작업하는 방식
- 하나의 페이지로 만들어진 웹앱을 (SinglePageApplication-SPA)라고 함.

##### 4. 대규모 상태관리

- Vuex, Flux 등 데이터의 상태 관리 패턴.
- Vuex, Flux는 해당 패턴을 도와주는 프레임워크.

![Vue.js state]({{site.url}}/assets/img/vue-state-vuex.png )

##### 5. 빌드 시스템

- Vue를 쉽게 빌드 하게 해주는 빌드 프레임워크들
- VueCLI를 활용한 단일 컴포넌트 등등

*vue file*

```html
<!-- .vue 파일 구조 -->
<template>
  <!-- html (뷰 컴포넌트의 표현단, 템플릿 문법) -->
</template>

<script>
  // 자바스크립트 (뷰 컴포넌트 내용)
</script>

<style>
  /* CSS (뷰 템플릿의 스타일링) */
</style>
```

6. 데이터 퍼시스턴스
    - rest API를 이용한 서버쪽 데이터, 또는 파일을 이용한 로컬 데이터 persistence 가 있음.

## 특징

- Progressive Programing철학과, 누구든 쉽게 웹앱을 만들게끔 하자는 취지가 만나 기본적인 script import 방식부터, 대규모 웹앱을 위한 점진적인 도구들 까지 단계별로 존재한다.
- 기존 레거시 html에 단순 script 삽입 방식으로 끼워 넣을 수도 있고, 처음부터 끝까지 Vue.js로 Vue.js 구조에 맞게 설계부터 시작하여 웹앱을 만들 수도 있다.
- MVVM구조로 양방향 데이터 바인딩 구조를 갖고있음
![MVVM Architect]({{site.url}}/assets/img/mvvm.png )
- 컴포넌트 구조로 단방향 이벤트 플로우를 갖고 있음.

### 사용 방법

- 기초 구조

```javascript
<div id="app-2">
  <span v-bind:title="message">
    내 위에 잠시 마우스를 올리면 동적으로 바인딩 된 title을 볼 수 있습니다!
  </span>
</div>

var app2 = new Vue({
  el: '#app-2',
  data: {
    message: '이 페이지는 ' + new Date() + ' 에 로드 되었습니다'
  }
})
```

### 설계구조

- MVVM 구조 사용.
  - MVC변형으로 MS의 WPF 프레임워크에서 밀던 패턴.
  - 리엑티브도 사용함. (리엑티브 프로퍼티 개념 차용)
  - 간단하게 쓸떈 안해도 됨
- MVVM지원을 위한 양방향 바인딩 개념 사용.

## 생명주기?

![Vue.js Life Cycle]({{site.url}}/assets/img/vue-lifecycle.png )

```javascript
<body>
<div id="app">
    <h1 v-if="!message">You must send a message for help!</h1>
    <my-component v-else v-bind:msg="message"></my-component>
    <textarea v-model="message"></textarea>
    <button v-show="message">
        Send word to allies for help!
    </button>
    <pre>
        {{ $data }}
    </pre>
</div>
</body>
<script src="https://cdnjs.cloudflare.com/ajax/libs/vue/2.5.17/vue.js"></script>



// 등록
Vue.component('my-component', {
  props: ['msg'],
  template: '<div>{{ msg }}</div>',
  beforeCreate: function() {
    console.log('beforecreated')
  },
  created: function() {
    console.log('created');
  },
  beforeMount: function() {
    console.log('beforemount')
  },
  mounted: function() {
    console.log('mounted')
  },
  beforeUpdate: function() {
    console.log('beforeupdate')
  },
  updated: function() {
    console.log('updated')
  },
    beforeDestroy: function() {
    console.log('beforedestroy')
  },
    destroyed: function() {
    console.log('destroyed')
  }
})

new Vue({
        el: '#app',
        data: {
             message: 'Our king is dead! Send help!'
        }
})
```