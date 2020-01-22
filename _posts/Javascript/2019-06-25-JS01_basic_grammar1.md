---
layout: post
title: JavaScript 01 - 기본문법 (1) / HTML 조작, 변수, 자료형, 조건 & 반복
category: Javascript
tags: [Javascript]
comments: true
---



# JavaScript 기본 문법 (1)

## 1. JavaScript로 HTML 조작

#### 1) 기본 작성 위치

- JS는 html에서 사용된 제목 본문을 수정해야 하기 때문에 html로 작성된 구문 밑에서 작성됨. 

```html
  <!DOCTYPE html>
  <html lang="en">
  <head>
      <meta charset="UTF-8">
      <meta name="viewport" content="width=device-width, initial-scale=1.0">
      <meta http-equiv="X-UA-Compatible" content="ie=edge">
      <title>Document</title>
      <!-- JS 위치 -->
  </head>
  <body>
      
      <!-- JS 위치 -->
      <script>
          
      </script>
  </body>
  </html>
```

  

#### 2) 주석달기

- 한줄에 주석을 달때는 `//` 을 사용하고, 여러줄을 쓸때는 `/*`  `*/` 를 사용함.

```javascript
/*
This is Comment
*/

// This is Comment
```



#### 3) 문서조작

- JS로 문서 조작
  - 문서에 html태그가 적용되어서 글이 써짐

```javascript
//문서에 출력하기
document.write('<h1>Hello, World!</h1>')
```



- 브라우저 상에서 JS로 웹 문서 조작
  - 개발자 도구 -> Console 창에서 입력해서 태그를 선택하고 태그의 글을 수정할 수 있음.

```javascript
document.querySelector('h1')
<h1>​Hello, World!​</h1>​
document.querySelector('h1').innerText
"Hello, World!"
document.querySelector('h1').innerText = 'Bye, World!'
"Bye, World!"
```



## 2. 변수

#### 1) var, let, const

- 파이썬과 달리 javascript에서는 기본적으로 변수를 선언해야함.
- ES6 이전에는 `var`을 이용하였으나 이후에는 `let` 과 `const`를 사용함.
- `let`은 변수를 선언하는 keyword로  재할당 가능함.
- `const`는 상수를 선언하는 keyword로 재할당 불가능함.
  - console에서 재할당 하면 `Uncaught TypeError: Assignment to constant variable.` 에러 발생

```javascript
// let: 재할당 가능
let word1 = '하이'
document.write(word) 

// const: 재할당 불가능
const word2 = '안녕?'
document.write(word2)
const word2 = '안녕2?' // Uncaught TypeError: Assignment to constant variable 에러발생
```


#### 2) `var` vs `let`, `const`

- `var`은 function 스코프이며, `let` 과 `const`는 block 스코프이다.

```javascript
// let: 블록 스코프. {} 내에서만 선언된 변수를 사용할 수 있음. 
for (let j=0; j<1; j++){
    console.log(j) // 0 
}
// console.log(j) // Uncaught TypeError: Assignment to constant variable 에러발생

// var: 함수 스코프. 함수 내에서 선언된 변수를 사용할 수 있음. 
// 따라서 반복문 블록{} 밖에서도 함수 내라면 선언된 함수를 사용할 수 있음.
for (var i=0; i<1; i++){
    console.log(i)
}
console.log(i)

const myFunction = function() {
    for (var k=0; k<1; k++){
        console.log(k)
    }
    console.log(k)
}
myFunction() 
// 0 
// 1

console.log(k) // ReferenceError: k is not defined
```

- 만약 변수 선언 키워드를 사용하지 않으면, 함수/블록 안에서 선언되어더라도 무조건 전역변수로 취급된다.

```javascript
// 변수 선언 키워드 사용 X -> 무조건 전역변수로 취급됨
function myFunction1(){
    for (p=0; p<1; p++){
        console.log(p)
    }
    console.log(p)
}
myFunction1()
console.log(p) // 1
```



#### 3) 문자열과 포맷팅

- 아래의 형태로 formatting을 활용하여 입력할 수 있음.
- `(backtick) 통해서 문자열을 안에 넣어서 작성할 수 있다.

```javascript
const firstName = 'Harry'
const lastName = 'Potter'
const fullName = firstName + lastName
document.write('<h1>' + fullName + '!!' + '</h1>')
document.write(`<h1>${fullName}!!</h1>`)

// console에 출력하기
console.log(`Console: ${fullName}!!`)
```



#### 4) 변수 네이밍 

- JavaScript는 userName 과 같은 띄워쓰기를 대문자로 구문하는 `CamelCase`를 사용함
  - python은 under bar (_) 로 변수명을 구분지음. 예) user_name (`snake_case`)



#### 5) 사용자 입력받기

- `prompt` 를 사용하면 브라우저 실행 시 팝업 입력창을 띄울 수 있음. 이를 활용하여, 브라우저에 변수의 값을 나타낼 수 있음.

```html
<body>
    <script>
        const userName = prompt('Hello! Who are you?')
        let message = `<h1>hello ${userName}</h1>`
        document.write(message)
    </script>
</body>
```



## 3. 자료형

#### 1) 기본 자료형 (primitve)

- Boolean (true/false)
- null - 어떤 값이 의도적으로 비어있음을 표현함.
- undefined - 값을 할당하지 않은 변수를 undefined로 나타냄.
- number(`Infinty`, 양수, 음수 등)
- string - 텍스트 데이터를 나타낼 때 사용. immutable한 속성을 갖고 있음.

#### 2) object

- key와 value가 있는 형태로, key를 통해 value에 접근이 가능함. JSON과 유사한 형태임.



## 4. 조건/반복

#### 1) 조건문

- if / else if  / else 로 사용하며 기본적인 구조는 아래의 예시를 참조하자.
- 또한, 위에서 배운 내용을 활용하여 입력받은 변수로 조건문 분기를 사용할 수 도 있다.

```javascript
const userName = prompt('Hello! Who are you?')
let message = ''

// if문
if (userName === 'admin') {
    message = '<h1>This is secret Admin page</h1>'
}
else if (userName === 'HarryLee'){
    message = '<h1> I am inevitable </h1>'
}
else {
    message = `<h1>Hello ${userName}</h1>`
}
```



- `==`  과 `===`  비교
  - `==` 는 동등 연산자로, 피연산자가 서로 다른 타입일 경우, 타입을 강제로 형변환하여 비교함.
  - `===` 는 일치 연산자로, 두 피연산자를 더 정확하게 비교함.

```javascript
// == vs. ===
// == : '값' 만 비교 (ex. 0=="0"는 true) 타입은 다르지만 값은 동일해서 true
// === : '값 & '타입' 비교 (ex. 0 === "0"는 false. 한쪽은 integer 다른 쪽은 string이기 때문) 

// 동등 연산자(==) 사용
> 1 == '1'
true
> 1 == '1 '
true
> 1 == true
true
> undefined == null
true

> null == true
false
> 'true' == true
false
> true == 2
false

// 일치 연산자(===) 사용
> 1 === '1'
false
> true === 1
false
> undefined === null
false
```

<img src="/assets/javascript/equal.png"/>

![](img/equal.png)



#### 2) 삼항 연산자

- 조건문이 참이면 콜론(:)앞의 구문실행하고 거짓이면 콜론(:)뒤의 구문을 실행함.

```javascript
const number = 10
number === 10 ? console.log('number === 10') : console.log('number !== 10')
```



#### cf) 세미콜론 테스트

- javascript는 세미콜론의 사용은 자유로움.  에러가 발생 할 수 있는 구문에 세미콜론을 사용할 것.
- 세미콜론을 찍는 규칙은 C 언어와 동일함. 

```javascript
const a = 1
const b = 2
const c = a + b
(a+b).toString()

구문이
const a = 1
const b = 2
const c = a+ b(a+b).toString() 이렇게 연결되서 인식 될 수 있다.
```



#### 3) 반복문

- 방법 1, 2는 let 변수를 사용해서 연산자로 변수의 값을 수정해서 반복문을 돌리고
- 방법 3을 const로 선언할 경우, 반복문이 돌면서, 기존에 선언된 값은 지워지고, number에 새로운 값으로 선언을 해서 반복문을 돌려짐.

```javascript
// 방법 1 
let i = 0
while (i<10){
    console.log(i)
    i++
}
// 방법 2
for(let j=0; j<10;j++){
    console.log(j)
}
// 방법 3
for (let number of [1,2,3,4,5]) { // const로도 선언 가능
    console.log(number)
}
```
