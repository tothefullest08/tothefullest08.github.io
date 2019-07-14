---
layout: post
title: 실습 UI 개발을 통해 배워보는 JS & Vue JS (11) - Vue Component란? / Form Component 구현
category: Javascript
tags: [Javascript, Vue]
comments: true
---



> 본 강의는 Inflearn의 김정환 개발자 님의 강의([실습 UI 개발로 배워보는 순수 javascript 와 VueJS 개발]([https://www.inflearn.com/course/%EC%88%9C%EC%88%98js-vuejs-%EA%B0%9C%EB%B0%9C-%EA%B0%95%EC%A2%8C/dashboard](https://www.inflearn.com/course/순수js-vuejs-개발-강좌/dashboard)))를 듣고 배운 내용을 정리한 포스팅 입니다. 



## Vue Component 소개 

#### 1) 컴포넌트란?

- 컴포넌트란 화면의 구조를 모듈별로 나눈 것. 
- 화면은 header, section, sidebar 등 각각 작은 모듈로 쪼갤 수 있으며, 쪼개진 모듈을 트리처럼 묶을 수 있음.
- 아래 우측 사진의 최상단에 위치한 것이 전체 페이지를 의미하며 그아래 있는 것이 header와 본문과 sidebar등을 의미함.  그아래있는 것들이 본문, sidebar 등의 하위 모듈을 의미하며, 각각의 박스를 컴포넌트라고 함. 
- Vue.js는 컴포넌트를 손쉽게 구현할 수 있는 방법을 제공하고 있음.

![](img/component.png)

#### 2) 컴포넌트의 구성

- 파일의 확장자는 vue로 끝나며  .vue로 끝나는 파일을 컴포넌트 파일이라고 부름
- vue 디버깅을 쉽게 하기 위해 devtools 설치 (컴포넌트의 상태를 쉽게 확인 할 수 있음)

| 컴포넌트 명 | 분류     | 역할 |
| ----------- | -------- | -------- |
| HTML        | Template | 화면에 출력되는 부분 |
| JS          | Script   |  컴포넌트의 로직이 구현되는 부분|
| CSS         | Style          | 어떻게 표현될지(색상, 수치)의 정보가 들어감   |



지금까지 우리가 작업했던 Vue 코드를 하나 하나씩 component 로 바꿔보도록 하자.



## Form Component

> 입력 폼 자체를 하나의 Vue Component로 바꿔보도록 하자. 

#### 1) 기본 코드 작성(FormComponents.js)

- 입력 폼을 컴포넌트로 만들기 위해 js 폴더 밑에 component 폴더 생성 후 FormComponent.js 파일 생성

- FormComponents.js 안에서 객체 생성. 객체를 export 해줄 수 있도록 코드 작성 - `export default` 

- components는 바인딩될 DOM을 선택해야함.

- data 속성도 생성(객체를 return 할 수 있도록 return 입력)

- methods 생성

```javascript
export default {
    template: '#search-form',
    data() {
        return {
        }
    },
    methods: {
        
    }
}
```



#### 2) index.html 코드 수정

- index.html에서 form을 출력하는 코드를 따로 잘라 낸 후, 맨 아래에 template 태그를 만들어 붙임.
- template 태그의 id값으로, FormComponent.js에서 정한 값을 입력(`template: '#search-form'`)
- 이를 통해 FormComponent와 template 태그 및 태그 안의 든 코드는 서로 연결이 됨.

```html
<!-- 기존 코드 -->
<form v-on:submit.prevent="onSubmit">
    <input type="text" v-model="query" v-on:keyup="onKeyup" 
           placeholder="검색어를 입력하세요" autofocus>
    <button v-show="query.length" v-on:click="onReset" 
            type="reset" class="btn-reset"></button>
</form>

<!-- 변경된 코드 -->
<template id="search-form">
  <form v-on:submit.prevent="onSubmit">
    <input type="text" v-model="query" v-on:keyup="onKeyup" 
           placeholder="검색어를 입력하세요" autofocus>
    <button v-show="query.length" v-on:click="onReset" 
            ype="reset" class="btn-reset"></button>
  </form>
</template>
```



#### 3) FormComponents.js 내 변수 및 메서드 추가

- 컴포넌트에서 사용할 변수(View Model)을 정의
  - 위의 코드에서는 query 라는 1개의 변수만을 사용하고 있음.=> data 내에 변수 추가

- 코드 내 사용된 메서드 (`onSubmit` , `onKeyup`,  `onReset`) 또한 폼 컴포넌트의 methods에 정의

```javascript
export default {
    template: '#search-form',
    data() {
        return {
            query: ''
        }
    },
    methods: {
        onSubmit() {
        },
        onKeyUp() {
        },
        onReset() {
        },
    }
}
```



#### 4) Vue 인스턴스에 FormComponent 설정하기

- FormComponent.js는 모듈형식으로 구성하였기 때문에 app.js에서 사용 할 수 있음. 
- import 하여 Vue 인스턴스에서 사용을 하려면 components 속성에 key/value 형태로 추가해줘야함. 
- key에는 우리가 실제로 사용할 디렉티브 명을 입력하며,  value에는 불러온 FormComponent를 설정
- 실제로 FormComponent를 사용하기 위해서는 key로 입력한 `search-form`으로 사용해야함

```javascript
import FormComponent from './components/FormComponent.js'

new Vue({
  el: '#app',
  // 중략
  components: {
    'search-form': FormComponent
  },
```



- index.html에서 form 태그가 있었던 부분에 `<search-form></search-form>` 디렉티브를 입력하면, form 태그를 입력했던 것과 동일한 폼양식이 뜨는 것을 알 수 있음. 
- search-form 자체가 FormComponent를 의미함.

```html
    <div class="container">
      <search-form></search-form>
```



#### 5) props (parent -> child)

- query 변수는 실제 검색어 & 질의어를 나타냄. 이 질의어는 실제로 입력을 해서 설정할 수도 있지만, 추천 검색어나 최근검색어를 클릭하게 되면 클릭한 값이 입력값으로 설정하게도 해줘야함. 
- 따라서 query라는 변수는 FormComponent가 관리하기보다는 FormComponent인 상위 객체인 app.js에서 정의한 Vue 인스턴스에서 관리하는것이 더 적합함.  그러므로 app.js 내 query 값으로 관리하는 것이 낫음.
- FormComponent의 query 값을 app.js 의 query값으로 설정하는 방법이 필요
- index.html 에서 search-form 디렉티브를 사용하는데, 여기서 query값을 바인딩 해주면 됨 (v-bind 디렉티브 사용) 
- `<search-form v-bind:??="query">` : "bind의 어떤값을 Vue인스턴스의 query로 정하겠다"  라는 의미.
  - `??`로 표시한 부분은 search-form 컴포넌트 안에 있는 View Model이 될 것임. 이름이 중복되지 않게 value라고 설정

```html
<div class="container">
    <search-form v-bind:value="query"></search-form>
```

- value 라는 값을 통해서 Vue 인스턴스 즉,  search-form 외부로 부터 데이터를 받게 되어있음. 그러한 데이터는 `props` 라는 키를 통해서 받게 되어 있음. value로 받게 설정하였으므로, 배열 안에 문자열의 형태로 적어줌.

- 받아온 데이터를 실제 내부 데이터로 만들어 줘야하는데, query라는 변수명을 그대로 사용할 경우 혼동이 있으므로 inputValue로 변경해서 사용. 초기값으로 프로퍼티로 받았던 this.value로 설정

```javascript
export default {
    template: '#search-form',
    props: ['value'],
    data() {
        return {
            inputValue: this.value
        }
    },
```



- template 태그 내 사용한 변수명도 query -> inputValue로 변경

```html
<template id="search-form">
  <form v-on:submit.prevent="onSubmit">
    <input type="text" v-model="inputValue" v-on:keyup="onKeyup" 
           placeholder="검색어를 입력하세요" autofocus>
    <button v-show="inputValue.length" v-on:click="onReset" 
            type="reset" class="btn-reset"></button>
  </form>
</template>
```



#### 6) $emit() (child -> parent)

SearchForm 컴포넌트에서 엔터키를 쳐서 submit 이벤트가 발생했을 때 부모 컴포넌트에게 알려주는 작업 또한 필요함. (child -> parent). 이때 사용하는 메서드가 `$emit()` 임

- SearchForm 컴포넌트 내 `onSubmit` 이벤트에 대해서도 아래와 같이 코드를 작성하여 부모 컴포넌트에 알려 줄 수 있음.  첫번째 인자에는 app.js에서 사용할 이벤트명을 정의하고, 두번째 인자로는 함께 보낼 데이터를 입력
- search-form을 사용한 부분에서 v-on 디렉티브를 사용하여 이벤트와 연결시킬 수 있음.
- `@submit` 이벤트가 발생 했을 때, app.js에 있는 `onSubmit` 이 실행됨.
- app.js의 `onSubmit(query)` 함수의 `query`는  FormComponent.js 내 `onSubmit()` 함수의 `$emit`으로 넘어가는 2번째 인자인 `this.InputValue.trim()`을 의미함.

```javascript
//Form Components.js
methods: {
    onSubmit() {
        this.$emit('@submit', this.inputValue.trim())
    },
        
 //app.js
methods: {
     onSubmit(query) {
         this.query = query
         this.search()
     },
```

```html
<div class="container">
    <search-form v-bind:value="query" v-on:@submit="onSubmit"></search-form>
```



#### 7) 나머지 이벤트 수정

- reset 버튼을 클릭 했을 때, reset이라는 이벤트를 부모에게 전달해주고 부모가 처리하는 코드를 작성해보자.

```javascript
//FormComponents.js
methods: {
    onReset() {
        this.inputValue = ''
        this.$emit('@reset')
    },
        
 //app.js
methods: {
    onReset(e) {
      this.resetForm()
    },
```

```html
<search-form v-bind:value="query" v-on:@submit="onSubmit"
             v-on:@reset="onReset"></search-form>
```



- 폼 컴포넌트의 onKeyup 이벤트가 발생했을 때 로직을 작성해 보자.
  - 로직은 app.js 의 onKeyup 이벤트와 중복됨. 따라서 제거
  - 코드를 폼 컴포넌트의 onKeyup 메서드로 이동후 수정

```javascript
// app.js 내 onKeyup => 삭제할 것!
onKeyup(e) {
    if (!this.query.length) this.resetForm()
},
    
// FormComponents.js
    onKeyup() {
    if (!this.inputValue.length) this.onReset()
},   
```
