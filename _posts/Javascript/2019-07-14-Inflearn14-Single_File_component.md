---
layout: post
title: 실습 UI 개발을 통해 배워보는 JS & Vue JS (14) - 단일 파일 컴포넌트 
category: Javascript
tags: [Javascript, Vue]
comments: true
---



> 본 강의는 Inflearn의 김정환 개발자 님의 강의([실습 UI 개발로 배워보는 순수 javascript 와 VueJS 개발]([https://www.inflearn.com/course/%EC%88%9C%EC%88%98js-vuejs-%EA%B0%9C%EB%B0%9C-%EA%B0%95%EC%A2%8C/dashboard](https://www.inflearn.com/course/순수js-vuejs-개발-강좌/dashboard)))를 듣고 배운 내용을 정리한 포스팅 입니다. 



## 단일 파일 컴포넌트(Single File Component) 구현

지금까지 컴포넌트를 통해 작성한 코드를 살펴보면 index.html + component.js로 구성되어 있는 것을 알 수 있음. Vue에서는 컴포넌트를 .vue라는 확장자를 이용하여 하나의 단일컴포넌트로 구현할 수 있도록 기능을 지원하고 있음. 

예를 들어, TabComponent.vue라는 파일만 보면 index.html과 TabComponent.js 파일 둘다를 볼 필요 없게됨. 가독성이 훨씬 더 증가하며, 유지보수에도 편리하다는 장점이 생김.



#### 1) SFC  설치

```bash
npm install vue-cli global  #Vue Cli 전역으로 설치
vue list #template 확인하기
vue init webpack-simple #webpack-simple 설치
npm install #개발 서버 가동을 위해 npm install
npm run dev #개발 서버 가동
```



#### 2) SFC 기본 구성 & 시작하기

- src 폴더의 main.js가 바로 진입점. 여기서부터 시작함

```javascript
import Vue from 'vue'
import App from './App.vue'

new Vue({
  el: '#app',
  render: h => h(App)
})
```



- 메인 어플리케이션은 App.vue에서 만듦. template & script & style 이라는 3부분으로 구성됨.

```vue
<template>
</template>

<script>
export default {
  name: 'app',
  data () {
    return {
    }
  }
}
</script>

<style>
</style>
```



- base로 사용하였던 style.css를 root 디렉토리에 붙여넣은 후, index.html과 연결시킴.
- viewport 설정을 head 태그 내 진행
- src 폴더 내 models 폴더 복사 & 저장

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
  	<meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>base_sfc</title>
    <link rel="stylesheet" href="style.css">
  </head>
  <body>
    <div id="app"></div>
    <script src="/dist/build.js"></script>
  </body>
</html>
```



- header 출력하기;  App.vue에 코드 삽입

```vue
<template>
  <div>
    <header>
      <h2 class="container">검색</h2>
    </header>
  </div>
</template>
```



#### 3) FormComponent.vue

- src 폴더 내 components 폴더 생성
- FormComponent.vue 생성 후 기본 구조에 따라 코드 작성 후, Component 에서 사용했던 template & script 코드를 불러옴
- 어느 template에 사용할 것이냐에 대해 명시하는 부분이었던  template 속성은 삭제 (바로 위에 있는 template에 사용하므로 쓸 필요가 없음)

```vue
<template>
  <form v-on:submit.prevent="onSubmit">
    <input type="text" v-model="inputValue" v-on:keyup="onKeyup" 
           placeholder="검색어를 입력하세요" autofocus>
    <button v-show="inputValue.length" v-on:click="onReset" 
            type="reset" class="btn-reset"></button>
  </form>
</template>

<script>
export default {
    // template: '#search-form',
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
    methods: {
        onSubmit() {
            this.$emit('@submit', this.inputValue.trim())
        },
        onKeyup() {
            if (!this.inputValue.length) this.onReset()
        },
        onReset() {
            this.inputValue = ''
            this.$emit('@reset')
        },
    }
}
</script>
```



#### 4) App.vue 

- 컴포넌트 import & components 속성에 추가
- FormComponent를 출력하는 부분을 header 태그 바로 아래에 작성(이전 프로젝트에서 작성한 코드 그대로 사용)
- components 프로젝트 내 app.js 에서 사용했던 data, created와 methods를 그대로 사용
- Model import

```vue
<template>
  <div>
    <header>
      <h2 class="container">검색</h2>
    </header>
    <div class="container">
      <search-form v-bind:value="query" v-on:@submit="onSubmit" v-on:@reset="onReset"></search-form>
    </div>
  </div>
</template>

<script>
import SearchModel from './models/SearchModel.js'
import KeywordModel from './models/KeywordModel.js'
import HistoryModel from './models/HistoryModel.js'

import FormComponent from './components/FormComponent.vue'

export default {
  name: 'app',
  data () {
    return {
      query: '',
      submitted: false,
      tabs: ['추천 검색어', '최근 검색어'],
      selectedTab: '',
      keywords: [],
      history: [],
      searchResult: []
    }
  },
  created() {
    this.selectedTab = this.tabs[0]
    this.fetchKeyword()
    this.fetchHistory()
  },
  components: {
    'search-form': FormComponent
  },
  methods: {
    onSubmit(query) {
      this.query = query
      this.search()
    },
    onReset(e) {
      this.resetForm()
    },
    onClickTab(tab) {
      this.selectedTab = tab
    },
    onClickKeyword(keyword) {
      this.query = keyword;
      this.search()
    },
    onClickRemoveHistory(keyword) {
      HistoryModel.remove(keyword)
      this.fetchHistory()
    },
    fetchKeyword() {
      KeywordModel.list().then(data => {
        this.keywords = data
      })
    },
    fetchHistory() {
      HistoryModel.list().then(data => {
        this.history = data
      })
    },
    search() {
      SearchModel.list().then(data => {
        this.submitted = true
        this.searchResult = data
      })
      HistoryModel.add(this.query)
      this.fetchHistory()
    },
    resetForm() {
      this.query = ''
      this.submitted = false
      this.searchResult = []
    }

  }
}
</script>
```



- 같은 방식으로 나머지 component도 단일 파일 컴포넌트로 변경해서 구현 할 수 있음.

#### 5) ResultComponent.vue

```vue
<template>
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


<script>
export default {
    props: ['data', 'query'],
}
</script>
```



#### 6) ListComponent.vue

```vue
<template>
  <div v-if="data.length">
    <ul class="list">
      <li v-for="(item, index) in data" v-on:click="onClickList(item.keyword)">
        <span v-if="keywordType" class="number">{{index + 1}}</span>
        {{item.keyword}}
        <span v-if="historyType" class="date">{{item.date}}</span>
        <button v-if="historyType" class="btn-remove" v-on:click.stop="onRemoveList(item.keyword)"></button>
      </li>
    </ul>
  </div>
  <div v-else>
    <span v-if="keywordType">추천 검색어가 없습니다</span>
    <span v-if="historyType">최근 검색어가 없습니다</span>
  </div>
</template>


<script>
export default {
    props: ['data', 'type'],
    computed: {
        keywordType() {
            return this.type === 'keywords'
        },
        historyType() {
            return this.type === 'history'
        },
    },
    methods: {
        onClickList(keyword) {
            this.$emit('@click', keyword)

        },
        onRemoveList(keyword) {
            this.$emit('@remove', keyword)

        }
    }
}
</script>
```



#### 7) TabComponent.vue

```vue
<template>
  <ul class="tabs">
    <li v-for="tab in tabs" v-bind:class="{active: tab === selectedTab}" v-on:click="onClickTab(tab)">
      {{tab}}
    </li>
  </ul>
</template>

<script>
export default {
    props: ['tabs', 'selectedTab'],
    methods: {
        onClickTab(tab) {
            this.$emit('@change', tab)
        }

    }
}
</script>
```



#### 8) App.vue

```vue
<template>
  <div>
    <header>
      <h2 class="container">검색</h2>
    </header>
    <div class="container">
      <search-form v-bind:value="query" v-on:@submit="onSubmit" v-on:@reset="onReset"></search-form>
    </div>
    <div class="content">
      <div v-if="submitted">
        <search-result v-bind:data="searchResult" v-bind:query="query"></search-result>
      </div>
      <div v-else>
        <tabs v-bind:tabs="tabs" v-bind:selected-tab="selectedTab" v-on:@change="onClickTab"></tabs>
        <div v-if="selectedTab === tabs[0]">
          <list v-bind:data="keywords" type="keywords" v-on:@click="onClickKeyword"></list>
        </div>
        <div v-else>
          <list v-bind:data="history" type="history" v-on:@click="onClickKeyword" v-on:@remove="onClickRemoveHistory">
          </list>
        </div>
      </div>      
    </div>
  </div>
</template>

<script>
import SearchModel from './models/SearchModel.js'
import KeywordModel from './models/KeywordModel.js'
import HistoryModel from './models/HistoryModel.js'

import FormComponent from './components/FormComponent.vue'
import ResultComponent from './components/ResultComponent.vue'
import ListComponent from './components/ListComponent.vue'
import TabComponent from './components/TabComponent.vue'

export default {
  name: 'app',
  data () {
    return {
      query: '',
      submitted: false,
      tabs: ['추천 검색어', '최근 검색어'],
      selectedTab: '',
      keywords: [],
      history: [],
      searchResult: []
    }
  },
  components: {
    'search-form': FormComponent,
    'search-result':ResultComponent,
    'list':ListComponent,
    'tabs':TabComponent,
  },
  created() {
    this.selectedTab = this.tabs[0]
    this.fetchKeyword()
    this.fetchHistory()
  },
  methods: {
    onSubmit(query) {
      this.query = query
      this.search()
    },
    onReset(e) {
      this.resetForm()
    },
    onClickTab(tab) {
      this.selectedTab = tab
    },
    onClickKeyword(keyword) {
      this.query = keyword;
      this.search()
    },
    onClickRemoveHistory(keyword) {
      HistoryModel.remove(keyword)
      this.fetchHistory()
    },
    fetchKeyword() {
      KeywordModel.list().then(data => {
        this.keywords = data
      })
    },
    fetchHistory() {
      HistoryModel.list().then(data => {
        this.history = data
      })
    },
    search() {
      SearchModel.list().then(data => {
        this.submitted = true
        this.searchResult = data
      })
      HistoryModel.add(this.query)
      this.fetchHistory()
    },
    resetForm() {
      this.query = ''
      this.submitted = false
      this.searchResult = []
    }

  }
}
</script>
```