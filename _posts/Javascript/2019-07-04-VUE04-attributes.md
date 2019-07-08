---
layout: post
title: Vue.js 04 - Vue의 속성(Computed, watch) / v-text & v-html / v-if & v-show
category: Javascript
tags: [Javascript, Vue]
comments: true
---



## 1. Vue의 속성(Computed, watch) 

#### 1) Computed

- 어떠한 계산된 결과를 미리 담아 놓는 Vue 클래스의 속성
- 앞 포스팅에서 구현한 간단한 To do 앱 methods 에서 구현한 todoByStatus 함수를 computed로 이동
- 반복문을 `todosByStatus()` => `todosByStatus`로 변경

```html
<!-- 변경 전 -->
<li v-for="todo in todosByStatus()" v-bind:key="todo.id">
<!-- 변경 후 -->
<li v-for="todo in todosByStatus" v-bind:key="todo.id">
```



#### 1-1) Computed 사용 예시

- 새로운 Todo를 나타내는 newTodo를 거꾸로 출력하는 코드를 DOM에서 작성하면 아래와 같음

```html
<span> {{ newTodo.split('').reverse().join('') }}</span>
```

- 그러나 DOM에 어떠한 기능에 대한 로직을 작성하는 것은 일반적으로 권장되지 않으며, 로직에 대한 부분을 바깥쪽, computed 속성에 작성함. 복잡한 자바스크립트 표현식을 HTML에 끼워넣기 보다는 computed 속성안에 코드를 작성하는 것이 선호됨.

```html
<span> {{ reverseNewTodo }}</span>

<script>
    const app = new Vue ({
        // 중략
        computed: {
            reverseNewTodo: function(){
                return this.newTodo.split('').reverse().join('')
            },
        }
    })
</script>
```



#### 2) methods vs computed

- methods와 computed 간 차이의 핵심은 **캐싱의 유무**
- computed는 한번 계산을 해놓고, 계산한 결과를 캐싱해놓음(저장해 놓음). 한번더 함수를 실행시키는 것이 아니라, 저장된 값을 불러와 그대로 사용함.
- 하기 예시에서 Toggle Rendering을 눌러서 사용하면, methods는 보여질때마다 사용함. 그러나 computed는 내부적으로 변화한 부분이 없으므로 이전에 계산했던 값을 그대로 사용함.
- 내부적으로 값을 변경할 때 methods를 사용함.

```html
<div id="app">
    <button v-on:click="visible=!visible"> Toggle Rendering </button>
    <ul v-if="visible">
		<li>dateMethod: {{ dateMethod() }}</li>
        <li>dateComputed: {{dateComputed }}</li>
    </ul>
</div>
<script>
    const app = new Vue ({
        el: '#app',
        data: {
            visible: true,
        },
        methods: {
            dateMethod: function(){
                return new Date()
            },
        },
        computed: {
            dateComputed: function(){
                return new Date()
            }
        }
    })
</script>
```



#### 3) watch

watch 속성은 어떠한 값이 변경디면 함수가 실행되게 할 때 사용함. 어떠한 값이 변경이 되었을 때 이 함수가 오래 걸리거나, 언제 끝날지 모르는 경우에 주로 사용.

[Yes or No API](<https://yesno.wtf/>)를 사용하여 Watch 속성에 대해 살펴보도록 하자.

- 하기 예시의 경우, qeustion 값이 바뀌면, watch 내 question 함수가 실행됨.

- Yes or No API에 요청을 보내는 것을 methods 내 함수로 구현
  - 우리가 입력한 질문이 ?으로 끝날 경우, API에 요청을 보내도록 분기문 적용
  - answer의 값에 API를 통해 반환되는 response.data.answer 값을 입력

```html
<div id="app">
    <h1>질문을 입력하세요</h1>
    <input type="text", v-model="question">
    <p> {{ answer }} </p>
</div>
<script>
    const app = new Vue ({
        el: '#app',
        data: {
            question: '',
            answer: '',
        },
        watch: {
            question: function(question){
                console.log(question)
                this.getAnswer()
            }
        },
        methods: {
            getAnswer: function(question){
                if (this.question[this.question.length-1] === "?"){
                    axios.get('https://yesno.wtf/api')
                    	.then((response)=>{
                        	console.log(response)
                        	this.answer = response.data.answer
                    })
                }
            }
        }
    })
</script>
```

- yesorno API를 통해 yes & no 랜덤 사진도 갖고올 수 있음.
  - data 내 이미지 경로를 담을 imageUrl 정의 
  - API를 통해 return 되는 이미지 경로를 imageUrl에 저장(`this.imageUrl = response.data.image` )

```html
<body>
    <div id="app">
        <h1>질문을 입력하세요</h1>
        <input type="text" v-model="question">
        <p> {{ answer }}</p>
        <img v-bind:src="imageUrl">
    </div>
    <script>
        const app = new Vue({
            el: '#app',
            data: {
                question: '',
                answer: '',
                imageUrl: '',
            },
            watch: {
                question: function (question) {
                    console.log(question)
                    this.answer='생각중입니다....'
                    this.getAnswer()
                }
            },
            methods: {
                getAnswer: function (question) {
                    if (this.question[this.question.length - 1] === "?") {
                        axios.get('https://yesno.wtf/api')
                            .then((response) => {
                                console.log(response)
                                this.answer = response.data.answer
                                this.imageUrl = response.data.image
                            })
                    }
                }
            }
        })
    </script>
</body>
```





## 2. v-text vs v-html  

- `v-text` 는 그 값을 단순문자열로 취급함.  하기 코드의 경우, `{{ header }}`를 썼을 때와 같은 결과를 보여줌 - `'<h1> 제목입니다 </h1>'`  
- `v-html`의 경우, 실제 HTML으로 rendering하여 화면에 보여줌.





## 3. 조건부 렌더링(v-if vs v-show)

- `v-if`는 조건을 만족하지 못할 경우, 코드가 html에서 사라짐
- `v-show`의 경우, 속성값을 적용하여 안보이게 하나, 코드는 남아있음.  `v-show`가 있는 엘리먼트는 항상 렌더링 되고 DOM에 남아있음.  `v-show`는 단순히 엘리먼트에 `display` CSS 속성을 토글함.

```html
<div id="app">
    {{ header }}
    <div v-text="header"></div>
    <div v-html="header"></div>

    <h1 v-if="visible">v-if</h1>
    <h1 v-show="visible">v-show</h1>

</div>
<script>
    const app = new Vue({
        el: '#app',
        data: {
            header: '<h1> 제목입니다 </h1>',
            visible: true,
        }
    })

</script>
```

