---
layout: post
title: Vue.js 02 - To-do 앱 구현  & Cat API 사용하기
category: Javascript
tags: [Javascript, Vue]
comments: true
---




# To-do 앱 구현 & Cat API 사용하기

## 1. 간단한 To-do 앱 구현 

#### 1) 반복문 & 조건문 동시 활용 하기

- data 내 todo 리스트를 오브젝트 형태로 구현
- Case 1
  - {{ todo. content }}를 통해 content만 출력 할 수 있음
- Case 2
  - 조건문과 반복문을 둘다 사용하여 completed:false 만 출력 가능
  - `v-for` 가 항상 `v-if` 보다 우선순위가 높음
  - else 조건은 `v-else`를 사용하여 표현 가능. 혹은 `v-else-if`로도 표현 가능

```html
<div id="app">
    <ul>
        <!-- case 1 -->
        <li v-for="todo in todos">
            {{ todo.content}}
        </li>
          
        <!-- case 2 -->
        <li v-for="todo in todos" v-if="!todo.completed">
            {{ todo.content }}
        </li>
        <li v-else-if="true">[완료]</li>
        <li v-else> [완료!] </li>
    </ul>
</div>
<script>
    const app = new Vue({
        el: '#app',
        data: {
            todos: [
                {
                    content: '점심메뉴 고민하기',
                    completed: true,
                },
                {
                    content: '사다리타기',
                    completed: false,
                },
                {
                    content: '약속의 2시, 낮잠자기',
                    completed: false,
                },
                {
                    content: '야자하기',
                    completed: false,
                }
            ]
        }
    })
</script>
```



#### 2)  Vue.js 를 활용한 EventListener 구현 (1)

- `v-on`  사용
- html 태그에  "이걸 누르면 실행하라!" 라고 구현 가능.
  -  `v-on:click="todo.completed = true"`  : 클릭이 이루어지면 todo.completed의 값을 true로 변경

```html
<div id="app">
    <ul>
        <li v-for="todo in todos" v-if="!todo.completed" v-on:click="todo.completed = true">
            {{ todo.content }}
        </li>
        <li v-else> [완료!] </li>
    </ul>
</div>
```



#### 3) Vue.js를 활용한 EventListener 구현 (2)

- 더 나아가,  `v-on:click` 의 값으로 methds 내 함수를 호출함으로써 더 간결한 코드를 작성할 수 있음.
- methods 내 `check` 라는 이름을 가진 함수를 정의.  함수를 통해  `todo.completed = !todo.completed` 를 작성.  `todo.completed` 값을 반전시킴(true인 경우는 false로, false인 경우는 true로 반전)
- 완료된 경우, todo 내용이 다시 뜨도록, v-else 조건 뒤에 동일한 이벤트 리스너 적용

```html
<div id="app">
    <ul>
        <li v-for="todo in todos" v-if="!todo.completed" 
            v-on:click="check(todo)">
            {{ todo.content }}
        </li>
        <li v-else v-on:click="check(todo)"> [완료!] </li>
    </ul>
</div>
<script>
    const app = new Vue({
        el: '#app',
        data: {
            todos: [
                {
                    content: '점심메뉴 고민하기',
                    completed: true,
                },
                {
                    content: '사다리타기',
                    completed: false,
                },
                {
                    content: '약속의 2시, 낮잠자기',
                    completed: false,
                },
                {
                    content: '야자하기',
                    completed: false,
                }
            ]
        },
        methods: {
            check: function(todo){
                todo.completed = !todo.completed
            }
        }
    })
</script>
```



#### 4) Vue.js devtools 설치

- Vue.js 디버깅을 위한 chrome 확장 프로그램으로  Vue.js devtools 설치

- chrome - 도구더보기 - 확장프로그램 관리 - 세부정보 - 파일 URL에 대한 엑세스 허용
- F12 눌린 후, 개발자 도구 탭의 끝으로 가면 Vue 라는 새로운 메뉴가 생긴 것을 알 수 있음.

<img src="/assets/javascript/devtools.png" />





## 2. Cat API를 Vue에 적용하기

고양이 사진을 random으로 보여주는 [웹 사이트](<https://thecatapi.com/>)가 있음. 이 사이트는 외부에서도 사용할 수있는 API를 제공하고 있는데, Vue.js 를 통해 API에 요청을 보내고 응답으로 사진을 받는 코드를 동적으로 구현해보자.



#### 1) 기본 코드 작성

- Vue.js & Axios CDN을 head 태그에 추가
- 이미지 크기를 고정시키기 위해 style 태그를 head에 추가한 후, width를 400px으로 고정
- data 내 이미지의 url을 저장시킬 imageURL 추가. methods에서 정의한 함수를 통해 갖고오는 url을 해당 변수에 저장시킴.
- methods에는 cat API를 통해 이미지를 갖고오는 함수를 정의. axios를 사용하자!

```html
<head>
    <script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
    <script src="https://unpkg.com/axios/dist/axios.min.js"></script>
    <style>
        img {
            width: 400px;
        }
    </style>
</head>
<body>
    <div id="app">
    </div>
    <script>
        const app = new Vue({
            el: '#app',
            data: {
                imageURL:'',
            },
            methods: {
                }
            }
        })
    </script>
</body>
```



#### 2) 코드 확장

- 버튼과 사진을 보여줄 위치 지정
- `v-on:click`의 값으로 Vue 인스턴스 메서드 안에 정의한 `getCatImage()` 함수 입력. 이를 통해, 버튼 클릭 시 함수가 실행됨.
- `axios.get`의 인자로, cat API 주소를 삽입함. API 주소의 응답정보를 출력 해보면, resonse.data[0].url에 고양이의 사진 이미지 경로가 저장되어있음을 알 수 있음.  경로를 갖고 온 후, imageURL 값에 저장
- 이때 axios의 return 값인 `.then` 이하는 arrow function으로 구현할 것.
- 이미지의 주소를 변수로 작성할 때는 `v-bind`를 사용해야함. `v-bind`는 html 속성에 vue instance의 값을 사용하기 위한 용도로 쓰임. html 속성(attributes)에는 인터폴레이션(interpolation)으로 값을 직접 넣지 못하므로 디렉티브 `v-bind`를 사용함.

```html
<body>
    <div id="app">
        <button v-on:click="geetCatImage()"> 고양이 사진 보기 </button>
        <div>
            {{ imageURL }}
            <img v-bind:src="imageURL" alt="cat Image">
        </div>        
    </div>
    <script>
        const app = new Vue({
            el: '#app',
            data: {
                imageURL:'',
            },
            methods: {
                getCatImage: function(){
                    axios.get('https://api.thecatapi.com/v1/images/search')
                    	.then((response)=>{
                        	return this.imageURL = response.data[0].url 
                    })
                }
            }
        })
    </script>
</body>
```



#### 3) Arrow Function 내 this

자바스크립트의 this는 일반적인 프로그래밍 언어에서의 this와 조금 다르게 동작함. java this와 python self의 인스턴스의 호출한 대상의 현재 객체를 뜻하는 것(참조)이었다.

하지만 자바스크립트의 function에서는 함수가 어떻게 호출되었는지에 따라 동적으로 동작함.

- 기본적인 함수 선언 후 this 출력 시, 전역 객체(window)를 호출됨.
- 메서드를 선언하고 호출할 경우, 오브젝트의 메서드이므로 오브젝트가 호출됨.
- arrow function에서의 this는 호출과 위치와 상관없이 상위 스코프 this를 가리킴.

위의 예시에서, `.then` 이하의 익명함수를 일반적인 함수의 형태로 구현할 경우, this는 전역 객체인 window를 가리킴. 이러한 불편함을 해결하기 위해 arrow function을 이용하여 `this` (Vue 인스턴스)를 그대로 사용 할 수 있게 됨.