---
layout: post
title: 실습 UI 개발을 통해 배워보는 JS & Vue JS (7) - 준비, 검색 폼 구현 (Vue.js)
category: Javascript
tags: [Javascript, Vue]
comments: true
---



> 본 강의는 Inflearn의 김정환 개발자 님의 강의([실습 UI 개발로 배워보는 순수 javascript 와 VueJS 개발]([https://www.inflearn.com/course/%EC%88%9C%EC%88%98js-vuejs-%EA%B0%9C%EB%B0%9C-%EA%B0%95%EC%A2%8C/dashboard](https://www.inflearn.com/course/순수js-vuejs-개발-강좌/dashboard)))를 듣고 배운 내용을 정리한 포스팅 입니다. 



## Vue.js (MVVM) 시작하기

#### 1) MVVM 패턴 

- Vue.js는 MVVM 패턴을 갖고 있음. MVVM 는 Model, View, View-Model으로 구성되어으며, M(Model)과 V(View)는 MVC 패턴의 M,V와 동일한 기능을 함.
- VM(View-Model)은 Model과 비슷하지만 조금 다른 역할을 하고 있음. 구조상으로 보면, Model과 View 사이에 위치하고 있음. Model로 부터 데이터를 갖고 오는데, 데이터는 View에 적합한 형태의 데이터로 가공됨.View-Model이 변경될 때마다 자동으로 연결되어있는 View 화면에 반영이 됨. Model보다 좀 더 적극적으로 뷰에 적합한 Model이라고 볼 수 있음. 
- 하나의 View에는 하나의 View-Model이 1:1로 매칭되어있음. (View가 많은 경우 여러개의 View-Model이 생성될 수 있음)



#### 2) 개발 환경 구성

- Vue.js CDN을 index.html에 추가

```html
  <script src="https://unpkg.com/vue"></script>
  <script type="module" src="./js/app.js"></script>
</body>
```

- app.js에 hello world 찍어보기

```html
<!-- index.html -->
<body>
  <div id="app">
    <header>
      <h2 class="container">검색</h2>
    </header>

    <div class="container">
      {{msg}}
    </div>
  </div>              
```

```javascript
// app.js

new Vue({
    el: '#app',
    data: {
        msg: 'hello world'
    }
})
```





## 검색폼 구현 (1)

> 검색 상품명 입력 폼이 위치한다. 

#### 1) index.html

- 검색 상품명 입력폼은 JS 디렉토리 내 index.html에서 구현했던 폼 태그를 사용

```html
<div id="app">
    <header>
        <h2 class="container">검색</h2>
    </header>

    <div class="container">
        <form>
            <input type="text" placeholder="검색어를 입력하세요" autofocus>
            <button type="reset" class="btn-reset"></button>
        </form>
    </div>
</div>
```



> 검색어가 없는 경우 X 버튼을 숨긴다 & 검색어를 입력하면 x 버튼이 보인다

#### 2) app.js

- Vue 인스턴스 data 내  입력을 담당하는 query 생성 (입력데이터를 받아서 저장하는 역할)
- query와 폼 태그의 입력값을 바인딩시키기 위해 `v-model` 사용
- 버튼도 입력데이터에 따라 보이고 숨기게 할 수 있음 => `v-show` 사용

```javascript
new Vue({
    el: '#app',
    data: {
        query: ''
    }
})
```

```html
<form>
    <input type="text" v-model="query" placeholder="검색어를 입력하세요" autofocus>
    <button type="reset" v-show="query.length" class="btn-reset"></button>
</form>
```



## 검색폼 구현(2)

> 엔터키를 입력하면 검색 결과가 보인다

#### 1) app.js & index.html

- form 태그에, submit 이벤트가 발생했을 때 동작하는 함수를 구현 시킬 수 있음. => `v-on`  사용
- `v-on`은 DOM에서 일어나는 이벤트를 listen하는 역할을 함.
- 그 역할을 하는 함수 `onSubmit` 생성. 함수는 Vue 인스턴스의 methods 파라미터 내 작성 (methods에서는 DOM과 바인딩할 함수를 정의 할 수 있음)

```html
<form v-on:submit="onSubmit"> 
    <input type="text" v-model="query" placeholder="검색어를 입력하세요" autofocus>
    <button type="reset" v-show="query.length" class="btn-reset"></button>
</form>
```

```javascript
new Vue({
    el: '#app',
    data: {
        query: ''
    },
    methods: {
        onSubmit(e) {
            debugger
        }
    }
})
```



#### 2) 이벤트 처리

- 위까지 코드를 작성한 후 서버를 실행시켜 검색어 입력 후 submit을 하면 breakpoint로 설정한 debugger에서 멈추는것을 알 수있음. 이때  디버거를 재개하면 화면이 갱신되는 것을 알 수 있음. 
- 여태것 화면 갱신을 막았던 방법을 사용하면 `onSubmit`에서 받은 이벤트에 `e.preventDefault()`로 구현을 했음 (as per plain JS). 그러나 Vue.js에서는 손쉽게 이벤트 처리를 할 수 있음. 
- 이벤트 종류 뒤에 `.prevent` 입력

```html
<div class="container">
    <form v-on:submit.prevent="onSubmit"> 
        <input type="text" v-model="query" placeholder="검색어를 입력하세요" autofocus>
        <button type="reset" v-show="query.length" class="btn-reset"></button>
    </form>
</div>
```



## 검색폼 구현(3)

> x 버튼을 클릭하거나 검색어를 삭제하면 해당 결과를 삭제한다.

#### 1) index.html

- 버튼에 click 이벤트를 listen 함.

```html
<button v-on:click="onReset" type="reset" v-show="query.length" class="btn-reset"></button>
```



#### 2) app.js

- methods 내 함수 구현

```javascript
    methods: {
        onSubmit(e) {
            // debugger
        },
        onReset() {
            this.query = ''
        },
    }
```

