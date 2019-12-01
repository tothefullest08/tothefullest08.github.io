---
layout: post
title: 생활 코딩 PHP 05 - 함수, 배열
category: PHP
tags: [생활코딩, PHP]
comments: true
---



## 1. 함수

#### 1-1. 기본 문법

- 함수의 형식

```php
function 함수명( [인자...[,인자]] ){
   코드
   return 반환값;
}
```

- 함수의 정의와 호출

```php
<?php
function numbering(){
    $i = 0;
    while ($i < 10){
        echo $i;
        $i += 1;
    }
}
numbering();
?>
```



#### 1-2. 입력과 출력

- `return`: 출력
- 함수의 인자로 입력값을 받을 수 있으며, 복수의 인자도 입력 가능

```php
<?php
function get_arguments($arg1, $arg2){
    $arg3 = $arg1 + $arg2;
    return $arg3;
}
echo get_arguments(10, 20);
?>
```



#### 1-3. 인자의 기본값

- 키워드 인자로 인자의 기본값 설정 가능

```php
<?php
function get_arguments($args1=100){
    return $args1;
}
echo get_arguments(15);
echo get_arguments();
?>
```



## 2. 배열

#### 2-1. 배열의 생성

```php
<?php
$member = ['harry', 'potter', 'ron'];
$member2 = array('harry', 'potter', 'ron'); #php 5.4 이전 버전
echo $member[0];
echo $member[1];
echo $member2[2];
?>

<?php
function get_members(){
    return ["harry", "ron", "sally"];
}
$tmp = get_members();
echo $tmp[1];
echo get_members()[0]; #php 5.4 이상 버전에서 사용 가능
?>
```



#### 2-2. 배열의 사용(반복문)

- `count()` : 배열의 길이 반환
- `ucfirst()` : string의 첫번째 문자를 대문자로 변경

```php
<?php
function get_members(){
    return ['harry', 'ron', 'micheal', 'tom'];
}
$members = get_members();

for ($i=0; $i < count($members); $i++){
    echo ucfirst($members[$i])."<br />";
}
?>
```



#### 2-3. 배열의 제어 - 삽입

- `count`: 배열의 크기 확인
- `array_push(배열명, 데이터)` : 배열의 끝에 1개의 데이터 추가
- `array_merge(배열명, 복수의데이터)` : 배열의 끝에 여러개의 데이터(배열) 추가. 이때 변수의 값으로 덮어씌어줄 것
- `array_unshift` : 배열의 제일 앞에 데이터 추가
- `array_splice()` : 배열 자르기 및 중간에 추가

```php
<?php
$tmp = [1,2,3,4,5];
echo count($tmp);
?>
<br />

<?php
$li = ['a', 'b', 'c', 'd', 'e'];
array_push($li, 'f'); # $li 배열의 끝에 'f' 추가
var_dump($li);
?>
<br />

<?php
$li2 = ['f', 'g', 'h', 'i'];
$li2 = array_merge($li2, ['j', 'k']); # 여러개의 데이터 추가
var_dump($li2);
?>
<br />

<?php
$li3 = ['l','m','n'];
array_unshift($li3, 'z'); # 배열의 제일 앞에 추가
var_dump($li3);
?>
<br />

<?php
$li = ['a','b','c','d','e'];
array_splice($li, 2, 0, 'B'); #  2번째 인덱스 뒤에 'B' 삽입
var_dump($li);
?>
```



#### 2-4. 배열의 제어 - 제거

- `array_pop` : 배열의 제일 끝 데이터 제거
- `array_shift` : 배열의 첫 데이터 제거

```php
<?php
$li = ['a','b','c','d','e'];
array_pop($li);
var_dump($li); # e 제거
array_shift($li); 
var_dump($li); # a 제거
?>
```



#### 2-5. 배열의 제어 - 정렬

- `sort` : 정렬
- `rsort` : 역순으로 정렬

```php
<?php
$li = ['c', 'a', 'b', 'z' , 'f'];
sort($li); #정렬
var_dump($li);
rsort($li); #역순으로 정렬
var_dump($li);
?>
```



#### 3. 연관배열(Aassociative Array)

- 배열의 식별자(인덱스)로 숫자가 아니라 문자를 사용(Python의 Dictionary와 매우 유사한 자료 구조)
- `array()` 와  `=>`를 조합하여 생성. 혹은 Python Dict처럼  key, value값을 입력 가능
- 열거 할 경우, `foreach` 사용
  - `for`은 숫자를 기반으로 사용 가능. 문자기반인 연관배열에서는 `foreach` 사용

```php
<?php
$grades = array('harry'=>10, 'potter'=>6, 'Ron'=>80);
var_dump($grades);

$grades1 = [];
$grades1['harry'] = 10;
$grades1['potter'] = 6;
$grades1['Ron'] = 5;
var_dump($grades1);
echo $grades1['Ron'];
?>
<br />

<?php
$grades2 = array('harry'=>10, 'potter'=>5, 'malpoe'=>20);
var_dump($grades2);
foreach($grades2 as $key => $value){
    echo "key:{$key} value: {$value}<br />";
}
?>
```
