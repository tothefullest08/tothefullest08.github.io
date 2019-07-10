---
layout: post
title: 실습 UI 개발을 통해 배워보는 JS & Vue JS (10) - 추천 검색어, 최근 검색어 구현 (Vue.js)
category: Javascript
tags: [Javascript, Vue]
comments: true
---



> 본 강의는 Inflearn의 김정환 개발자 님의 강의([실습 UI 개발로 배워보는 순수 javascript 와 VueJS 개발]([https://www.inflearn.com/course/%EC%88%9C%EC%88%98js-vuejs-%EA%B0%9C%EB%B0%9C-%EA%B0%95%EC%A2%8C/dashboard](https://www.inflearn.com/course/순수js-vuejs-개발-강좌/dashboard)))를 듣고 배운 내용을 정리한 포스팅 입니다. 



## 1. 추천 검색어 구현 

> 번호, 추천 검색어 목록이 탭 아래 위치한다.

#### 1) 기본 코드 작성

- 추천검색어는 KeywordModel을 통해서 데이터를 갖고와 Vue 인스턴스 data 내 keywords 라는 새로운 배열에 저장시킬 수 있음. 이때, 추천 검색어가 있는 경우와 없는 경우를 고려 할 수 있음.
- 추천 검색어가 있는 경우는 모델을 통해 데이터를 갖고와 출력.
- 그렇지 않은 경우에는 "추천 검색어가 없습니다"를 보여줌.

```html
<div v-if="selectedTab === tabs[0]">
    <div v-if="keywords.length">
    </div>
    <div v-else>
        추천 검색어가 없습니다.
    </div>
</div>
```

```javascript
    data: {
        query: '',
        submitted: false,
        tabs: ['추천 검색어', '최근 검색어'],
        selectedTab: '',
        keywords: [],
        searchResult: [],
    },
```



#### 2) KeywordModel 을 통해 데이터 갖고오기

- KeywordModel을 갖고오는 코드 작성
- Vue 인스턴스가 실행될 때, 데이터를 갖고옴 => created 함수 내 코드 작성(`fetchKeyword` 함수 호출)
- `fetchKeyword` 함수 작성
- keywords에 저장된 데이터를 반복문을 돌면서 index.html에 출력. 이때 숫자를 함께 출력하기 위해 배열의 인덱스 활용. 
- v-for을 이용하여 loop를 돌때는 바로 배열에 있는 객체를 갖고 올 수 있는데 (item, index)와 같이 작성하여 인덱스도 갖고올 수 있음.

```javascript
    created() {
        this.selectedTab = this.tabs[0]
        this.fetchKeyword()
    },
    methods: {
        fetchKeyword() {
            KeywordModel.list().then(data => {
                this.keywords = data
            })
        },
    }        
```

```html
<div v-if="keywords.length">
    <ul class="list">
        <li v-for="item in keywords">
            {{ item.keyword }}
        </li>
    </ul>
</div>
```



#### 3) 추천 검색어 구현 (2), (3)

>  목록에서 클릭하면 선택된 검색어로 검색 결과 화면으로 이동,  검색폼에 선택된 추천 검색어 설정

- 검색어 목록에 클릭 이벤트를 바인딩함.
- `onClickKeyword` 함수 정의. 
  - 입력한 값(검색한 값)이 저장되는 곳이 query임. 그곳에 keyword를 바인딩 시킴
  - 이후,  `search` 함수 호출

```html
<ul class="list">
    <li v-for="(item, index) in keywords"
        v-on:click="onClickKeyword(item.keyword)">
        <span class="number">{{index +1 }}</span>
        {{ item.keyword}}
    </li>
</ul>
```

```javascript
onClickKeyword(keyword){
    this.query = keyword
    this.search()
},
```



## 2. 최근 검색어 구현

#### 최근 검색어 구현 (1)

> 최근 검색어 목록이 탭 아래 위치한다. 

위의 명세는 추천 검색어 구현때와 거의 유사한 맥락이므로, 동일한 방식으로 쉽게 코드를 구현 할 수 있음.

- HistoryModel에서 데이터를 받아와 저장시킬 history 라는 변수를 data 내 생성 
- Vue인스턴스가 호출될 때 함수를 호출하는 created()에 `fetchHistory()` 함수 호출
- `fetchHistory()`: `HistoryModel` 에서 데이터를 받아와 history 변수에 데이터를 저장 시킴.
- loop를 돌면서 최근 검색어 목록을 index.html에 뿌려줌.

```javascript
import HistoryModel from './models/HistoryModel.js'

    data: {
        // 중략
        history: [],

    },
	created() {
        // 중략
        this.fetchHistory()
    },
    methods: {
        // 중략
        fetchHistory() {
            HistoryModel.list().then(data => {
                this.history = data
            })
        }
    }
```

```html
<div v-else>
    <div v-if="history.length">
        <ul class="list">
            <li v-for="item in history">
                {{ item.keyword }}
            </li>
        </ul>
    </div>
    <div v-else>
        최근 검색어가 없습니다.
    </div>
</div>
```



#### 최근 검색어 구현 (2)

> 목록에서 검색어를 클릭하면 선택된 검색어로 검색 결과 화면으로 이동한다.

- v-on디렉티브를 이용하여 click 이벤트를 받을 수 있음.
- keyword를 검색한 값이 저장된 query에 저장 시킨 후 , 검색 기능이 구현된 함수 search()로 뿌려주는 함수, `onClickKeyword`을 그대로 활용

```html
<div v-if="history.length">
    <ul class="list">
        <li v-for="item in history"
            v-on:click="onClickKeyword(item.keyword)">
            {{ item.keyword }}
        </li>
    </ul>
</div>
```



#### 최근 검색어 구현 (3)

> 검색일자, 버튼 목록이 있다.

```html
<ul class="list">
    <li v-for="item in history"
        v-on:click="onClickKeyword(item.keyword)">
        {{ item.keyword }}
        <span class="date"> {{item.date}}</span>
        <button class="btn-remove"></button>
    </li>
</ul>
```



#### 최근 검색어 구현 (4)

> 목록에서 x 버튼을 클릭하면 선택된 검색어가 목록에서 삭제된다.

- 반복문 내에서 v-on 디렉티브를 이용하여 클릭 이벤트를 설정
- HistoryModel.js 내 remove 메서드를 이용하여 키워드 삭제 후, 리스트를 다시 호출

```html
<ul class="list">
    <li v-for="item in history" v-on:click="onClickKeyword(item.keyword)">
        {{ item.keyword }}
        <span class="date"> {{item.date}}</span>
        <button class="btn-remove" v-on:click.stop="onClickRemoveHistory(item.keyword)"></button>
    </li>
</ul>
```

```javascript
onClickRemoveHistory(keyword){
    HistoryModel.remove(keyword)
    this.fetchHistory()
},
```



위의 코드대로 검색어를 삭제할 경우, 잠깐 삭제는 되나, 최근 검색어 키워드로 다시 검색이 진행됨. 이는 데이터가 버블링되면서 기존(아래)의 이벤트가 전파되기 때문임. 

```html
<li v-for="item in history" v-on:click="onClickKeyword(item.keyword)">
```

- JS에서는 이벤트를 막기 위해 `stopPropagation()`을 사용함. 
- Vue.js에서는 간단하게 이벤트 뒤에 `.stop`을 사용하여 이벤트 버블링을 막을 수 있음.

```html
<button class="btn-remove" v-on:click.stop="onClickRemoveHistory(item.keyword)"></button>
```



#### 최근 검색어 구현 (5)

> 검색시마다 최근 검색어 목록에 추가된다.

- 검색을 할때마다 search() 함수가 매번 호출됨. 이 함수 내에서 검색어 목록에 추가하는 코드를 추가하면 됨.

```javascript
search() {
    SearchModel.list().then(data => {
        this.submitted = true
        this.searchResult = data
    })
    HistoryModel.add(this.query)
    this.fetchHistory()
},
```



