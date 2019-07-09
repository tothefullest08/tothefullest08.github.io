---
layout: post
title: 실습 UI 개발을 통해 배워보는 JS & Vue JS (5) - 추천 검색어 구현 (JS)
category: Javascript
tags: [Javascript, Vue]
comments: true
---



> 본 강의는 Inflearn의 김정환 개발자 님의 강의([실습 UI 개발로 배워보는 순수 javascript 와 VueJS 개발]([https://www.inflearn.com/course/%EC%88%9C%EC%88%98js-vuejs-%EA%B0%9C%EB%B0%9C-%EA%B0%95%EC%A2%8C/dashboard](https://www.inflearn.com/course/순수js-vuejs-개발-강좌/dashboard)))를 듣고 배운 내용을 정리한 포스팅 입니다. 



## 추천 검색어 구현 (1)

> 번호, 추천 검색어 목록이 탭 아래 위치한다.

#### 1) index.html

- 추천 검색어 목록을 보여줄 코드 구현
- div 엘리먼트 내 `search-keyword`  id 값 적용
- 실제 데이터는 서버에서 갖고오고, 그 데이터를 기반으로  DOM을 만들어내기 때문에 1개의 div만 만듦.
- `search-keyword` 를 이용하여 `KeywordView.js` 생성

```html
<div class="container">
    <ul id="tabs" class="tabs">
        <li>추천 검색어</li>
        <li>최근 검색어</li>
    </ul>

    <div id="search-keyword"></div>
    <div id="search-result"></div>
</div>
```



#### 2) KeywordView.js

-  다른 View와 동일하게 View를 가져와 복사해서 사용 & 디버깅을 위한 태그 생성
- `setup` 함수 & 데이터를 받아서 뿌려줄 `render` 함수 생성
- 엘레멘트의 innerHTML에 데이터를 넣음. 2가지의 경우 고려
  - 데이터가 있는 경우(`data.length`): `getKeywordHtml` 함수를 통해서 html 문자열을 받아옴
  - 그렇지 않은 경우, "추천 검색어가 업습니다" 문자열 출력
  - `this.show()` 메서드를 이용하여 실제로 출력
- `getKeywordHtml`  함수 정의. 우선은 간단히 `debugger` 만 입력 
- MainController.js에서 사용 할 수 있도록 모듈을 export 해 줌

```javascript
import View from './View.js'

const tag = '[KeywordView]'

const KeywordView = Object.create(View)

KeywordView.setup = function(el) {
    this.init(el)
}

KeywordView.render = function (data = []) {
    this.el.innerHTML = data.length ? this.getKeywordsHtml(data) : '추천 검색어가 없습니다'
    this.show()
}

KeywordView.getKeywordsHtml = function (data) {
    debugger
}

export default KeywordView 
```



#### 3) MainController.js

- KeywordView.js 를 import 해오고 controller를 초기화하는 코드에 삽입
- `search-keyword` 엘레멘트 (id값) 로 setup 설정

```javascript
import KeywordView from '../views/KeywordView.js'

export default {
    // 컨트롤러 초기화 부분
    init() {
		// 중략
        KeywordView.setup(document.querySelector('#search-keyword'))
```



- search-keyword에 render하는 코드를 작성.  MainController.js 내 `renderView`가 그 역할을 담당하고있음.
- controller가 가지고 있는 데이터 중 selectedTab이라는 게  있음. (default값: "추천 검색어")
- selectedTab 값이 어떤 것이냐에 따라서 "추천 검색어" 또는 "최근 검색어"를 출력하는 로직을 짤 수 있음.
  - "추천 검색어" 인 경우 KeywordView.js에 `render` 함수를 호출함. (현재까지의 코드로는 "추천 검색어가 없습니다" 가 출력됨)

```javascript

	renderView() {
        console.log(tag, 'renderView()')
        TabView.setActiveTab(this.selctedTab)
        ResultView.hide()

        if (this.selctedTab=== '추천 검색어') {
            KeywordView.render()
        } else {

        }
    },

```



#### 4) MainController.js (2)

- models/KeywordModel.js 를 통해 추천 검색어를 가져 옴.  기 파일 내`list` 함수를 이용해 데이터를 수신
- controller에서 KeywordModel을 가져온 후 `renderView` 함수 내에서 KeywordModel의 `list`를 호출 함.
- 호출한 값은 `promise` 객체를 반환하므로 .then 함수를 통해 데이터를 얻어 올 수 있음.
-  받아온 데이터를 `render` 함수에 그대로 넘김
- 이후 서버를 실행시키면, `KeywordView.getKeywordsHtml` 함수가 실행되어 `debugger`로 지정한 곳에 멈춤. 이는 `keywordView.render` 함수를 통해 `data.length`가 있기 때문에  `getKeywordsHtml`이 호출된 것.
- 나머지 우리가 해야할 일은 `getKeywordHtml`을 구현하는 것

``` javascript
import KeywordModel from '../models/KeywordModel.js'

renderView() {
    console.log(tag, 'renderView()')
    TabView.setActiveTab(this.selctedTab)
    ResultView.hide()

    if (this.selctedTab=== '추천 검색어') {
        KeywordModel.list().then(data => {
            KeywordView.render(data)
        })
    } else {

    }
},
```



#### 5) KeywordView.js

- 받은 데이터를 reduce함수를 통해 HTML을 생성
- 초기 값으로 `<ul>` 입력. style.css를 통해 이미 정의된 css를 적용하기 위해 list 클래스 추가
- `reduce` 함수를 사용하는 매커니즘은 이전 포스팅과 동일
- 추가로, number를 넣기 위해 `reduce`의 3번째 인자로 `index`를 삽입

```javascript
KeywordView.getKeywordsHtml = function (data) {
    return data.reduce((html, item, index) => {
        html += `<li>
        <span class="number">${index + 1} </span>
        ${item.keyword}
        </li>` 
        return html        
    }, '<ul class="list">') + '</ul>'
```



#### 6) MainController.js

- MainController.js 내 renderView() 함수에서, 추천검색어를 선택한 경우 모델에서 데이터를 갖고와 KeyworView.render함수에 뿌려주는 코드는 view를 rendering하는 역할이기 때문에 모델에서 데이터를 갖고오는 부분은 별도로 따로 떼 낼 수 도 있음. 
- 코드를 떼네어, `fetchSearchKeyword` 라는 함수에게 위임

```javascript
    renderView() {
        console.log(tag, 'renderView()')
        TabView.setActiveTab(this.selctedTab)

        if (this.selctedTab=== '추천 검색어') {
            this.fetchSearchKeyword()
        } else {

        }

        ResultView.hide()

    },

    fetchSearchKeyword() {
        KeywordModel.list().then(data => {
            KeywordView.render(data)
        })
    },

```



## 추천 검색어 구현 (2)

> 목록에서 검색어를 클릭하면 선택된 검색어로 검색 결과 화면으로 이동 

추천 검색어에 있는 리스트를 클릭하면 클릭 이벤트를 수신하도록 하자

#### 1) KeywordView.js (1)

- bind이벤트(`bindClickEvent`)를 setup에서 호출 & 함수 구현
- li 엘레멘트를 찾은 후, 클릭 이벤트를 하나씩 바인딩 하는 코드 작성 
- 유사 배열이므로 Array.from을 통해 array로 만든 후, forEach를 통해 array의 요소에 하나씩 접근
- li 엘레멘트에 클릭 이벤트 바인딩한 후 onClickKeyword 함수로 이벤트를 전달 해줌.

```javascript
KeywordView.bindClickEvent = function() {
    Array.from(this.el.querySelectorAll('li')).forEach(li => {
        li.addEventListener('click', e => this.onClickKeyword(e))
    })
}
```



#### 2) Keyword.View.js (2)

- onClickKeyword 함수 구현
- 데이터를 클릭했을 때 이벤트는 발생. 이 때 어떤 추천 검색어가 클릭되었는지를 알아야함. 그렇게 하기 위해서는 이벤트(`e`) 에 데이터를 심어서 보내줘야함.
- `getKeywordHtml` 함수 내 `reduce` 메서드를 통해 html에 쌓아올리는 코드에서 `data` 변수를 추가
- `data-keyword`에 실제 출력한 키워드(`item.keyword`)를 바인딩 함. 이 데이터를 갖고와 활용
- `keyword`를 추출  (`keyword`는 `e.currentTarget.dataset`에 저장되어있음)
- 클릭했을 때 검색 결과 페이지로 넘어가는 역할은 KeywordView의 역할은 아님. 이는 mainController가 다룰 예정이므로, MainController에게 어떤 값이 클릭되었다는 것을 통지만 함(위임) => emit 함수 사용
- 해당 이벤트를 controller가 수신할 수 있도록 setup 함수에서 this를 return 시킴

```javascript
KeywordView.setup = function(el) {
    this.init(el)
    this.bindClickEvent()

    return this
}

KeywordView.getKeywordsHtml = function (data) {
    return data.reduce((html, item, index) => {
        html += `<li data-keyword=${item.keyword}>
        <span class="number">${index + 1} </span>
        ${item.keyword}
        </li>` 
        return html        
    }, '<ul class="list">') + '</ul>'
}

KeywordView.onClickKeyword = function(e) {
    const {keyword} = e.currentTarget.dataset
    this.emit('@click', {keyword}) 
}
```



#### 3) MainController.js

- 해당 이벤트를 MainController에서 수신 할 수 있도록 KeywordView.setup 함수를 체이닝으로 수신
-  `onClickKeyword` 함수를 호출하며, 이벤트의 키워드 값을 전달함
- keywordView에서 전달한 값이 제대로 들어오는지 확인하기 위해 breakpoint 설정

```javascript
export default {
    init() {
        // 중략
        KeywordView.setup(document.querySelector('#search-keyword'))
            .on('@click', e => this.onClickKeyword(e.detail.keyword))
    },
	
    // 중략

    onClickKeyword(keyword) {
        debugger
    }
}

```



#### 4) keywordView.js

- setup에서 bind이벤트를 발생 시키기 않고, DOM이 만들어 진 후에 이벤트를 발생시켜야함. 

- render 함수에서 innerHTML을 추가한 뒤에 bind 이벤트를 발생시키도록 수정

```javascript
KeywordView.setup = function(el) {
    this.init(el)
    return this
}

KeywordView.render = function (data = []) {
    this.el.innerHTML = data.length ? this.getKeywordsHtml(data) : this.messages.NO_KEYWORDS
    this.bindClickEvent()
    this.show()
}
```

 

#### 5) MainController.js

- OnClickKeyword 함수 구현. 이 함수는 키워드가 클릭을 했을 때 실행 됨.
- 실제 검색을 해야함. 검색하는 기능은 미리 구현해놓은 search 함수 활용

```javascript
onClickKeyword(keyword) {
    this.search(keyword)
},
```



#### 6) MainController.js

- 여기까지 코드로 서버를 실행시키면 탭과 검색결과가 동시에 뜸
- onClickKeyword 함수가 실행되면서 들어오는 keyword를 search 함수에 넘겨줌
- search 함수는 받은 값을 가지고 searchModel에서 조회를 한 후, 거기서 받은 데이터를 onSearchResult로 넘김.
- onSearchResult에서는 resultView를 그리는 역할을 함. 여기서 그리기 전에, 기존에 있던 TabView와 KeywordView를 숨겨줄 필요가 있음. =>  `TabView.hide()`  & `        KeywordView.hide()` 삽입!

```javascript
    search(query) {
        console.log(tag, 'search()', query)
        // search API
        SearchModel.list(query).then(data => {
            this.onSearchResult(data)
        })
    },
    

    onSearchResult(data) {
        TabView.hide()
        KeywordView.hide()
        ResultView.render(data)
    },


    onClickKeyword(keyword) {
        this.search(keyword)
    },
}

```



## 추천 검색어 구현 (3)	

>검색폼에 선택된 추천 검색어 설정 

#### 1) MainController.js

- onClickKeyword 함수는 받은 키워드를 search 함수로 넘겨줌.
- `search` 함수에 FormView를 셋팅하는 로직을 추가 => `setValue` 함수 호출

```javascript
    search(query) {
        FormView.setValue(query)
        SearchModel.list(query).then(data => {
            this.onSearchResult(data)
        })
    },
        
    onClickKeyword(keyword) {
        this.search(keyword)
    },
```



#### 2) FormView.js (1)

- FormView.js 내 `setValue` 함수 구현
- `inputEl`의` value값에 받아온 value를 저장시킴

```javascript
FormView.setValue = function(value = '') {
    this.inputEl.value = value
}
```



#### 3) FormView.js (2)

- 추가로 검색폼에 선택된 추천 검색어를 뜨게했을 때, 삭제가능한 x버튼도 뜨도록 코드를 작성할 수 있음.
- 이부분은 FormView.js 내 `showResetBtn` 함수를 통해 구현되어있음.
- showResetBtn 값을 true로 주거나,  inputEl.value.length을 boolean 값으로 해석하도록 넘겨 줘도 됨.

```javascript

FormView.setValue = function(value = '') {
    this.inputEl.value = value
    // this.showResetBtn(true)
    this.showResetBtn(this.inputEl.value.length)
```



#### 4) MainController.js

- 검색폼에 검색어를 삭제했을 경우, TabView가 사라짐. 이 View를 복구하는 코드를 작성할 수 있음. 
- 검색 폼에서 x버튼을 누르면 MainController.js 를 통해 `@reset` 이벤트가 발생함. 그러면 controller에서는 `onResetForm()` 함수가 실행됨 `onResetForm` 은 단순히 ResultView를 감쳐주기만 하고 있음.
- 이부분을 수정하여, controller가 view를 그리는 renderView()를 호출하도록 함.

```javascript
export default {
    init() {

        FormView.setup(document.querySelector('form'))
            .on('@submit', e => this.onSubmit(e.detail.input))
            .on('@reset', e=> this.onResetForm())
    },
    
    onResetForm(input) {
        console.log(tag, 'onResetForm()')
        // ResultView.hide()
        this.renderView()
    },   
```



#### ※ Error 발생 case

이번 챕터를 따라가다가 두 차례 에러가 발생한 이력이 있어 메모를 남김.  두 차례 모두 this.show를 입력하지 않아 발생한 에러였음. 해당 강의는 git을 통해 각 강의별로 branch를 불러와 진행하는데,  자세히 코드를 살펴보면 이전 강의에서 언급하지 않았던 코드가 일부 수정되거나 추가된 경우를 종종 보았다. 

- 검색어를 입력하고 엔터를 쳤을 때 결과가 나오지 않은 경우 
  - ResultView.js 내 `render` 함수에서 `this.show()`삽입

```javascript
ResultView.render = function(data = []) {
    console.log(tag, 'render()', data)
    this.el.innerHTML = data.length ? this.getSearchResultsHtml(data) : '검색 결과가 없습니다'
    this.show()
}
```



- 검색어를 삭제한 후, TabView가 뜨지 않고 추천검색어 목록만 뜨는 경우
  - TabVivew.js 내 `setActiveTab` 함수에서 `this.show()` 삽입

```javascript
TabView.setActiveTab = function(tabName){
    Array.from(this.el.querySelectorAll('li')).forEach(li => {
        li.className = li.innerHTML == tabName ? 'active' : ''
    })
    this.show()
}
```
