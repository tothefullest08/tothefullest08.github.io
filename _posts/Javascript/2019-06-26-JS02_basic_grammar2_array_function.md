---
layout: post
title: JavaScript 02 - 기본문법 (2) / 배열, 함수
category: Javascript
tags: [Javascript]
comments: true
---



# JavaScript 기본 문법 (2) - 배열, 함수

## 5. 배열

#### 1) length / reverse / push

- .length : 배열의 길이 
- .reverse() : 배열 뒤집기
- .push(data) : 배열의 끝에 데이터 추가, 배열의 길이를 반환함

```javascript
const numbers = [1,2,3,4]

>console.log(numbers[0])
1

> const numbers = [1,2,3,4]
undefined
> numbers
[ 1, 2, 3, 4 ]
> numbers.length
4
> numbers.reverse()
[ 4, 3, 2, 1 ]
> numbers
[ 4, 3, 2, 1 ]
> numbers.reverse()
[ 1, 2, 3, 4 ]
> numbers.push('a')
5
> numbers
[ 1, 2, 3, 4, 'a' ]
> numbers.push(5)
6
> numbers
[ 1, 2, 3, 4, 'a', 5 ]
```



#### 2) pop / unshift / shift 

- .pop() : 배열의 오른쪽 끝 값 제거. 배열의 길이를 반환함.
- .unshift() : 배열의 왼쪽에 값 추가
- .shift() : 배열의 왼쪽 값 제거. 제거되는 값을 반환함.

```javascript
> numbers.pop()
5
> numbers.pop()
'a'
> numbers.unshift('a')
5
> numbers
[ 'a', 1, 2, 3, 4 ]
> numbers.shift()
'a'
> numbers
[ 1, 2, 3, 4 ]
```



#### 3) includes / indexOf

- .includes(data): 배열에 data가 들어있으면 true 반환, 없으면 false 반환
- .indexOf(data): 배열에 data가 들어있으면 왼쪽에서 부터 인덱스 번지를 반환, 없으면 -1 반환

```javascript
> numbers.includes(1)
true
> numbers.includes(0)
false
> numbers.indexOf('a')
-1
> numbers.push('a')
5
> numbers.push('a')
6
> numbers.indexOf('a')
4
```



#### 4) join / slice

* join(형식): 데이터를 형식으로 구분해서 나타냄
* slice(2,4): 2번부터 4번 전까지 인덱스에 해당하는 값 출력
* slice(2): 인덱스 2번부터 값 출력
* slice(-2): 뒤에서부터 두번째부터 출력

```javascript
> numbers.join()
'1,2,3,4,a,a'
> numbers.join('')
'1234aa'
> numbers.join('-')
'1-2-3-4-a-a'
> numbers.join()
'1,2,3,4,a,a'
> numbers.join('')
'1234aa'
> numbers.join('-')
'1-2-3-4-a-a'
> numbers.slice(2,4)
[ 3, 4 ]
> numbers.slice(2)
[ 3, 4, 'a', 'a' ]
> numbers.slice(-3)
[ 4, 'a', 'a' ]
> numbers.slice(-2)
[ 'a', 'a' ]
```



#### 5) Object

- 원시타입(primitive)를 제외한 나머지 값들(함수, 배열, 정규표현식 등)을 일컫으며, 객체는 key와 value로 구성된 Property들의 집합임.
- 앞에 붙는 key값에 쌍따옴표를 해주지 않아도 됨.
  - object를 사용할 때는 []안에 키를 넣어 value를 불러올때 따옴표를 붙여줘야 함.
- 중간에 띄어쓰기가 있다면 따옴표가 필요.
- 오프젝트의 value로 배열, object 뭐든지 올 수 있음.
- 오브젝트 사용시 `me.name` 과 같이 띄어쓰기가 없다면 `.` 으로 key를 불러 사용할 수 있음.

```javascript
const me = {
... name: 'harry',
... 'phone number': '01012345678',
... appleProduct :{
..... ipad: true,
..... iphone: 'X'
..... }
... }

> me['name']
'harry'
> me.name
'harry'
> me.appleProduct
{ ipad: true, iphone: 'X' }
```

- 아래와 같이 선언한 변수도 Object안에서 사용 가능하다.

```javascript
const food = {
    fruits : ['apple', 'grapes'],
    vegetables : ['onion', 'lettuce']
};

const server = null;

const restaurant = {
    food,
    server
}

console.log(restaurant)
```

- method
  - 파이썬과 달리 별도의 문법이 잇는 것이 아니라 오브젝트의 value에 함수를 할당함
  - 앞서 변수/object와 같이 함수도 ES6+부터 아래와 같이 선언도 가능하다.
  - 메서드 정의시, `arrow function`은 사용하지 않음.

```javascript
const me2 = {
    name : 'LEE',
    greeting: function(message) {
        return `${this.name} : ${message}`
    }
}

console.log(me2.greeting('hi')) // LEE : hi
me2.name = 'PARK'
console.log(me2.greeting('hello')) // PARK : hello
```

```javascript
const greeting = function(message) {
    return `${this.name} : ${message}`
}

const you = {
    name : 'you',
    greeting
}

console.log(you.greeting('hi'))
you.name = "KIM"
console.log(you.greeting('hello'))
```





#### 6) JSON - JavaScript Object Notation (JS 객체 표기법)

* 어떠한 요청을 보낼때는 문자열 타입으로 보내고 받는 쪽은 문자열을 object형태로 변환하는 과정이 웹에서 발생한다.
* JSON은 `stringify()` 와 `parse()` 를 이용하여  object와 string으로 변환이 가능하다.

```javascript
JSON.stringify() // Object -> JSON String
JSON.parse() // JSON String -> Object
```

```javascript
> const JsonData = JSON.stringify(me)
undefined
> JsonData
'{"name":"harry","phone number":"01012345678","appleProduct":{"ipad":true,"iphone":"X"}}'
> typeof JsonData
'string'
> const parseData = JSON.parse(JsonData)
> parseData
{ name: 'harry',
  'phone number': '01012345678',
  appleProduct: { ipad: true, iphone: 'X' } }
> typeof parseData
'object'
```



## 6. 함수

#### 1) 함수 선언식

```javascript
function add(num1, num2){
    return num1 + num2
}

console.log('add: ' + add(1,2))
```



#### 2) 함수 표현식

```javascript
const sub = function(num1, num2){
    return num1 - num2
}

console.log('add: ' + sub(5,3))
```



#### 3) Arrow Function(ES6 +)

- ES6+ 부터 Arrow Function을 사용하면 함수를 더욱 간소화 시킬 수 있음.

```javascript
// 기존방법
const mul = function(num1, num2) {
    return num1 * num2
}

//Arrow: function을 제거하고, => 를 추가함.
const mul = (num1, num2) => {
    return num1* num2
}

let square = (num) => {
    return num ** 2
}

// return문 단 한줄이면 {} & return 생략 가능
square = (num) => num ** 2

// ()안의 인자가 하나뿐이면 () 생략 가능. 인자가 없는 경우는 생략 불가
// 0개인 경우는 아래와 같이 입력 요
square = num => num ** 2
let noArgs = () => 'No args'

// object를 return하는 경우
// 괄호가 없으면 {}를 함수의 {}로 인식하기 때문에 ()가 필요!
let returnObject = () => ({key:'value'})
```



- 함수의 기본 인자 설정 가능

```javascript
const sayHello = (name='noName') => `hi${name}`

console.log(sayHello('John'))
console.log(sayHello())

=> hiJohn
=> hinoName
```



- 익명 함수; 이름이 없는 함수를 익명함수 라고 함.

```javascript
function (num) {return num ** 3} // 세제곱
(num) => {return num ** 0.5} // 제곱근

// 익명 함수 즉시 호출
(function (num) {return num**3})(3) // 3의 3제곱근
```


