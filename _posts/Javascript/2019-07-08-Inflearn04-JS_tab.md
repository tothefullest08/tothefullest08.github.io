---
layout: post
title: 실습 UI 개발을 통해 배워보는 JS & Vue JS (4) - 탭 구현 (JS)
category: Javascript
tags: [Javascript, Vue]
comments: true
---



> 본 강의는 Inflearn의 김정환 개발자 님의 강의([실습 UI 개발로 배워보는 순수 javascript 와 VueJS 개발]([https://www.inflearn.com/course/%EC%88%9C%EC%88%98js-vuejs-%EA%B0%9C%EB%B0%9C-%EA%B0%95%EC%A2%8C/dashboard](https://www.inflearn.com/course/순수js-vuejs-개발-강좌/dashboard)))를 듣고 배운 내용을 정리한 포스팅 입니다. 



## 탭 구현 (1)

> 추천 검색어, 최근 검색어 탭이 검색폼 아래 위치한다.

#### 1) index.html

- 검색 폼과 검색 결과 사이에 탭 구현
- id와 class 값에 `tabs`  입력. tabs 클래스에 대한 스타일은 style.css에 정의되어있음.

```html
<div class="container">
    <ul id="tabs" class="tabs">
        <li>추천 검색어</li>
        <li>최근 검색어</li>
    </ul>
    <div id="search-result"></div>
</div>
```

```css
  ul.tabs {
    display: flex;
  }
  .tabs li {
    display: inline-block;
    width: 50%;
    padding: 15px;
    text-align: center;
    box-sizing: border-box;
    border-bottom: 1px solid #ccc; 
    background-color: #eee;
    color: #999;
  }
```



## 탭 구현 (2)

> 기본으로 추천 검색어 탭을 선택한다.

style.css에는 tabs 클래스 내 li 태그 내 active 클래스 속성에 대한 스타일이 정의 되어있음.  따라서, active 클래스를 추천 검색어 탭에 적용시키면 됨.

#### 1) TabView.js

- 다른 View들과 마찬가지로, View를 import 하여 객체를 복사. 디버깅을 위해 Tag 정의
- `TabView`의 `setup`이라는 함수를 만들어 엘레멘트를 주입받아 init함수에 전달함.
- 탭에 active 클래스를 추가 하기 위해 `setActiveTab` 이라는 함수 정의. `tabName`이라는 인자를 받음
  - `li` 태그를 찾아서 array로 만든 후, forEach 반복문을 돌림.
  - 이때, 처음 맞는 li만 반환하는 것이 아니라 전체 데이터를 반환하기 위해, `querySelectorAll()` 메서드를 사용해야함.
  - `li`의 `className`에다가 `active` 문자열을 추가해주면 됨.
  - 이때 `innerHTML` 과 `TabName`이 같은 경우에만 active 클래스가 추가되도록 삼항 연산자 사용

```javascript
import View from './View.js'

const Tag = '[TabView]'

const TabView = Object.create(View)

TabView.setup = function(el) {
    this.init(el)
}

//active tab을 셋팅하는 함수
TabView.setActiveTab = function(tabName){
    Array.from(this.el.querySelectorAll('li')).forEach(li => {
        li.className = li.innerHTML == tabName ? 'active' : ''
    })
}

// MainController.js 에서 사용할 수 있게 export 시킴.
export default TabView
```



#### 2) MainController.js

- MainController.js 에서 TabView.js를 호출
- 다름 view와 마찬가지로, init 함수에서 TabView를 setup 함.
- 추천검색어를 선택하도록 만들어야됨. 탭은 추천 검색어 또는 최근 검색어를 선택할 수 있음. 그러므로 controller에서 어떤 탭을 선택했는지 내부적으로 가지고 있으면 좋음.
- controller에 `selectedTab`이라는 변수를 정의하고 default로 추천 검색어를 입력함.
- 이후, TabView의 `setActiveTab` 함수를 호출함. 이때 controller가 가지고있는 `selectedTab`을 인자로 넘김

```javascript
import FormView from '../views/FormView.js'
import ResultView from '../views/ResultView.js'
import TabView from '../views/TabView.js' // TabView 추가

import SearchModel from '../models/SearchModel.js'

const tag = '[MainController]'

export default {
    init() {
        FormView.setup(document.querySelector('form'))
            .on('@submit', e => this.onSubmit(e.detail.input))
            .on('@reset', e=> this.onResetForm())
        
        TabView.setup(document.querySelector('#tabs')) // TabView에 대한 setup 설정

        ResultView.setup(document.querySelector('#search-result'))  
        
        this.selectedTab = '추천 검색어'
        TabView.setActiveTab(this.selectedTab)
    },
```



#### 3) MainController.js

- controller에는 이미 3개의 view를 갖고 있음. 이 view들을 한번에 그려줄 함수를 추가로 정의해보자.
- `renderView`라는 함수 정의. 한번만 `renderView`라는 함수를 호출하면 controller가 갖고있는 view들을 다 그릴 수 있도록 `renderView`쪽으로 역할을 위임시킴.

```javascript
export default {
    init() {
 
		// 중간의 코드는 중략        
        this.selctedTab = '추천 검색어'
        this.renderView()
    },
    
    // 중략
    
    renderView() {
        console.log(tag, 'renderView()')
        TabView.setActiveTab(this.selctedTab)
        ResultView.hide()
    },

```





## 탭 구현 (3)

> 각 탭을 클릭하면 탭 아래 내용이 변경된다.

#### 1) TabView.js

- TabView.setup 내에서 `bindClick` 함수 호출 (클릭 이벤트에 대하여 bind 시키는 기능) & `bindClick` 함수 정의
- 탭들은 li 엘레멘트로 되어있으므로 li 전체(`querySelectorAll('li')`)를 arrary로 만든 후 반복문을 돌림.
- `li`에 대해 click event를 listener 하도록 코드 구현  & eventListener는 `onClick()` 으로 넘겨줌. 이때 `li.innerHTML`  (즉 , `TabName`을 인자로 넘겨줌)
- `onClick` 함수에는 `TabName`이 들어옴. `setActiveTab`의 인자로 `TabName`을 넘김.
- 또한 Tab이 change되었다는것을 Main Controller에게 알려줘야함. TabView는 Tab만 관리하며, 그 아래의 내용은 TabView에서는 신경쓰지 않는 내용임. 따라서 Tab이 변경된 사실만 controller에게 전달하면 됨.
- controller에서 setup 함수를 실행한다음 체이닝을 이용하여 on 이라는 함수를 실행할 예정이므로 TabView.setup에서 this를 return 시켜 줄 것.

```javascript
TabView.setup = function(el) {
    this.init(el)
    this.bindClick()
    return this
}


TabView.bindClick = function() {
    Array.from(this.el.querySelectorAll('li')).forEach(li => {
        li.addEventListener('click', e => this.onClick(li.innerHTML))
    })
}

TabView.onClick = function (tabName) {
    this.setActiveTab(tabName)
    this.emit('@change', {tabName})
}

```



#### 2) MainController.js

- `@change` 이벤트에 대해서 수신하고 있어야함.  이벤트가 들어왔을 때, controller에 onChangeTab이라는 함수를 실행하게 함.(이벤트의 tabName을 전달함 `e.detail.tabName` )
- 동작유무를 확인하기 위해 debugger 만 찍어봄. 최종적으로 탭을 누르면 breakpoint에 도달함을 알 수 있음.

```javascript
export default {

    init() {

        TabView.setup(document.querySelector('#tabs'))
            .on('@change', e => this.onChangeTab(e.detail.tabName))

    },
    
    onChangeTab(tabName) {
        debugger
    }
```

