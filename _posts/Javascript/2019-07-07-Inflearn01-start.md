---
layout: post
title: 실습 UI 개발을 통해 배워보는 JS & Vue JS (1) - 준비, 순수 JS 시작하기
category: Javascript
tags: [Javascript, Vue]
comments: true
---



> 본 강의는 Inflearn의 김정환 개발자 님의 강의([실습 UI 개발로 배워보는 순수 javascript 와 VueJS 개발]([https://www.inflearn.com/course/%EC%88%9C%EC%88%98js-vuejs-%EA%B0%9C%EB%B0%9C-%EA%B0%95%EC%A2%8C/dashboard](https://www.inflearn.com/course/순수js-vuejs-개발-강좌/dashboard)))를 듣고 배운 내용을 정리한 포스팅 입니다. 




# 1. 준비

## 1. 개발 환경 구성

- Node.js
- 에디터: VS CODE
- Git (코드형상관리)
- Chrome 최신버젼
- Lite Server - 개발 서버. 홈페이지를 개발시, 개발용 서버로 사용되는 툴
  - Global 설치

```bash
$ npm install -g lite-server
 
# To run: 
$ lite-server
```



## 2. 요구사항 분석

- 검색폼 구현
- 검색 결과 구현
- 탭 구현
- 추천 검색어 구현
- 최근 검색어 구현





# 2. 순수 JS (MVC) 시작하기

## 1. MVC 패턴 설명

MVC는 Model, View, Controller으로 구성되어있음.

- Model: 데이터를 관리하는 역할. DB 데이터를 갖고와서 또다른 객체에게 전달하는 역할. 반대로 외부 객체로부터 입력데이터를 받아서 넣는 역할도 함.  웹프론트에서 모델의 역할은 데이터베이스에 직접 접근하지 않고, API의 형태로 접근함 API의 형태로 데이터를 갖고와 다른 객체에 전달하는 역할을함. 외부 객체로부터 데이터를 입력받으면, 그 데이터를 백엔드 API를 통해서 호출하는 역할을 함. 
- View: 데이터를 가지고 화면을 관리하는 역할, 모델의 데이터를 화면에 그리는 역할을 함. (HTML, CSS, JS로 구성되어있음). 사용자가 입력한 데이터를 처리하는 역할도 함. 입력하면 입력이벤트를 받아서 입력된 값을 다를객체에게 전달하는 역할을 함. 
- 모델과 뷰는 직접적으로 연결되어 있지 않음. 이 둘을 연결하는 역할을 하는게 controller. 컨트롤러는 모델로 부터 데이터를 갖고오고 갖고온 데이터를 뷰에게 전달해줌. 반대로 뷰로부터 사용자입력데이터를 얻고 그 데이터를 모델에게 전달하는역할도 함. 컨트롤러는 모델과 뷰를 둘다 관리하는 역할을 함.



## 2. 기본 폴더 구조 

#### 1) 기본 폴더 구조

```
│  index.html
│  style.css
│  
└─js
    │  app.js
    │  
    ├─controllers
    │      MainController.js
    │      
    ├─models
    │      HistoryModel.js
    │      KeywordModel.js
    │      SearchModel.js
    │      
    └─views
            View.js
```



#### 2) views/View.js 

- 앞으로 만들게 될 Vue는  View.js module을 기반으로 만들게 됨.

```javascript
const tag = '[View]'

export default {

  // init은 el(element)를 주입받음.
  init(el) {
    if (!el) throw el
    // el을 자신의 property로 받게됨
    this.el = el
    return this
  },

  // On: 특정 이벤트에 대한 행동을 정의함. 이벤트와 핸들러를 같이 인자로 받음. 
  on(event, handler) {
    // 현재 가지고 있는 엘레멘트에서 특정 이벤트가 발생했을 때 handler가 실행되도록하는 역할
    // handler: vue를 사용 시, vue에서 나온 이벤트를 어떻게 핸들링 할 것인가를 위해서 사용을 한 것
    this.el.addEventListener(event, handler)
    return this
  },

  // Emit: 스스로 이벤트를 방출하는 기능을 함. 이벤트와 데이터를 인자로 받음.
  emit(event, data) {
    // customEvent는 첫번째 인자로 event 이름을 받고, 
    // 2번째 인자로 디테일 키를 갖고 있는 데이터 객체를 받음
    const evt = new CustomEvent(event, { detail: data })
    
    // HTML el(element)에는 dispatchEvent라는 메서드가 있음. 인자로 방금 만든 evt 객체를 넘겨줌.
    // 이를 통해 el는 이벤트를 방출함.
    this.el.dispatchEvent(evt)
    return this
  },

  // css 
  hide() {
    // css 속성 중 display 값이 none으로 주면 hide가 됨
    this.el.style.display = 'none'
    return this
  },

  show() {
   	// css 속성 중 display 값을 주지않으로 주면 show
    this.el.style.display = ''
    return this
  }
}
```



#### 3) models/HistoryModel.js

- 검색 히스토리에 대한 데이터를 관장함.

```javascript
export default {
  // 데이터를 collecction의 형태로 갖고 있음.
  // keyword: 검색어, data: 검색날짜
  data: [
    { keyword: '검색기록2', date: '12.03' },
    { keyword: '검색기록1', date: '12.02'},
    { keyword: '검색기록0', date: '12.01' },
  ],

  // ㅣist 메서드는 데이터를 return 해줌. 특별하게는 Promise 패턴을 이용함. return this.data를 하지 않고 Promise 패턴을 사용한 이유는 history 모델의 경우 서버에서 비동기로 갖고 오는 경우도 있고, 쿠키로도 데이터를 얻을 수도 있기 떄문에 공통적으로 사용하기 위해 Promise 패턴 사용
  list() {
    return Promise.resolve(this.data)
  },
  
 // 추가될 검색어를 받아(keyword='') 실제 데이터가 데이터에 있는지를 체크한 후, 있으면 삭제, 다시 날짜를 산정하여 기존데이터와 합쳐서 추가하는 기능
  add(keyword = '') {
    keyword = keyword.trim()
    if (!keyword) return 
    if (this.data.some(item => item.keyword === keyword)) {
      this.remove(keyword)
    }

    const date = '12.31'
    this.data = [{keyword, date}, ...this.data]
  },
  
 // keyword를 받아서 그 키워드에 해당하는 것들을 삭제하는 기능
  remove(keyword) {
    this.data = this.data.filter(item => item.keyword !== keyword)
  }
}
```



#### 4) models/KeywrodModel.js

- 추천 검색어를 보여주기 위한 모델

```javascript
export default {
  // 데이터를 콜렉션의 형태로 갖고 있음. keyword만 갖고 있음.
  data: [
    {keyword: '이탈리아'}, 
    {keyword: '세프의요리'}, 
    {keyword: '제철'}, 
    {keyword: '홈파티'}
  ],

  // 리스트 메서드만 갖고 있음. Promise 패턴을 이용. setTimeout을 이용하여 200ms 이후 데이터를 반환하도록 구현
  list() {
    return new Promise(res => {
      setTimeout(() => {
        res(this.data)
      }, 200)
    })
  }
}

```



#### 5) models.SearchModel.js

- 검색하는 모델

```javascript
const data = [
  {
    id: 1,
    name: '[키친르쎌] 홈메이드 칠리소스 포크립 650g',
    image: 'https://cdn.bmf.kr/_data/product/H1821/5a4ed4e8a6751cb1cc089535c000f331.jpg'
  },
  {
    id: 2,
    name: '[키친르쎌] 이탈리아 파티 세트 3~4인분',
    image: 'https://cdn.bmf.kr/_data/product/H503E/300d931e3b8252ed628b6a3c2f56936b.jpg'
  }
]

export default {
  // 리스트 메서드에 쿼리를 인자로 받아 쿼리에 해당하는 데이터를 return 하도록 코드 구현
  // setTimeout을 이용하여 200ms 이후 데이터를 반환하도록 구현
  list(query) {
    return new Promise(res => {
      setTimeout(()=> {
        res(data)
      }, 200);
    })
  }
}
```



#### 6) 서버 실행

- Root directory 에서 `lite-server` 로 서버 실행



#### 7) Header 구현

- index.html > body > 내 div & header를 통해 헤더 구현. 
- h2 태그에 container 클래스 속성 적용

```html
<body>
  <div>
    <header>
      <h2 class="container">검색</h2>
    </header>
  </div>
  <script type="module" src="./js/app.js"></script>
</body>
</html>
```



#### 8) Controllers 생성

- js 디렉토리 아래 controllers 디렉토리 생성 및 MainController.js 파일 생성
- debugging을 위해 sample code 입력
- app.js에서 MainController.js를 import 한 후, DOM이 전부 로드 되는 이벤트가 발생하였을 때, MainController의 생성자메서드가 실행되도록 EventListener 구현

```javascript
// MainController.js
const tag = '[MainController]'

export default {
    init() {
        console.log(tag, 'init()')
    }
}

//app.js
import MainController from './controllers/MainController.js'

document.addEventListener('DOMContentLoaded', ()=>{
    MainController.init()
})
```
