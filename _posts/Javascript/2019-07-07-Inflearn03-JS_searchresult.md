---
layout: post
title: 실습 UI 개발을 통해 배워보는 JS & Vue JS (3) - 검색결과 구현
category: Javascript
tags: [Javascript, Vue]
comments: true
---



> 본 강의는 Inflearn의 김정환 개발자 님의 강의([실습 UI 개발로 배워보는 순수 javascript 와 VueJS 개발]([https://www.inflearn.com/course/%EC%88%9C%EC%88%98js-vuejs-%EA%B0%9C%EB%B0%9C-%EA%B0%95%EC%A2%8C/dashboard](https://www.inflearn.com/course/순수js-vuejs-개발-강좌/dashboard)))를 듣고 배운 내용을 정리한 포스팅 입니다. 



# 실습 UI 개발을 통해 배워보는 JS & Vue JS (3) - 검색결과 구현

## 검색 결과 구현 (1)

> 검색 결과가 검색폼 아래 위치한다

#### 1) index.html

- FormView와 다르게 검색결과는 미리 데이터가 미리 정해져있지 않음. 검색 데이터를 서버에 요청하고 서버가 준 검색데이터를 받아서 동적으로 보여줘야함.
- 검색 결과를 위한 ResultView가 mount 될 수 있는 엘레멘트만 넣도록 하면 됨.
- 기본 틀 생성 후, ResultView.js 생성

```html
<div class="container">
    <div id="search-result"></div>
</div>
```



#### 2) ResultView.js

- index.html > search-result 라는 엘레멘트에 무언가를 출력하도록 구현
- FormView와 마찬가지로 View.js를 이용하여 코드 작성
- 디버깅을 위한 태그 정의 및 View 객체를 이용하여 복사
- Setup 메서드 정의
  - 엘레멘트를 주입받아 내부 속성으로 엘레멘트를 갖고 있음 (그 역할은 init 메서드가 수행)

```javascript
import View from '.View.js'

// 디버깅을 위한 태그 정의
const tag = '[ResultView]'

// View 객체를 이용하여 복사
const ResultView = Object.create(View)


// Setup 메서드 정의
// 엘레멘트를 주입받아 내부 속성으로 엘레멘트를 갖고 있음.
// (그 역할은 init 메서드가 수행)
ResultView.setup = function(el) {
    this.init(el)
}
```



#### 3) MainController.js

- ResultView를 MainController에서도 사용할 수 있게 코드 추가
- 컨트롤러를 초기화하는 init 메서드에서 ResultView를 setup

```javascript
import FormView from '../views/FormView.js'
import ResultView from '../viewsResultView.js'

const tag = '[MainController]'

export default {
    // 컨트롤러 초기화 부분
    init() {
        FormView.setup(document.querySelector('form'))
            .on('@submit', e => this.onSubmit(e.detail.input))
            .on('@reset', e=> this.onResetForm())
        
        ResultView.setup(document.querySelector('#serach-reesult'))  
    },
```



#### 4) ResultView.js

- 실제 검색결과가 그려지기 위해 ResultView에 render 함수 정의
- render 함수는, 서버로부터 받아온 검색결과 데이터를 동적으로 그려주는 역할을 함.
  - data는 컬렉션으로 받으며, 없는 경우는 빈배열로 기본값 설정
  - 엘레멘트의 innerHTML을 통해 데이터를 받도록 하자, 2가지 경우가 존재(데이터가 있는/없는 경우)
  - 배열의 길이가 있는 경우 => 실제 그 데이터로 그리도록 함.(그리는 역할은 `getSearchResultsHtml` 이라는 함수로 뽑을 예정)
  - 데이터가 없는 경우,  문구 삽입
- `getSearchResultsHtml` 는 데이터를 받아 debugger를 걸어 멈추도록 하자
- ResultView를 컨트롤러에서 사용할 수 있도록 모듈을 export 

```javascript
//HTML DOM을 만들어내는 함수
ResultView.render = function(data = []) {
    console.log(tag, 'render()', data)
    this.el.innerHTML = data.length ? this.getSearchResultsHtml(data) : '검색 결과가 없습니다'
}

ResultView.getSearchResultsHtml = funtion(data) {
    debugger
}

export default ResultView;
```



#### 5) MainController.js

HTML DOM을 만들어내는 render 함수를 MainController가 호출해야됨. 그시점에대해서 생각해보도록 하자.

- Form에서 입력이 발생 했을 때, `@submit` 이벤트 발생 -> `@submit` 이벤트를 처리하는 것이 `onSubmit` 함수 -> `onSubmit` 함수는 입력데이터를 받아서 검색 요청을 하게 될 것임  ->  검색요청하는 메서드 `search` 를 `onsubmit` 함수 내에서 호출 
- `Search` 함수는 search API를 백엔드로 호출. 그 결과를 받아 `onSearchResult`가 처리. 뭔가 데이터를 받아 넘겨주도록 할 것임.
- `onSearchResult` 함수는 서버로 부터 받은 데이터를 ResultView.js > `render` 메서드로 넘김

```javascript
import FormView from '../views/FormView.js'
import ResultView from '../viewsResultView.js'

const tag = '[MainController]'

export default {
    init() {
        FormView.setup(document.querySelector('form'))
            .on('@submit', e => this.onSubmit(e.detail.input))
            .on('@reset', e=> this.onResetForm())

        ResultView.setup(document.querySelector('#serach-reesult'))  
    },

    // step 2
    search(query) {
        console.log(tag, 'search()', query)
        // search API
        this.onSearchResult([])

    },

    onSubmit(input) {
        console.log(tag, 'onSubmit()', input)
        // Step 1
        this.search(input)
    },

    onResetForm(input) {
        console.log(tag, 'onResetForm()')
    },

    // step 3
    onSearchResult(data) {
        ResultView.render(data)
    },
}
```



#### 6) 검색 결과 프로세스 Summary

1. FormView에서 `@submit` 이벤트가 발생하였을 때(엔터키가 눌러졌을 때), onSumit 함수가 구동
2. 검색 요청을 위해 search 함수 실행
3. search 함수는 search API를 통해 데이터를 얻어옴.
4. 그 데이터를 받아서 onSearchResult를 실행함
5. onSearchResult는 데이터를 받아 ResultView.js 의 render 함수로 넘겨줌
6. render 함수는 검색 결과를 그림



#### 7) 실제 API 호출하기 

- models/SearchModel.js 내 list 함수를 이용하여 데이터를 갖고 오도록 하자.
- MainController에서 SearchModel import
- 검색하는 search 함수 내에서 SearchModel의 list를 호출함. 이때 검색어를 넘겨줄 것.
  - 이 list는 Promise를 반환하기 때문에 then 함수를 쓸 수 있음. then 함수의 검색결과로 data가 넘어오고, 그 data에다가 onSearchResult의 data로 넘겨줌.

```javascript
import SearchModel from '../models/SearchModel.js'

export default {
    /// 중략 
    search(query) {
        console.log(tag, 'search()', query)
        SearchModel.list(query).then(data => {
            this.onSearchResult(data)
        })

    },
```



위의 코드대로 서버를 실행시켜 검색어를 입력한 후 검색을 하면 `ResultView.getSearchResultHtml` 에서 멈춤. 이말은 ResultView에서 render를 그릴 때, data의 length값이 없었기 때문에, "검색 결과가 없습니다" 라는 결과가 떴지만, 이번에는 data의 length가 있기 때문에, `getSearchResultHtml` 가 실행되었고, debugger로 인해 break point로 멈춘 것을 확인 할 수 있음.



## 검색 결과 구현 (2)

> 검색 결과가 보인다.

실제 검색 결과를 보여주기 위해서 ResultView.js 내 getSearchResultsHtml 함수를 구현해보도록 하자.

#### 1) ResultView.js

- innerHTML의 값으로 들어가므로, HTML 문자열만 return 해주면 됨. 데이터는 collection 이므로, reduce 함수를 이용
- 초기값으로 `'<ul>' `을 넘겨주고, 함수가 끝난 후 닫는 ul 태그를 달아줌.
- reduce 콜백함수의 첫번째 인자로 들어가는 누적값에다가 li 태그 및 값을 만들어주면 됨.
- li를 만들어내는 `getSearchItemHtml` 함수를 만들고 `item` 데이터를 넘김.
- `getSearchItemHtml`
  - li 태그로 이루어진 문자열을 반환함
  - 검색 결과에는 상품 이미지와 상품 제목이 들어가므로, 이미지 태그와 p 태그를 이용하여 이미지와 제목을 출력

```javascript
ResultView.getSearchResultsHtml = function(data) {
    return data.reduce((html, item) => {
        html+= this.getSearchItemHtml(item)
        return html
    }, '<ul>') + '</ul>'
}

ResultView.getSearchItemHtml = function (item) {
    return `<li>
    <img src="${item.image}">
    <p>${item.name}</p>
    </li>`
}
```



## 검색 결과 구현 (3) - 실습

> x버튼을 클릭하면 검색폼이 초기화 되고, 검색 결과가 사라진다

#### 1) MainController.js

- 입력한 검색어를 초기화 시켜주는 `onResetForm` 함수를 활용하여 검색결과를 없앨 수 있음.
- ResultView 함수의 hide 메서드 사용
- ResultView의 hide는 ResultView가 복사한 View 함수 내 정의되어있는 메서드임.

```javascript
    onResetForm(input) {
        console.log(tag, 'onResetForm()')
        ResultView.hide()
    },
```



#### 2) 나의 코드

##### 2-1) MainController.js

- OnResetFrom 함수에서 ResultView 내 reset이라는 함수를 호출시킴 

```javascript
    onResetForm(input) {``
        console.log(tag, 'onResetForm()')
        ResultView.reset()
    },
```



##### 2-2) ResultView.js

- 엘레멘트의 innerHTML의 값을 초기화시킴

```javascript
ResultView.reset = function(){
    console.log(tag, 'reset()')
    this.el.innerHTML = ''

}
```



