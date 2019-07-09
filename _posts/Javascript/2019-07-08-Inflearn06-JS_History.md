---
layout: post
title: 실습 UI 개발을 통해 배워보는 JS & Vue JS (6) - 최근 검색어 구현 (JS)
category: Javascript
tags: [Javascript, Vue]
comments: true
---



> 본 강의는 Inflearn의 김정환 개발자 님의 강의([실습 UI 개발로 배워보는 순수 javascript 와 VueJS 개발]([https://www.inflearn.com/course/%EC%88%9C%EC%88%98js-vuejs-%EA%B0%9C%EB%B0%9C-%EA%B0%95%EC%A2%8C/dashboard](https://www.inflearn.com/course/순수js-vuejs-개발-강좌/dashboard)))를 듣고 배운 내용을 정리한 포스팅 입니다. 



## 최근 검색어 구현 (1), (2)

> 최근 검색어, 목록이 탭 아래 위치한다 
> [ ] 목록에서 검색어를 클릭하면 선택된 검색어로 검색 결과 화면으로 이동

#### 1) index.html

- div 태그 추가 (`id="search-history"`)

``` html
<div id="search-keyword"></div>
<div id="search-history"></div>
<div id="search-result"></div>
```



#### 2) HistroyView.js  & MainController.js

- KeywordView를 복사하여 구현
- HistoryView와 MainController를 연결
  - KeywordView를 연결했던 것과 동일한 방식으로 연결
  - `onClickHistory` 함수 구현. `search`함수에 keyword를 넘김
  - `selectedTab`의 default값을  "최근 검색어" 로 변경

```javascript
// HistoryView
import KeywordView from './KeywordView.js'

const tag = '[HistoryView]'

const HistoryView = Object.create(KeywordView)

export default HistoryView

// MainController
import HistoryView from '../views/HistoryView.js'


export default {
    init() {
        KeywordView.setup(document.querySelector('#search-keyword'))
            .on('@click', e => this.onClickKeyword(e.detail.keyword))
        
        HistoryView.setup(document.querySelector('#search-history'))
            .on('@click', e => this.onClickHistory(e.detail.keyword))
        
        this.selctedTab = '최근 검색어'
    },
    
    
    onClickKeyword(keyword) {
        this.search(keyword)
    },

    onClickHistory(keyword) {
        this.search(keyword)
    },
    
```



#### 3) MainController.js (2)

- 위의 코드대로 서버를 실행시키면 아래 함수에 따라 debugger로 입력된 breakpoint에서 멈추게된다. 
- 분기문  if 조건에서 작성한 코드처럼, 유사한 코드를 '최근 검색어' 에 대해 작성할 수 있음. (`this.fetchSearchHistory()`)
- HistoryModel 내 list 함수를 호출함(promise 객체를 반환함). 결과를 data에 저장하여 `HistoryView.render` 함수로 넘김. HistoryView에는 `render` 함수가 따로 존재하지 않지만 KeywordView를 복사하였으므로, 사실 KeywordView의 `render` 함수라고 봐도 무방하다.

```javascript
    renderView() {
        console.log(tag, 'renderView()')
        TabView.setActiveTab(this.selctedTab)

        if (this.selctedTab === '추천 검색어') {
            this.fetchSearchKeyword()
        } else {
            // debugger
            this.fetchSearchHistory()
        }

        ResultView.hide()
    },


    fetchSearchHistory() {
        HistoryModel.list().then(data => {
            HistoryView.render(data)
        })
    },
```



## 최근 검색어 구현 (3)

> [ ] 검색일자, x 버튼을 구현

#### 1) 추가 코드 

몇 파일에서 추가된 코드를 발견함

```javascript
// HistoryView.js
HistoryView.messages.NO_KEYWORDS = '검색 이력이 없습니다'

//KeywordView.js
KeywordView.messages = {
    NO_KEYWORDS: '추천검색어가 없습니다'
}
```


#### 2) HistoryView.js

KeywordView.js의 render 함수를 살펴보면 `getKeywordHtml` 함수를 통해 리스트를 보여주는데 이부분은 HistoryView.js와는 포맷이 조금 다름. => getKeywordHtml 함수를 HistoryView.js에서 오버라이딩하여 재 구현

```javascript
HistoryView.getKeywordsHtml = function (data) {
    return data.reduce((html, item) => {
        html += `<li data-keyword="${item.keyword}"> 
        ${item.keyword}
        <span class="date">${item.date}</span>
        <button class="btn-remove"></button>
        </li>`
        return html
    }, '<ul class="list">') + '</ul>'
}
 
```



## 최근 검색어 구현 (4)

> [ ] 목록에서 x 버튼을 클릭하면 선택된 검색어가 목록에서 삭제



#### 1) 추가 코드 

MainController.js 에서 추가된 코드를 발견함

```javascript
    renderView() {
        console.log(tag, 'renderView()')
        TabView.setActiveTab(this.selctedTab)

        if (this.selctedTab === '추천 검색어') {
            this.fetchSearchKeyword()
            HistoryView.hide() // 추가된 코드
        } else {
            this.fetchSearchHistory()
            KeywordView.hide()  // 추가된 코드
        }

        ResultView.hide()
    },
```



#### 2) HistoryView.js

- x 버튼을 클릭이벤트와 바인딩 함 => `bindRemoveBtn` 함수 정의
- 해당 함수는 버튼을 모두 찾아서 클릭 이벤트를 달아주는 기능을 함.
- 앞서 작성했던 코드와 동일한 방식으로 코드 작성
  - `e.stopPropagation()` 을 통해 클릭의 이벤트 전파를 막음
  - x 버튼이 클릭되었다는 것을 MainController에게 알려주기 위한 함수 `onRemove` 정의
  - 어떤 값을 지울건지. 지울려는 키워드를 보냄(키워드는 btn 위에 있는 li 엘레멘트의 data 어트리뷰트에 있음)

```javascript
HistoryView.bindRemoveBtn = function () {
    Array.from(this.el.querySelectorAll('button.btn-remove')).forEach(btn => {
        btn.addEventListener('click', e => {
            e.stopPropagation()
            this.onRemove(btn.parentElement.dataset.keyword)
        })
    })
}

HistoryView.onRemove = function (keyword) {
    this.emit('@remove', {keyword})
}

```



#### 3) MainController.js & KeywordView.js

`bindRemoveBtn`을 어느시점에서 호출하느냐가 또다른 관건임.

- HistoryView를 render하는 부분을 찾음 => `fetchSearchHistory` 에서 render 함. 이부분에 코드 추가
- render 함수가 호출되고 나면 데이터를 기반으로 DOM이 생성됨. 그 이후 이벤트를 바인딩 할 수 있기  때문에 체이닝을 이용하여 `bindRemoveBtn()` 을 만듦.
- 체이닝을 할려면 `render` 함수가 this를 return 하고 있어야함. render은 HistoryView에 있는 것이 아니라 이 HistoryView가 복사해온 KeywordView에 있음. KeywordView의 render 함수에 `this`를 return 하도록 하자.

```javascript
// MainController.js
fetchSearchHistory() {
    HistoryModel.list().then(data => {
        HistoryView.render(data).bindRemoveBtn()
    })
},
    

// KeywordView.js
KeywordView.render = function (data = []) {
    this.el.innerHTML = data.length ? this.getKeywordsHtml(data) : this.messages.NO_KEYWORDS
    this.bindClickEvent()
    this.show()
    return this
}
```



#### 4) MainController.js

- 추가로 `remove` 함수를 발산하도록 컨트롤러 초기화 부분에 코드 삽입
- `onRemoveHistory` 함수에 keyword를 넘김 & 함수 구현
- 실제로 데이터를 삭제하는 것은 controller가 아니라 model이 데이터를 관리함.
- HistoryModel 내 미리 구현해 놓은 `remove` 함수에 삭제할 keyword를 넘겨줌.
- 이후 다시 `renderView`를 호출해줌

```javascript
export default {
    init() {
        HistoryView.setup(document.querySelector('#search-history'))
            .on('@click', e => this.onClickHistory(e.detail.keyword))
            .on('@remove', e => this.onRemoveHistory(e.detail.keyword))
```



## 최근 검색어 구현 checkout

현재까지 작성한 코드를 기반으로 서버를 실행시키면, TabView의 기본 값은 최근 검색어로 되어있음. 이때 추천 검색어 탭을 클릭하면 설정한 breakpoint에 따라 정지되는데 그 로직은 다음과 같음.

- TabView.setup 함수에 따라 `onChangeTab` 함수가 실행됨.
- `onChangeTab` 함수에는 `debugger`가 설정되어있음.

이 코드를 다시한번 다듬어 보도록 하자.

```javascript
export default {
    // 컨트롤러 초기화 부분
    init() {
        TabView.setup(document.querySelector('#tabs'))
            .on('@change', e => this.onChangeTab(e.detail.tabName))
    },
    
    onChangeTab(tabName) {
        debugger
    },
```

- `onChangeTab` 함수에는 `tabName` 이 인자로 들어옴 => `this.selectedTab`에 `tabName`을 설정해줌
- 이후 다시 `renderView`를 호출함
- 마지막으로 탭구현의 2번 명세 (기본적으로 추천 검색어 탭을 선택한다)에 따라, this.selectedTab의 기본 값을 `this.selectedTab = '추천 검색어'` 으로 다시 변경하도록 하자.



## 최근 검색어 구현 (5)

> [ ] 검색시마다 최근 검색어 목록에 추가된다

#### MainController.js

- 검색을 하게 되면 MainController의 특정 함수로 수렴하게 되어 있음. 컨트롤러 초기화 부분에 따라, 어떠한 view에서 이벤트가 발생하는 경로를 잘 살펴보면 `search` 함수로 도달하게 되는 것을 알 수 있음. search 함수에서 검색한 이력을 추가하면 됨.

```javascript
    search(query) {
        FormView.setValue(query)
        HistoryModel.add(query)
        SearchModel.list(query).then(data => {
            this.onSearchResult(data)
        })
    },
```

