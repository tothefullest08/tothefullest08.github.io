---
layout: post
title: 실습 UI 개발을 통해 배워보는 JS & Vue JS (2) - 검색폼 구현
category: Javascript
tags: [Javascript, Vue]
comments: true
---



> 본 강의는 Inflearn의 김정환 개발자 님의 강의([실습 UI 개발로 배워보는 순수 javascript 와 VueJS 개발]([https://www.inflearn.com/course/%EC%88%9C%EC%88%98js-vuejs-%EA%B0%9C%EB%B0%9C-%EA%B0%95%EC%A2%8C/dashboard](https://www.inflearn.com/course/순수js-vuejs-개발-강좌/dashboard)))를 듣고 배운 내용을 정리한 포스팅 입니다. 



# 실습 UI 개발을 통해 배워보는 JS & Vue JS (2) - 검색폼 구현

## 검색폼 구현 (1)

> 검색 상품명 입력 폼이 위치한다. 검색어가 없는 경우 X 버튼을 숨긴다

#### 1) index.html 코드 작성

- 자동으로 커서가 갈수 있게 `autofocus` 속성적용
- 검색창에 입력한 내용을 삭제하는 기능을 넣기 위해 `button` 태그를 입력 후 type에 `reset` 적용
  - btn-reset 클래스 스타일은 style.css에 이미 구현되어 있음.

```html
<div class="container">
    <form>
        <input type="text" placeholder="검색어를 입력하세요" autofocus>
        <button type="reset" class="btn-reset"></button>
    </form>
</div>
```



>  검색어가 없는 경우이므로 x 버튼을 숨긴다

#### 2) FormView.js

- Form의 view를 담당하는 자바스크립트 모듈(FormView.js)을  views 디렉토리 내 생성
- `Object.create`를 사용하여 View 객체를 복사함
- `setup` 메서드 정의
  - HTML 엘레먼트를 주입받아 내부적으로 속성으로 갖게끔 함(View  함수 중 init이라는 함수 사용)
  - input 엘레멘트와 reset 엘레멘트를 각각 키로 생성
  - 리셋 버튼은 숨기기 위해서 `showResetBtn()` 라는 함수 생성
- `showResetBtn` 함수 정의. `show`라는 boolean 값을 인자로 받음(default값은 true)
  - `resetEl` 의 css의 속성에 접근하여 display 속성을 show 일때는 block을 주고, 그렇지 않은 경우는 none을 주는 방식으로 구현
- FormView를 controller에서 쓰기 위해 export 해줌 (`export default FormView`)

```javascript
// views.js 모듈을 복사해서 사용
import View from './Views.js'

// 디버깅 용도로 tag 생성
const tag = '[FormView]'

// 실제 FormView 생성
const FormView = Object.create(View)

FormView.setup = function (el) {
    this.init(el)
    this.inputEl = el.querySelector('[type=text]')
    this.resetEl = el.querySelector('[type=reset]')
    this.showResetBtn(false)
}

FormView.showResetBtn = function (show = true) {
    this.resetEl.style.display = show ? 'block' : 'none'
}

export default FormView
```



#### 3) MainController.js

- FormView를 가져와 사용 `import FormView from '../view/FormView.js'` 
- init 함수에서 FormView.setup 호출
  - 실제 form 엘레먼트를 넘김

```javascript
import FormView from '../view/FormView.js'

const tag = '[MainController]'

export default {
    init() {
        console.log(tag, 'init()')
        FormView.setup(document.querySelector('form'))
    }
}
```



## 검색폼 구현(2)

> 검색어를 입력하면 x 버튼이 보인다

입력을 했을 때, 이벤트가 발생하고, 이벤트가 발생할 때, 버튼을 보이는 함수가 호출되도록 하면 됨 => FormView.js 내 showResetBtn 함수 호출

#### 1) FormView.js

- 키보드 입력 이벤트에 대하여 bind해야하므로 bind 이벤트 정의.  
- bind 이벤트를 FormView가 시작되는 setup에서 호출

```javascript
FormView.setup = function (el) {
    this.init(el)
    this.inputEl = el.querySelector('[type=text]')
    this.resetEl = el.querySelector('[type=reset]')
    this.showResetBtn(false)
   // bindEvent 호출
    this.bindEvents()
    return this
}
```



- FormView에서 bindEvents 메서드를 정의
- inputEl은 HTML 엘레멘트이므로 addEventListener 함수 사용 가능
  - keyup 이벤트를 입력, 이벤트 받아서 처리할 수 있도록 함.
  - onKeyUp 이라는 함수를 따로 정의해서 이벤트를 넘겨주도록 함.

```javascript
FormView.bindEvents = function() {
    this.inputEl.addEventListener('keyup', e => this.onKeyup(e))
}
```



- onKeyup 메서드는 key가 입력됐을 때 마다 실행될 것임. 여기서 x 버튼을 보여줄건지 말지를 정의
  - showResetBtn 함수에 true 값을 넘겨주면 됨.
  - `inputEl.value.length` : value의 길이를 넘겨주면, 입력한 문자열이 있을 경우 true 값이 넘어감.

```javascript
FormView.onKeyup = function(e) {
    const enter = 13
    this.showResetBtn(this.inputEl.value.length)
} 
```



## 검색폼 구현(3)

> 엔터키를 입력하면 검색 결과가 보인다

#### 1) FormView.js

- 키를 입력하는 부분이 onKeyup 함수. 엔터키인지 아닌지를 구별하는 코드 입력. 

- 엔터키는 keyCode가 13으로 정의되어있음 (enter라고 선언한 상수에 저장)
- 만약에 이벤트의 키코드값이 엔터가 아닌경우 반환을 하고, 엔터인 경우에는 처리를 해줘야함. (엔터키를 입력하면 검색결과가 보인다)
- 검색결과를 보여주는 것이 FormView의 역할인지를 생각해봐야함. FormView는 검색결과를 보여주는 역할을 하지 않음. 대신에 MainController에게 알려주기만하면됨. MainController는 FormView에서 엔터키가 눌러진다는 것을 확인하고, 다른 View에게 검색결과를 보여주도록 요청을 하면됨. (컨트롤러에게 위임)
- 따라서 FormView에서는 엔터키가 눌러졌다라는 이벤트를 MainController에게 알려주기만 하면 됨. 이때 사용 하는 것이 View 모델에 있는 emit이라는 메서드 사용.  
- 첫번째 파라미터로는 우리가 정의한 @submit 이라는 이벤트 정의.

```javascript
FormView.bindEvents = function() {
    this.on('submit', e => e.preventDefault())
    this.inputEl.addEventListener('keyup', e => this.onKeyup(e))
}
```



- 파라미터로는 입력한 값을 전달함. (`this.inputEl.value`)

```javascript
FormView.onKeyup = function(e) {
    const enter = 13
    this.showResetBtn(this.inputEl.value.length)
    if (e.keyCode !== enter) return
    // todo ....
    this.emit('@submit', {input: this.inputEl.value})

} 
```



#### 2) MainController.js

- 컨트롤러에서는 그 이벤트에 대해서 받을 준비가 됨. 
- FormView에 on 메서드가 있음. on 메서드에 우리가 방금 정의했던 @submit 이벤트가 발생했을 때 어떤 함수가 호출되도록 정의해주면 됨. onSubmit 이라는 함수 정의 
- e.detail.input에는 우리가 입력한 값이 넘어오게됨.

```javascript
export default {
    init() {
        console.log(tag, 'init()')
        FormView.setup(document.querySelector('form'))
        .on('@submit', e => this.onSubmit(e.detail.input))
    },

    onSubmit(input) {
        console.log(tag, 'onSubmit()', input)
    }
}
```



- controller에서 setup 함수를 실행한다음 체이닝을 이용하여 on 이라는 함수를 실행했음. 이렇게 할면 setup에서 this를 return 해야함.

```javascript
FormView.setup = function (el) {
    this.init(el)
    this.inputEl = el.querySelector('[type=text]')
    this.resetEl = el.querySelector('[type=reset]')
    this.showResetBtn(false)
    this.bindEvents()
    return this
}
```



## 검색폼 구현(4)

> x 버튼을 클릭하거나 검색어를 삭제하면 해당 결과를 삭제한다.

- x 버튼을 클릭: 클릭이벤트 구현 바인딩 이벤트
- 검색 결과를 삭제는 FormView의 역할이 아님. 메인컨트롤러에 위임. 이벤트 발생시키자. @reset라는 이벤트를 컨트롤러에게 넘기도록 하자.

#### 1) FormView.js

- x 버튼은 reset 엘레멘트임.  x 버튼에 클릭 이벤트를 바인딩하고 `onclickReset` 함수 실행
-  FormView에 `onclickReset` 함수 구현
  - 이 함수에서는 실제 검색결과를 삭제할 필요는 없음. 
  - MainController에 reset이라는 이벤트를 전달하기만 함.
  - 실제 검색결과를 삭제하는 역할을 MainController에게 위임.
  - x 버튼을 누르면 버튼이 사라지게 하는 코드 또한 삽입. 

```javascript
FormView.bindEvents = function() {
    this.on('submit', e => e.preventDefault())
    this.on('reset', e => e.preventDefault())
    this.inputEl.addEventListener('keyup', e => this.onKeyup(e))
    // 코드 추가
    this.resetEl.addEventListener('click', e => this.onclickReset(e))
}

FormView.onclickReset = function(e) {
    this.emit('@reset')
    this.showResetBtn(false)

}
```



#### 2) MainController.js

- `@reset` 인 경우,  컨트롤러가 `onResetForm`이라는 함수 실행
- 작동 여부를 확인하기 위해 출력 코드 삽입

```javascript

export default {
    init() {
        console.log(tag, 'init()')
        FormView.setup(document.querySelector('form'))
        .on('@submit', e => this.onSubmit(e.detail.input))
        .on('@reset', e=> this.onResetForm())
    },

    
    onResetForm(input) {
        console.log(tag, 'onResetForm()')
    }    
```



#### 3) FormView.js

- backspace로 지웠을 때도 검색 결과가 사라지도록 Main Controller 에게 알려주도록 하자.
- inputEl 값의 길이가 false, 즉 없을 경우에는 emit으로 controller에게 `@reset`을 넘김.

```javascript
FormView.onKeyup = function(e) {
    const enter = 13
    this.showResetBtn(this.inputEl.value.length)
    if (!this.inputEl.value.length) this.emit('@reset')
    if (e.keyCode !== enter) return
    this.emit('@submit', {input: this.inputEl.value})

} 
```
