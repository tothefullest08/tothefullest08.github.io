---
layout: post
title: 실습 UI 개발을 통해 배워보는 JS & Vue JS (9) - 탭 구현 (Vue.js)
category: Javascript
tags: [Javascript, Vue]
comments: true
---



> 본 강의는 Inflearn의 김정환 개발자 님의 강의([실습 UI 개발로 배워보는 순수 javascript 와 VueJS 개발]([https://www.inflearn.com/course/%EC%88%9C%EC%88%98js-vuejs-%EA%B0%9C%EB%B0%9C-%EA%B0%95%EC%A2%8C/dashboard](https://www.inflearn.com/course/순수js-vuejs-개발-강좌/dashboard)))를 듣고 배운 내용을 정리한 포스팅 입니다. 



## 탭 구현 (1)

>추천 검색어,  최근 검색어 탭이 검색폼 아래 위치한다.

- `<div v-if="submitted">`  태그를 통해 검색결과를 출력하였음.  따라서, submitted이 발생하지 않았을 때, (`v-else`)를 사용하여 탭을 구현
- app.js 내 tabs 라는 변수 추가로 생성
- index.html 내 반복문을 통해 tabs 내 문자열인 "추천 검색어" 와 "최근 검색어"를 보여줌.
- tabs 클래스에 대한 style은 style.css에 기본적으로 구현되어 있음.

```html
<div v-else>
    <ul class="tabs">
        <li v-for="tab in tabs">
            {{ tab }}
        </li>
    </ul>
</div>
```

```javascript
    data: {
        query: '',
        submitted: false,
        tabs: ['추천 검색어', '최근 검색어'], 
        searchResult: [],
    },
```



## 탭 구현 (2)

> 기본으로 추천 검색어 탭을 선택한다.

- li 엘레멘트의 클래스에 active를 추가 시키면됨.
- `v-bind` 디렉티브를 사용하는데, class명을 bind 시킬 수 있음. 이때 클래스명을 조건을 줄 수 있음.
  - 설정할 클래스 명을 입력, 클래스 명이 설정될 조건을 추가로 입력
  - tab이 selectedTab과 같은 경우 active 적용
- `selectedTab: '추천 검색어'` 처럼 하드하게 data 내 입력해도 되지만, tabs 배열의 0번 인덱스의 값을 갖고오는 방식을 사용 할 수 있음.
- 라이프 싸이클 중, `created()`라는 함수를 사용할 수 있음. 이는 Vue 인스턴스가 생성될 때 호출 되는 함수임.

```html
<ul class="tabs">
    <li v-for="tab in tabs" v-bind:class="{active : tab === selectedTab}">
        {{ tab }}
    </li>
</ul>
```

```javascript
    data: {
        query: '',
        submitted: false,
        tabs: ['추천 검색어', '최근 검색어'],
        selectedTab: '',
        searchResult: [],
    },
    created() {
        this.selectedTab = this.tabs[0]
    },
```



## 탭 구현 (3)

> 각 탭을 클릭하면 탭 아래 내용이 변경된다.



#### 1) 탭 이동 구현

- li 엘레멘트에 v-on 디렉티브를 이용하여 클릭 이벤트를 설정 시킴. 
- 반복문을 도는 tab을 함수의 인자로 넘김. 함수 내에서 selectedTab의 값을 받아온 인자값으로 변경

```html
<ul class="tabs">
    <li v-for="tab in tabs" v-bind:class="{active : tab === selectedTab}" 
        v-on:click="onClickTab(tab)">
        {{ tab }}
    </li>
</ul>
```

```javascript
        onClickTab(tab) {
            this.selectedTab = tab
        },
```



#### 2) 탭 클릭 시 내용 변경

- 탭이름이 "추천 검색어" 인 경우와 "최근 검색어" 인 경우, 2가지의 조건을 고려하여 분기문 작성

```html
<div v-else>
    <ul class="tabs">
        <li v-for="tab in tabs" v-bind:class="{active : tab === selectedTab}" 
            v-on:click="onClickTab(tab)">
            {{ tab }}
        </li>
    </ul>
    <div v-if="selectedTab === tabs[0]">
        추천 검색어 목록
    </div>
    <div v-else>
        최근 검색어 목록
    </div>
</div>
```



