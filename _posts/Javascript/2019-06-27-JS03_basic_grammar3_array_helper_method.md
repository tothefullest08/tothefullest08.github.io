---
layout: post
title: JavaScript 03 - 배열 조작 메서드(forEach, map, filter, find, every, some, reduce)
category: Javascript
tags: [Javascript]
comments: true
---



# JavaScript 기본 문법 (3)

## 7. Array Helper Methods - 배열 조작 메서드

#### 1) `forEach`

- ES5 표준화 버전에서는 반복문을 돌릴때 `var` 메서드를 사용했었음

```javascript
var colors = ['red', 'blue', 'green']
for (var i=0; i<colors.length, i++){
    console.log(colors[i]);
}
```

- ES6+ 에는 `forEach` 메서드 사용
  - 가장 기본적인 형태의 반복문으로, 단순 반복을하고 return 값을 따로 가지지 않음.
  - 함수 자체가 파라미터로 넘어가서 동작하는 것을 callback 함수라고 하며 JS에서 가장 대표적으로 사용하는 방법임. callback 함수로서 동작하게 코드를 아래와 같이 작성 할 수 도 있다.

```javascript
const colors = ['red', 'blue', 'green']

// 기본적인 방법
const f = function(color){
    console.log(color)
}
colors.forEach(f)

// callback 함수로서 동작
colors.forEach(function(color){
    console.log(color)
})
```



#### 2) `map`

- ES5

```javascript
var numbers = [1,2,3]
var doubleNumbers = []

for (var i=0; i<numbers.length; i++){
    doubleNumber.push(numbers[i]*2)
}

console.log(doubleNumbers)
```

- ES6+ 에는 `map` 메서드 사용
  - return 값을 가지며 새로운 배열을 만듦.

```javascript
const numbers2 = [1,2,3]
const doubleNumbers2 = numbers2.map(function(number2){
    return number2 * 2
})
console.log(doubleNumber2)
```



#### 3) `filter`

- 원하는 값들롤만 새로운 배열을 만듦.
- 해당 조건이 true인 값에 대해서만 가져와 배열에 넣음

```javascript
const products = [
    {name: 'cucumber', type:'vege'},
    {name: 'banana', type: 'fruit'},
    {name: 'onion', type:'vege'},
    {name: 'apple', type: 'fruit'},
]

const fruitproducts = products.filter(function(product){
    return product.type == 'fruit'
})

console.log(fruitproducts)
```





#### 4) `find`

- 단 하나의 결과만을 출력함.
- 조건을 만족하는 값을 찾을 경우, 반복문을 바로 종료시킴.

```javascript
const users = [
    {name: 'HARRY'},
    {name: 'ADMIN'},
    {name: 'MANSU'},
]

const foundUser = users.find(function(user){
    return user.name == 'ADMIN'
})

console.log(foundUser)
```



#### 5) `every` & `some`

- `every`: 배열안의 모든 데이터가 조건을 만족하는 경우 true를 반환함. else, false를 반환.
- `some`:  배열안의 데이터 중 하나라도 조건을 만족하는 경우 true를 반환함. else, false를 반환.

```javascript
const computers = [
    {name: 'macbook', ram: 16},
    {name: 'gram', ram: 8},
    {name: 'series9', ram:32},
]

const everyComputersAavailable = computers.every(function(computer){
    return computer.ram > 16
})

const someComputersAvailable = computers.some(function(computer){
    return computer.ram > 16
})

console.log(everyComputerAvailable)
console.log(someComputerAvailable)
```



## 7-1. `reduce`

배열의 각 요소에 대해 주어진 reduce 함수를 실행시키고 하나의 결과 값을 반환함.

#### 1) 구문

```javascript
arr.reduce(callback[, initalValue])
```

#### 2) 매개변수

- `callback`
  - `accumulator`: 콜백의 반환값을 누적.  콜백의 이전 반환 값 또는 콜백의 첫번째 호출이면서 `initialValue`를 제공한 경우 `initalValue`의 값임.
  - `currentValue`: 배열 내 현재 처리되고 있는 요소
  - `currentIndex`(optional): 배열 내 현재 처리되고 있는 요소의 인덱스. `initalValue`를 제공한 경우 0,  아니면 1부터 시작함.
  - `array`(optional): `reduce()`를 호출한 배열
- `initialValue`(optional): `callback`의 첫호출 때 첫번째 인수에 제공하는 값. 초기값을 제공하지 않을 경우, 배열의 첫번째 요소를 사용함

`reduce`의 return 값에는 배열이 들어 갈 수도 있으며, 특정한 값도 들어갈 수 있음.

#### 3) `reduce()` 작동방식

- 1부터 5까지 있는 배열 내 숫자들의 합 계산하기

```javascript
var arr = [1,2,3,4,5]
var cnt = 0
var sum = arr.reduce(function(acc, value){
    cnt ++ 
    return acc + value
});

console.log(cnt) // 4
console.log(sum) // 15
```

콜백은 4번 호출되며, 각 호출의 인수와 반환값은 다음과 같음.

- `initialValue` 의 값을 따로 지정하지 않았기 때문에, 배열의 첫번째 요소(1)이 `initialValue` 으로 사용되었으며, 콜백의 누적값인 `accumulator`에 저장되어있음.
- 배열의 첫번째 요소(1)을 사용하였으므로 순서대로 `currentValue` 에는 2가 들어가고 첫번째 반환값은 3이된다. 같은 방식으로 콜백은 총 4번 호출되며 각 호출의 인수와 반환값은 아래와 같이 정리할 수 있음.
- `cnt` 를 통해 몇번 콜백이 되었는지 쉽게 알 수있음.

| call back | acc (accumulator) | value(currentValue) | 반환값(return) |
| --------- | ----------------- | ------------------- | -------------- |
| 1st 호출  | 1                 | 2                   | 3              |
| 2nd 호출  | 3                 | 3                   | 6              |
| 3rd 호출  | 6                 | 4                   | 10             |
| 4nd 호출  | 10                | 5                   | 15             |



- `initalValue` 값을 5로 지정했을 경우, 값이 아래와 같이 달라짐을 알 수 있다.
- 5라는 초기값이 새로 주어졌으므로 callback은 5번 호출되며, 최종 합은 20으로 변한다.

```javascript
var arr = [1,2,3,4,5]
var cnt = 0
var sum = arr.reduce(function(acc, value){
    cnt ++ 
    return acc + value
}, 5);

console.log(cnt) // 5
console.log(sum) // 20
```



#### 4)` reduce` vs `map`

- `reduce`
  - `initalValue` 으로 빈 배열을 우선 만들었음. 이에 따라, `value` 에는 순차적으로 1,2,3이 들어가고 콜백이 총 3번됨. 
  - 사이사이에 삽입한 console.log를 면밀히 살펴 보면 어떠한 프로세스로 진행되는지를 알 수 있음.

```javascript
var num = [1,2,3]
var count = 0
var doubleNum = num.reduce(function(acc, value){
    console.log(acc, value)
    acc.push(value*2)
    console.log(acc)
    count ++
    return acc
}, []);

console.log(count) // 3
console.log(doubleNum) // [ 2, 4, 6 ]
```



| call back | acc (accumulator) | value(currentValue) | acc.push(value*2) | 반환값(return) |
| --------- | ----------------- | ------------------- | ----------------- | -------------- |
| 1st 호출  | []                | 1                   | [2]               | 2              |
| 2nd 호출  | [2]               | 2                   | [2,4]             | 4              |
| 3rd 호출  | [2,4]             | 3                   | [2,4,6]           | 6              |



#### 5) `reduce` vs `filter`

- `reducer`
  - `map` 때와 동일한 방식으로, `initalValue` 으로 빈 배열을 우선 만듦. 그 후, 분기문에 따라, `type== 'vegetable'` 을 만족하는 값에 대해서만 배열에 추가한 후, 배열을 return 함.

```javascript
const products = [
    {name: 'banana', type:'fruit'},
    {name: 'onion', type:'vegetable'},
    {name: 'apple', type: 'fruit'},
    {name: 'lettuce', type: 'vegetable'},
]

var vegetableProducts = products.reduce(function(acc, value){
    if (value.type == 'vegetable') {
        acc.push(value)
    }
    return acc
}, [])
console.log(vegetableProducts)
```

- `filter`

```javascript
const products2 = [
    {name: 'banana', type:'fruit'},
    {name: 'onion', type:'vegetable'},
    {name: 'apple', type: 'fruit'},
    {name: 'lettuce', type: 'vegetable'},
]

const vegetableProducts2 = products2.filter(function(product){
    return product.type == 'vegetable'
})

console.log(vegetableProducts2)

```



#### 6) `reduce` vs `find`

- `reduce`
  - `find`의 속성은 원하는 값을 찾을 경우 반복문이 바로 끝나야함. 이 조건을 `reduce` 에서 걸어주기 위해 `type of acc == 'undefined'`  인 조건을 추가로 삽입하였음.

```javascript
const users = [
    {name:'HARRY'},
    {name:'ADMIN'},
    {name:'MANSU'}
]

const findUser = users.reduce(function(acc, value){
    if (typeof acc == 'undefined' && value.name == 'ADMIN'){
        acc = value;
    }
    return acc;
}, undefined)

console.log(findUser)
```

- `find`

```javascript
const users2 = [
    {name:'HARRY'},
    {name:'ADMIN'},
    {name:'MANSU'}
]

const findUser2 = users2.find(function(user){
    return user.name == 'ADMIN'
})

console.log(findUser2)
```



