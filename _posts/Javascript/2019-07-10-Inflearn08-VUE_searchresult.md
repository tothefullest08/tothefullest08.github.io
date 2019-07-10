---
layout: post
title: 실습 UI 개발을 통해 배워보는 JS & Vue JS (8) - 검색 결과 구현 (Vue.js)
category: Javascript
tags: [Javascript, Vue]
comments: true
---



> 본 강의는 Inflearn의 김정환 개발자 님의 강의([실습 UI 개발로 배워보는 순수 javascript 와 VueJS 개발]([https://www.inflearn.com/course/%EC%88%9C%EC%88%98js-vuejs-%EA%B0%9C%EB%B0%9C-%EA%B0%95%EC%A2%8C/dashboard](https://www.inflearn.com/course/순수js-vuejs-개발-강좌/dashboard)))를 듣고 배운 내용을 정리한 포스팅 입니다. 



## 검색 결과 구현 (1), (2)

> 검색 결과가 검색폼 아래 위치한다. & 검색 결과가 보인다

#### 1) 추가 코드 

- 이전 강의와 비교해서 강의에서 추가된 코드는 아래와 같음

```javascript
// 이전 코드
onReset() {
    this.query = ''
},
    
// 변경된 코드
onReset() {
    this.resetForm()
},
resetForm() {
    this.query = ''
    // todo remove result
}
```



#### 2) app.js

- 검색 결과의 데이터를 저장할 변수를 정의 (배열의 형태로 초기화) - `searchResult`
- `searchResult`를 출력해주는 부분을 index.html에서 작성

```javascript
data: {
    query: '',
    searchResult: [],
},
```



#### 3) index.html

- `v-if`를 사용하여 데이터가 있는 경우와 없는 경우를 구분하여 작성. 이때 `searchResult.length` 활용
- 데이터가 없는 경우 -  `{{ query }} 검색어로 찾을 수 없습니다.` 
- 데이터가 있는경우  `v-for`을 이용하여 배열을 반복돌며 출력
  - img 태그의 src값에 바인딩된 값을 사용할 때는 `v-bind` 사용 (attribute의 값을 바인딩하는 역할)

```html
<div v-if="searchResult.length">
    <ul>
        <li v-for="item in searchResult">
            <img v-bind:src="item.image"> {{item.name}}
        </li>
    </ul>
</div>
<div v-else>
    {{ query }} 검색어로 찾을 수 없습니다.
</div>
```



#### 4) app.js

- 검색을 하는 코드 구현(`onSubmit`) -> `search` 라는 함수 호출
- search 함수는 SearchModel.js에 있는 데이터를 갖고와, searchResult 변수에 저장시키는 로직
- SearchModel의 데이터를 사용하기 위해 import

```javascript
import SearchModel from './models/SearchModel.js'

    methods: {
        onSubmit(e) {
            this.search()
        },
        search() {
            SearchModel.list().then(data => {
                this.searchResult = data
            })
        },
```



#### 5) app.js 

아무것도 입력하지 않았는데 "검색어로 찾을 수 없습니다" 가 출력되고 있음. 검색을 했는지 안했는지를 Form이 submit 되었는지 안되었는지를 관리하는 변수가 필요함. 

- `submitted` 라는 변수 생성
- 검색이 일어났을 때, `submitted =true`로 변경해줌
- `submitted` 값에 따라서 검색 결과창의 결과를 다르게 보여주면 됨.

```javascript
data: {
    query: '',
    submitted: false, 
    searchResult: [],
},
methods: {
    search() {
        SearchModel.list().then(data => {
            this.submitted = true
            this.searchResult = data
        })
    },
```

```html
<div v-if="submitted">
    <div v-if="searchResult.length">
        <ul>
            <li v-for="item in searchResult">
                <img v-bind:src="item.image"> {{item.name}}
            </li>
        </ul>
    </div>
    <div v-else>
        {{ query }} 검색어로 찾을 수 없습니다.
    </div>
</div>
```



## 검색 결과 (3)

> x 버튼을 클릭하면 검색폼이 초기화되고, 검색 결과가 사라진다

- app.js 내 resetForm() 함수에 코드를 삽입하면 됨
  - submitted을 false로 변경
  - searchResult를 빈배열로 바꿔줌

```javascript
resetForm() {
    this.query = ''
    this.submitted = false
    this.searchResult = []
}
```

