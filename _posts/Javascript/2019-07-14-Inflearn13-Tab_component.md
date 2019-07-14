---
layout: post
title: 실습 UI 개발을 통해 배워보는 JS & Vue JS (13) - Tab Component 구현
category: Javascript
tags: [Javascript, Vue]
comments: true
---



> 본 강의는 Inflearn의 김정환 개발자 님의 강의([실습 UI 개발로 배워보는 순수 javascript 와 VueJS 개발]([https://www.inflearn.com/course/%EC%88%9C%EC%88%98js-vuejs-%EA%B0%9C%EB%B0%9C-%EA%B0%95%EC%A2%8C/dashboard](https://www.inflearn.com/course/순수js-vuejs-개발-강좌/dashboard)))를 듣고 배운 내용을 정리한 포스팅 입니다. 



## Tab Component 구현

#### 1) 기본 코드 작성

- TabComponent.js 생성

```javascript
export default {
    template: '#tabs'
}
```

- index.html 코드 수정

```html
<div class="content">
    <div v-if="submitted">
        <search-result v-bind:data="searchResult" 
                       v-bind:query="query">
        </search-result>
    </div>
    <div v-else>
        <tabs></tabs>
        <!-- <ul class="tabs">
                <li v-for="tab in tabs" 
                	v-bind:class="{active: tab === selectedTab}"
                	v-on:click="onClickTab(tab)">
                	{{tab}}
                </li>
			</ul> -->

        
<!-- tabs -->
<template id="tabs">
    <ul class="tabs">
        <li v-for="tab in tabs" v-bind:class="{active: tab === selectedTab}" 
            v-on:click="onClickTab(tab)">
            {{tab}}
        </li>
    </ul>
</template>
```

- app.js 내 컴포넌트 import & 추가

```javascript
import TabComponent from './components/TabComponent.js'

  components: {
    'tabs' : TabComponent,
  },
```



#### 2) 탭 구현

- 실제  탭을 출력하기위한 탭 데이터를 v-bind를 사용하여 넘김
  - `v-bind:tabs="tabs"` 에서 `"tabs"` 는 app.js에 저장된 변수임.
- 선택된 탭에 스타일 속성을 적용하기 위해 바인딩 설정
  - 속성을 정의할 때는 하이픈 사용
- 탭이 클릭될 때 마다 추천 검색어 & 최근 검색어를 바꿔서 보여줘야하므로 클릭 이벤트 정의
  - `@change` 이벤트가 발생하면 onClickTab 함수 실행
- 기존의 코드와 어떻게 변경했는지를 비교해서 살펴보자

```html
<!-- 컴포넌트 사용 코드 -->
<tabs v-bind:tabs="tabs" v-bind:selected-tab="selectedTab"
      v-on:@change="onClickTab"></tabs>

<!-- 이전 코드 -->
<ul class="tabs">
    <li v-for="tab in tabs" v-bind:class="{active: tab === selectedTab}"
        v-on:click="onClickTab(tab)">
        {{tab}}
    </li>
</ul>
```

```javascript
export default {
    template: '#tabs',
    props: ['tabs', 'selectedTab'],
    methods: {
        onClickTab(tab) {
            this.$emit('@change', tab)
        }
    }
}
```

