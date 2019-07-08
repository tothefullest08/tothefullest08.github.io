---
layout: post
title: Vue.js 03 - To do 앱 확장하기
category: Javascript
tags: [Javascript, Vue]
comments: true
---




## To do 앱 확장하기

#### 1) `v-model` 디렉티브

- data 내 새로운 todo의 데이터가 저장될 `newTodo`를 추가
- 새로운 todo를 입력한 `input` 태그를 body 내에 추가

- `v-model` 디렉티브를 통해 input 태그에서 입력된 값이 Vue 인스턴스 내 data 안의 newTodo와 동기화시킬 수있음 (양방향 데이터 바인딩 생성)

```html
<body>
    <div id="app">
	    <!-- 중략 -->
            <div>
                <input type="text" v-model="newTodo">
            </div>
    </div>
    <script>
        const app = new Vue({
            el: '#app',
            data: {
                newTodo: '',
               // 중략
            },
        })
    </script>
</body>
</html>
```



#### 2) To do 리스트에 새로운 항목 추가하기

JS 코드를 이용하여 newTodo에 적힌 값을 todos에 값을 추가해보도록 하자

- vue 클래스 내 methods에 addTodo 함수 생성
  - `input` 내 입력된 값이 todo 리스트에 추가하도록 코드 구현
  - `newTodo`와 `input` 태그가 양방향 바인딩 되어있는 것을 활용하여 `newTodo=""` 을 입력하여 `input` 태그 안의 값을 초기화 시킴
- input 태그 내 `v-on:keyup.enter` 을 사용하여, 엔터키를 눌렀을 때, 이벤트가 발생하게 지정
  - `enter` / `tab` / `delete` 등을 `keyup. `이하에 붙일 수 있으며, keycode(숫자)로도 표현이 가능함
  - 또는 button 태그와 마우스 클릭을 이용하여 이벤트가 발생하게도 지정할 수 있음.

```html
<body>
    <div id="app">
        <ul>
            <li v-for="todo in todos" v-if="!todo.completed" v-on:click="check(todo)">
                {{ todo.content }}
            </li>
            <li v-else v-on:click="check(todo)"> [완료]</li> 
        </ul>
        <div>
            <input type="text" v-model="newTodo" v-on:keyup.enter="addTodo">
            <button v-on:click="addTodo"> 추가하기 </button>
        </div>
    </div>
    <script>
        const app = new Vue ({
            el: '#app',
            data: {
                newTodo: ""
                todos : [
                    {
                        content: '점심메뉴 고민하기',
                        completed: true,
                    },
                    {
                        content: '사다리타기',
                        completed: false,
                    },
                    {
                        content: '약속의 2시 낮잠자기',
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
                },
                addTodo: function(){
                    this.todos.push({
                        content: this.newTodo,
                        completed: false,
                    })
                    this.newTodo = ""
                }
            }
        })
    </script>
</body>
```



#### 3) To do 리스트에 삭제 기능 구현

- `filter` 메서드를 이용하여 원본데이터를 건들지 않고 새로운 배열을 만듦.
- 완료 된 것을 삭제한 것이 아니라, 완료되지 않은 것들을 `filter`로 보여주는 방식으로 구현
- `todos`를 `notCompletedTodos`로 새로 저장
- 익명 함수 내 새로운 함수로 구현할 때는 `Arrow Function`을 써야함

```html
<footer>
    <button v-on:click="clearCompleted">Clear Completed</button>
</footer>
```

```javascript
methods: {
    clearCompleted: function(){
        const notCompletedTodos = this.todos.filter((todo)=>{
            return !todo.completed
        })
        this.todos = notCompletedTodos
    },
}
```



#### 4) 완료 표시 형태 변경

- `v-model` 에 `"todo.completed"` 을 적용하여, 체크박스와 completed를 동기화시킴.
- 하나의 체크박스는 단일의 boolean 값을 가지는 것을 활용하여, completed의 값이 true일 때, 체크박스에 체크가 되도록 구현

```html
<div id="app">
    <!-- 체크박스 사용 코드 -->
    <li v-for="todo in todos">
        <input type="checkbox" v-model="todo.completed">
        <span>{{ todo.content }}</span>
    </li>
    <ul> 
    <!-- 기존 코드 -->
        <li v-for="todo in todos" v-if="!todo.completed" 
            v-on:click="check(todo)">
			{{ todo.content }}
		</li>
		<li v-else v-on:click="check(todo)">[완료!]</li>
    </ul>
</div>
```



#### 5) 취소선 구현

- `head` > `style` 태그 내 `completed` 클래스에 대한 스타일 적용가능
- `v-bind`를 사용 (`v-bind` 는 동적으로 하나 이상의 컴포넌트 속성 또는 표현식에 바인딩함)
- 속성 값인 `""` 에는 자바스크립트 코드가 들어갈 수 있으므로, 삼항 연산자를 사용하여 조건문을 구현 할 수 있음.
  - 삼항연산자 - `?` 앞에는 조건식을 적음. 조건 만족 시, `?`  바로 뒷 값 적용. else 부분은 `:`  뒤에 작성
  - 이를 통해 `todo.completed` 값이 `true` 인 데이터에 대하여 스타일을 적용 시 킬 수 있음

```html
<head    
	<style>
        .completed {
            text-decoration: line-through;
            opacity: 0.6;
        }
    </style>
</head>
<div id="app">
    <li v-for="todo in todos">
        <input type="checkbox" v-model="todo.completed">
        <span v-bind:class="todo.completed ? 'completed' : ''">
            {{ todo.content }}
        </span>
    </li>
```



#### 6) 취소선 구현 (2)

삼항 연산자는 단 하나의 조건만을 적용시킬 수 있다는 단점이 있음. 그러나 객체(오브젝트) 형태로 구현하면 여러 조건을 적용시킬 수 있음.

- 오브젝트 형태로  value에는 조건식을, key에는 조건에 대한 결과를 적음

```html
<li v-for="todo in todos">
    <input type="checkbox" v-model="todo.completed">
    <span v-bind:class="{completed:todo.completed}"> {{ todo. completed }}</span>
</li>
```



#### cf) 객체를 통한 스타일 속성 정의 

- 동적으로 스타일을 입히기 위해서 사용함
- font-size 처럼 속성에 `-` 이 들어간 경우, JS 표현식으로 입력할 때는 camelCase로 표현함 (=> fontSize)
- 새로운 Todo를 입력하는 input 태그에 color 속성 값(예; red)를 입력하면 스타일이 변경됨을 알 수 있음.

```html
<div v-bind:style="{color:newTodo, fontSize:'30px'}">
    <span> Red Text, 30 px </span>

</div>
```



#### 7) 셀렉트 박스 활용

셀렉트 박스를 통해 "완료된 리스트 / 완료되지 않은 리스트 / 모든 리스트"를 선별적으로 볼 수 있게 코드를 구현해보도록 하자.

- Vue-data 내 셀렉스 박스 내 목록으로 사용할 status를 추가 (status에 따라 다른 리스트를 보여주도록 구현)
- methods 내 셀렉트 박스 내 선택된 항목에 따라 다른 리스트를 보여주기 위한 함수 `todosByStatus`를 정의
  - `this.status` 가 `active` 인 경우, `filter`를 이용하여 `todo.completed` 가 `false` 즉,  완료되지 않은 리스트를 반환함.
  - 반대로, `this.status`가 `completed`인 경우, 완료된 리스트를 반환

```javascript
data: {
    status: 'all'
    // 중략
}
methods: {
    // 중략
    todosByStatus: function(){
        
        // 아직 완료되지 않은 리스트
        if (this.status === 'active'){
            return this.todos.filter((todo)=>{
                return !todo.completed
            })
        }
        
        // 완료된 리스트
        if (this.status == 'completed'){
            return this.todos.filter((todo)=>{
                return todo.completed
            })
        }
        
        // 전체리스트
        return this.todos
    },
}
```



- status와 바인딩할 또다른 input이 필요함. 이때 셀렉트 박스를 사용
- 반복문은 todos에서 `todosByStatus()`로 변경. `todosByStatus` 의 마지막 return이 전체 리스트인 `this.todos`를 나타내므로 변경해도 달라지는 부분은 없음.
- 셀렉트 박스를 통해 선택된 value에 따라 methods에 정의된 `todosByStatus` 의 조건문이 적용되며, 그에 따라 반복문이 선별적으로 돌면서 사용자는 다른 리스트를 볼 수있게 됨.

```html
<div id="app">
    <select v-model="status">
        <option value="all"> all </option>
        <option value="active"> active </option>
        <option value="completed">completed</option>
    </select>
    <ul>
        <li v-for="todo in todosByStatus()">
            <input type="checkbox" v-model="todo.completed">
```



#### 8) `v-bind: key=""`

지금까지 구현한 코드를 기반으로 체크박스에 체크를 누를 경우, 선택된 목록이 셀렉트박스 및 todosByStatus 함수에 따라 이동이 되나, 체크가 그대로 남아있는 등, 의도치 않는 동작이 발생함을 알 수 있음. 

4개의 todo가 있는 경우, 반복문을 통해서 4개의 li 태그가 생기는데, 나중에 어떤 사용자가 내용을 추가하거나 변경하는 경우 vue는 새롭게 내부적으로 브라우정 렌더링을 해야겠구나 것을 앎. 이때 vue는 key값을 기반으로 인지를 함. key는 DB의 기본키처럼 고유의 인덱스 값을 나타내는 역할을 함. key를 기준으로 어떤 위치의 값을 변경하고 삭제하는 등의 동작을 수행함.

따라서, `v-for`를 통해 반복문을 돌리는데, 반복문 내 개별적인 데이터에 key라는 속성을 부여해야함. 이때 사용하는 것이 `v-bind:key=""` 임. key값을 주지 않으면 vue는 혼란이 옴. 의도치 않은 동작이 생기는 이유는 고유한 인덱스 값인 key 값을 부여하지 않아서 발생하는 것이므로, key 값을 부여하도록 하자.

- todos 안 todo들에게 id값을 부여 
- 새로운 todo를 추가하는 함수인 `addTodo`에서 id값을 부여할 때는, 중복되지 않은 아이디 값을 적용할 때는 시간 개념을 활용하자 `Data.now()`

```html
<li v-for="todo in todosByStatus()" v-bind:key="todo.id">
    <input type="checkbox" v-model="todo.completed">
```

```javascript
data: {
    	status:'all',
        newTodo: '',
        todos: [
            {
            //추가된 부분//
                id: 1,
                content: '점심 메뉴 고민하기',
                completed: true,
            },
            
methods: {

    addTodo: function(){
        this.todos.push({
            //추가된 부분//
            id: Date.now(),
            content: this.newTodo,
            completed: false,
        })
        this.newTodo = ""
    },
```

