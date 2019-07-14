---
layout: post
title: 실습 UI 개발을 통해 배워보는 JS & Vue JS (12) - Result & List Component 구현
category: Javascript
tags: [Javascript, Vue]
comments: true
---



> 본 강의는 Inflearn의 김정환 개발자 님의 강의([실습 UI 개발로 배워보는 순수 javascript 와 VueJS 개발]([https://www.inflearn.com/course/%EC%88%9C%EC%88%98js-vuejs-%EA%B0%9C%EB%B0%9C-%EA%B0%95%EC%A2%8C/dashboard](https://www.inflearn.com/course/순수js-vuejs-개발-강좌/dashboard)))를 듣고 배운 내용을 정리한 포스팅 입니다. 



## Result Component 구현

- ResultComponent.js 작성

```javascript
export default {
    template:'#search-result'
}
```



- index.html내 검색 결과를 나타내는 태그를 잘라내 파일 하단으로 이동 후 template 태그로 묶어줌

```html
<div class="content">
    <div v-if="submitted">
        <!-- 코드 이동 -->
    </div>
    
<template id="search-result">
  <div v-if="searchResult.length">
    <ul>
      <li v-for="item in searchResult">
        <img v-bind:src="item.image"> {{item.name}}
      </li>
    </ul>
  </div>
  <div v-else>
    {{query}} 검색어로 찾을수 없습니다
  </div>
</template>
```



- app.js 내 컴포넌트 import 후 components에 추가

```javascript
import ResultComponent from './components/ResultComponent.js'

  components: {
    'search-result' : ResultComponent
  },
```



- index.html에 컴포넌트를 기존의 위치에 입력
  - Vue 인스턴스가 갖고 있는 검색 결과 값을 v-bind를 이용하여  ResultComponent에 넘겨 줄 것.
  - ResultComponent에서 props로 data를 받음
  - 결과값을 뿌려주는 부분에 대한 변수명을 변경(`search.length` -> `data.length`)

```html
<div class="content">
    <div v-if="submitted">
        <search-result v-bind:data="searchResult"></search-result>
    </div>
```

```javascript
export default {
    template:'#search-result',
    props: ['data'],
}
```

```html
<template id="search-result">
  <div v-if="data.length">
    <ul>
      <li v-for="item in data">
        <img v-bind:src="item.image"> {{item.name}}
      </li>
    </ul>
  </div>
  <div v-else>
    {{query}} 검색어로 찾을수 없습니다
  </div>
</template>
```



- 검색어(query) 또한 ResultComponent로 넘겨줌 

```html
<div class="content">
    <div v-if="submitted">
        <search-result v-bind:data="searchResult" v-bind:query="query"></search-result>
    </div>
```

```javascript
export default {
    template:'#search-result',
    props: ['data', 'query'],
}
```



## List Component 구현

추천 검색어와 최근검색어는 형식이 유사함. 이들을 출력할 수 있는 ListComponent를 만들어 공유해서 쓰도록 하자.

#### 1) 기본 코드 작성

- index.html를 보면 분기문에 따라 추천 검색어  또는 최근 검색어 코드가 작성되어있음. 둘중 하나를 써서 공유할 에정이므로,  하나만 잘라내어 파일 하단에 template 태그 안으로 이동
- v-if & v-for의 변수는, 추천/최근 검색어에 공통으로 쓰므로, 변수명을 수정
  - `keywords.length` -> `data.length`
  - `(item, index) in keywords` -> `(item, index) in data`
  - `onClickKeyword` -> `onClickList`
- 최근 검색어 목록에만 존재하는 날짜와 삭제 버튼에 대한 내용도 추가로 갖고온 후, 변수명 또한 수정
  - `onClickRemoveHistory` -> `onRemoveList`

```javascript
// ListComponent
export default {
    template: '#List'
}
```

```html
<!-- index.html -->
<div v-if="selectedTab === tabs[0]">
    <!-- 코드 이동 -->
</div>

<template id="List">
  <div v-if="data.length">
    <ul class="list">
      <li v-for="(item, index) in data" v-on:click="onClickList(item.keyword)">
        <span class="number">{{index + 1}}</span> 
        {{item.keyword}}
        <span class="date">{{item.date}}</span>
        <button class="btn-remove" 
                v-on:click.stop="onRemoveList(item.keyword)"></button>  
      </li>
    </ul>
  </div>
  <div v-else>
    추천 검색어가 없습니다
  </div>
</template>
```



- app.js 내 컴포넌트 import 및 등록

```javascript
import ListComponent from './components/ListComponent.js'

  components: {
    'search-form': FormComponent,
    'search-result' : ResultComponent,
    'list' : ListComponent,
  },
```



- index.html 내 추천 검색어 & 최근 검색어를 출력하는 부분을 수정
  - list 디렉티브 사용 & v-bind를 이용하여 Vue 인스턴스가 갖고 있는 값을 ListComponent로 넘김

```html
<div v-if="selectedTab === tabs[0]">
    <list v-bind:data="keywords"></list>
</div>

<div v-else>
    <list v-bind:data="history"></list>
</div>
```



- Vue 인스턴스로부터 받아온 값을 props에 저장 & methods 내 기본 함수 틀 생성

```javascript
export default {
    template: '#List',
    props: ['data'],
    methods: {
        onClickList(keyword) {

        },
        onRemoveList(keyword) {
            
        }
    }
}
```



#### 2) 결과 화면 다듬기

- 추천 검색어 리스트에는 불필요한 기능(날짜, x버튼)이 나타나고 있음. 리스트가 추천 검색어인지 최근 검색어인지 식별할 수 있는 방법이 필요함.  => `type` 데이터 사용 & `props`에 추가

```html
<!-- index.html -->
<div v-if="selectedTab === tabs[0]">
    <list v-bind:data="keywords" type="keywords"></list>
</div>
<div v-else>
    <list v-bind:data="history" type="history"></list>
</div>
```

```javascript
// ListComponent.js
export default {
    template: '#List',
    props: ['data', 'type'],
}
```

- type의 값에 따라 인덱스 & 날짜 & x버튼을 출력하고 감추게 분기문 작성

```html
<!-- index.html -->
<li v-for="(item, index) in data" v-on:click="onClickList(item.keyword)">
    <span v-if="type === 'keywords'" class="number">{{index + 1}}</span> 
    {{item.keyword}}
    <span v-if="type === 'history'" class="date">{{item.date}}</span>
    <button v-if="type === 'history'"class="btn-remove" 
            v-on:click.stop="onClickRemoveList(item.keyword)"></button>
</li>
```



#### 3) 추천 검색어 클릭 시, 검색 기능 동작 구현

- ListComponent.js 에서 클릭 이벤트가 발생했을 때, 이벤트를 재정의 하여 app.js로 넘겨주면 됨. `$emit` 사용
- 최근 검색어의 경우, 검색어를 삭제하는 이벤트에 대한 코드를 추가로 입력

```javascript
methods: {
    onClickList(keyword) {
        this.$emit('@click', keyword)
    },
    onRemoveList(keyword) {
        this.$emit('@remove', keyword)
    }
}
```

```html
<div v-if="selectedTab === tabs[0]">
    <list v-bind:data="keywords" type="keywords" v-on:@click="onClickKeyword"></list>
</div>
<div v-else>
    <list v-bind:data="history" type="history" v-on:@click="onClickKeyword"
          v-on:@remove="onClickRemoveHistory"></list>
</div>
```



#### 4) Computed

- type값이 keyword 인지, 혹은 history인지에 따라서 출력하는 부분이 달라짐. 
- template 태그 안에 조건문 같은 코드가 있을 경우,  코드의 가독성이 떨어짐. 이때 Vue에서 제공하는 computed를 사용.

```html
<span v-if="type === 'keywords'" class="number">{{index + 1}}</span> 
{{item.keyword}}
<span v-if="type === 'history'" class="date">{{item.date}}</span>
```



- ListComponent.js 내 computed 속성 추가
  - type이 keywords 인지 history 인지를 반환하는 함수 작성

```javascript
computed: {
    keywordType() {
        return this.type === 'keywords'
    },
    historyType() {
        return this.type === 'history'
    },
},
```

- computed 내 함수를 호출하여 사용

```html
<div v-if="data.length">
    <ul class="list">
      <li v-for="(item, index) in data" 
          v-on:click="onClickList(item.keyword)">
        <span v-if="keywordType" class="number">{{index + 1}}</span> 
        {{item.keyword}}
        <span v-if="historyType" class="date">{{item.date}}</span>
        <button v-if="historyType"class="btn-remove" 
                v-on:click.stop="onRemoveList(item.keyword)"></button>
      </li>
    </ul>
  </div>
  <div v-else>
    <span v-if="keywordType">추천 검색어가 없습니다</span>
    <span v-if="historyType">최근 검색어가 없습니다</span>    
  </div>
```



#### 5) watch - 추천 검색어를 검색폼에 띄우기

추천 검색어를 클릭하여 결과가 뜬 후, devtools를 통해 디버깅을 해보면,  아래와 같은 결과를 얻을 수 있다.  검색폼에 클릭한 추천 검색어가 뜨지 않는 이유는 searchForm 내 `inputValue` 값이 저장되어 있지 않았기 때문임.

- \<Root>   
  - data - query: "이탈리아"
- \<SearchForm>   
  - props - value: "이탈리아"
  - data - inputValue: ""



- FormComponent 내 watch 속성 추가
- watch:어떤 View Model을 감시하고 있다가,  그 값이 변경되면,  행동을 수행하는 함수
- watch 함수에 props 내 value를 호출 함. 
  - 이전 값과 새로운 값을 인자로 받아와, inputValue 에 새로운 값을 저장시킴.

```javascript
export default {
    template: '#search-form',
    props: ['value'],
    data() {
        return {
            inputValue: this.value
        }
    },
    watch: {
        value(newVal, oldVal) {
            this.inputValue = newVal
        }
    },
```





