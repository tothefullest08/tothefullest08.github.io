---
layout: post
title: JavaScript 04 - 
category: Javascript
tags: [Javascript]
comments: true
---



# JavaScript 04

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

