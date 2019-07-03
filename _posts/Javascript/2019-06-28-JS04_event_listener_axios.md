---
layout: post
title: JavaScript 04 - Event Listener & Axios를 통한 요청보내기
category: Javascript
tags: [Javascript]
comments: true
---



# Event Listener & Axios를 통한 요청보내기

## 1. Event Listener: [무엇]을 [언제], [어떻게]한다.

Event Listener란 이벤트가 발생했을 때 그 처리를 담당하는 함수를 가리킴. 지정된 타입의 이벤트가 특정 요소에서 발생하면, 웹 브라우저는 그 요소에 등록된 Event Listener를 실행시킴.

Event Listener는 말그대로 해당 Event에 대해 대기중인 것을 말함. 항상 "Listen"하고 있는 상태라고 할 수 있음. 사용자가 지정한 Event가 발생했을 때, 등록한 Event  Listener가 실행됨. 여기서 Event는 " [무엇] 을 [언제], [어떻게] 한다. " 라고 정의 할 수 있음.  

버튼을 클릭하면 메세지가 나온다" 를 event Listener를 통해 구현해보도록 하자.

#### 1) DOM Selector

- `querySelector`
- `querySelectorAll()`

#### 2) Event Listener 등록

"특정한 DOM element**[무엇]** 이 어떠한 행동을 했을 때**[언제] **, 어떻게 한다**[어떻게]**"

- id값은 html 문서에서 유일하게 존재하는 고유값으로, id값을 바탕으로 버튼을 갖고와 변수에 저장시킨다. `document.querySelector('#this-button')`
- `addEventListener`  매개변수 
  - 1번째 인자: 반응할 이벤트의 유형을 나타냄. `click` 은 마우스 클릭시 이벤트가 실행됨.
  - 2번째 인자: 지정한 타입의 이벤트가 발생했을 때, 알림을 받는 객체.  하기 예시의 경우 callback 함수를 받음.
- `.innerHTML` : 요소 내 포함된 HTML을 가져올 때 사용
- Event Listener에서의 콜백함수에는 arrow function 을 사용하지 않음.

```html
<body>
    <div id="my"></div>
    <button id="this-button"> Click me </button>
    <script>
        /*
            Event Listener
            [무엇]을 [언제] [어떻게]한다.
            버튼을 클릭하면 메세지가 나온다.
        */
    
    	// 1. 무엇 -> 버튼
        const button = document.querySelector('#this-button')
        
        // 2. 언제 -> 버튼을 '클릭' 하면
        button.addEventListener('click', function(event){
            const area = document.querySelector('#my')
            area.innerHTML = '<h1>뿅!</h1>'
        })
    </script>
</body>
```



#### 3) 이벤트의 유형

- `click` – 마우스버튼을 클릭하고 버튼에서 손가락을 떼면 발생한다. 
- `mouseover` – 마우스를 HTML요소 위에 올리면 발생한다. 
- `mouseout` – 마우스가 HTML요소 밖으로 벗어날 때 발생한다. 
- `mousemove` – 마우스가 움직일때마다 발생한다. 마우스커서의 현재 위치를 계속 기록하는 것에 사용할 수 있다. 
- `keypress` – 키를 누르는 순간에 발생하고 키를 누르고 있는 동안 계속해서 발생한다. 
- `keydown` – 키를 누를 때 발생한다. 
- `keyup` – 키를 눌다가 떼는 순간에 발생한다. 
- `load` – 웹페이지에서 사용할 모든 파일의 다운로드가 완료되었을때 발생한다. 
- `scroll` – 스크롤바를 드래그하거나 키보드(up, down)를 사용하거나 마우스 휠을 사용해서 웹페이지를 스크롤 할 때 발생한다. 페이지에 스크롤바가 없다면 이벤트는 발생하지 않다. 
- `change` – 폼 필드의 상태가 변경되었을 때 발생한다. 라디오 버튼을 클릭하거나 셀렉트 박스에서 값을 선택하 는 경우를 예로 들수 있다. 
- `input` - input 또는 textarea 요소의 값이 변경되었을 때 
- `submit` - form을 submit 할 때



## Axios를 이용한 요청 보내기(+ EventListener)

자바스크립트를 통해서 요청을 보내 보도록 하자. (강아지 사진을 random으로 가지고 오는 [API](https://dog.ceo/dog-api/)활용)

API에 요청을 보낼 때, 순수한 자바스크립트로 요청을 보내는 코드는 매우 지저분함.  (XHR 객체를 통함). 따라서 axios를 사용해보도록 하자. axios는 XHR(XMLHttpRequest)를 보내주고 그 결과를 promise 객체로 반환해주는 라이브러리이다. 

#### 1) 기본 코드 작성

- 외부 라이브러리인 Axios를 사용하기 위해 \<head> 태그에 CDN 삽입
- EventListener를 이용하여, 마우스 클릭 시, 강아지 사진을 random으로 갖고 오게 코드를 작성
- `axios.get()` 으로 요청을 보내면,  `.then` 이하로 응답을 함.
- 요청이 들어오면,  `.then` 이하의 익명함수를 실행 시킴. 그렇지 않을 경우, `.catch` 이하의 익명함수를 실행.

```html
<head>
	<!-- Axion CDN 추가 -->
    <script src="https://unpkg.com/axios/dist/axios.min.js"></script>
</head>
<body>
    <button id="dog">Dog!</button>
    <script>
        const button = document.querySelector('#dog')
        button.addEventListener('click', function(event){
            //API로 요청을 보냄
            axios.get('https://dog.ceo/api/breeds/image/random')
            	.then(function(response){
                	//handle sucess
                	console.log(response);
            })
            	.catch(function(error){
                	//handle error
                	console.log(error);
            })
        })
    </script>
</body>
```



※자바스크립트는 기본적으로 모든 코드가 비동기적으로 동작함. 시간이 오래걸리는 코드가 중간에 있다고 가정할경우(응답이 언제 올지 모르는 코드가 실행되었다 가정),  자바 스크립트는 그 코드가 끝나길 기다리지 않고 그다음 코드를 바로 실행시킴. 

비동기적으로 작동하지 않으면(싱글 스레드이므로), 이 코드가 실행될 때 까지 브라우저는 동작하지 않고 멈추어버리기 때문임. 따라서 요청을 기다리지 않고, 브라우저가 작동할 수 있도록 비동기적으로 동작해야함. 

따라서 비동기적으로 동작하는 것을 막기 위해 `.then` 으로 이어줌. 위의 코드는 가독성을 위해 줄바꿈을 하였지만 실제로 `.then` 과 `.catch` 등은 앞의 코드와 이어진 총 한줄의 코드라고 할 수 있음.



#### 2) console.log 확인

- `console.log(response)` 의 결과 값을 개발자도구 console 창을 통해 확인되는 값은 아래와 같음 (오브젝트 객체로 반환됨을 알 수 있음)
- 여기서 우리가 얻고자 하는 이미지는 data 내 message의 값으로 저장되어있음.
- 따라서, `response.data.message` 로 접근하여 이미지를 가지고 올 수 있음.

```javascript
{data: {…}, status: 200, statusText: "", headers: {…}, config: {…}, …}
config: {adapter: ƒ, transformRequest: {…}, transformResponse: {…}, timeout: 0, xsrfCookieName: "XSRF-TOKEN", …}
data: {status: "success", message: "https://images.dog.ceo/breeds/briard/n02105251_6387.jpg"}
headers: {content-type: "application/json", cache-control: "no-cache, private"}
request: XMLHttpRequest {onreadystatechange: ƒ, readyState: 4, timeout: 0, withCredentials: false, upload: XMLHttpRequestUpload, …}
status: 200
statusText: ""
__proto__: Object
```



#### 3) 코드 확장

`.then` 은 앞의 코드가 끝나면 뒤의 코드가 실행되게 하는 메소드임. 앞서 설명한 것 처럼, 비동기 처리를 막기 위해 사용하는 메소드인데, `.then` 이하의 코드가 끝난 후, 다른 코드가 실행 시키고 싶을 경우, `.then` 을 이어서 또 사용할 수 있음. 이 때,  앞의 익명함수의 return 값이 함수의 인자로 들어감.

`.then`을 여러번 사용하는 경우는 앞의 함수와 뒤의 함수가 처리하는 것이 다를 경우를 의도적으로 구분하기 위해서 사용하는 것이 일반적임.

한단계 더 나아가, `.then` 은 모든 함수의 메서드로 쓸 수 있는 것은 아님.  `.then` 사용 유무는 get의 return값이 무엇이냐에 따라 결정됨. `axios.get`의 return 값은 `Promise` 객체임. 이는 응답을 보내주겠다는 약속을 하는 것! 수행되는 시간이 보장되지 못할 때 사용되며, `.then` 메소드는 Promise 객체의 메소드임. (`.catch` 도 동일함)

- `response.data.message` 를 통해 이미지 주소를 갖고 올 수 있음.
- 두번째 콜백함수의 인자인 url은 바로 위 함수의 return 값인 `response.data.message`를 갖고 있음.
- `document.createElement()` : 새로운 태그를 만듦. 인자에 img를 넣어 img 태그르 만든 후 새롭게 선언한 변수에 저장함.
- `imgTag.src = url;` : img 태그에 경로를 설정하는 속성인 src에 들어오는 네임인자인 url을 저장시킴.
- `document.querySelector('body').append(imgTag)` : 이후, body 태그에 imgTag 값을 append함.

```html
<head>
	<!-- Axion CDN 추가 -->
    <script src="https://unpkg.com/axios/dist/axios.min.js"></script>
</head>
<body>
    <button id="dog">Dog!</button>
    <script>
        const button = document.querySelector('#dog')
        button.addEventListener('click', function(event){
            //API로 요청을 보냄
            axios.get('https://dog.ceo/api/breeds/image/random')
            	.then(function(response){
                	//handle sucess
                	console.log(response);
                	return response.data.message
            })
            	.then(function(url){
                	const imgTag = document.createElement('img')
                    imgTag.src = url;
                	document.querySelector('body').append(imgTag)
            })
            	.catch(function(error){
                	//handle error
                	console.log(error);
            })
        })
    </script>
</body>
```