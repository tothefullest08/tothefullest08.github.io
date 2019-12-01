---
layout: post
title: 생활 코딩 PHP 04 - 조건문, 반복문
category: PHP
tags: [생활코딩, PHP]
comments: true
---



## 1. 조건문

#### 1-1 기본 문법

- `if`  // `else` // `else if` 

```php
<?php
if (false){
    echo 1;
} else if (true){
    echo 2;
} else if (true){
    echo 3;
} else {
    echo 4;
}
?>
```



#### 1-2. 조건문 응용

- HTML 폼 태그 & 비교 연산자 & 조건문등을 응용한 유사 로그인 어플리케이션 구현

```html
<html>
<body>
    <form method="get" action="2.php">
        <input type="text" name="id" />
        <input type="submit" />
    </form>
</body>
</html>
```

```php
<?php
if ($_GET['id'] == 'harry'){
    echo 'right';
} else {
    echo 'wrong';
}
?>
```

- POST방식으로 변경 & 비밀번호 INPUT 태그 추가 

```html
<html>
<body>
    <form method="post" action="3.php">
        id :  <input type="text" name="id" />
        password : <input type="text" name="password" />
        <input type="submit" />
    </form>
</body>
</html>
```

```php
<?php
if ($_POST['id'] == 'harry') {
    if ($_POST['password'] == '111111') {
        echo 'correct';
    } else {
        echo 'password wrong';
    }
} else {
    echo 'id wrong';
}
?>
```



#### 1-3 논리연산자

- `and(&&)` // `or` //  `!` 
- boolean 값은 `!` 을 앞에 입력함으로써 값을 역전 시킬 수  있음. ex) `!true` 
- `1` : `true`  // `0` : `false` 

```php
<?php
if ($_POST['id'] == 'harry' && $_POST['password'] === '111111') {
    echo 'welcome harry';
} else {
    echo 'login failed';
}
?>
```



## 2. 반복문

#### 2-1. 기본 문법

- `while` 

```php
<?php
while (true) {
    echo 'coding everybody';
}
?>
```

```php
<?php
$i = 0;
while($i < 5) {
    echo '<br />';
    echo 'coding everybody';
    $i += 1;
    echo '현재 변수 i 값 '.$i;
}
?>
```

- `for`

```php
for(초기화; 반복 지속 여부; 반복 실행){
    코드;
}
```

```php
<html> 
<body> 
<?php
for ($i = 0; $i < 10; $i++){
    echo 'coding everybody'.$i.'<br />';
}
?>
</body>
</html>
```



#### 2-2. 반복문의 제어

- `break` : 반복 작업 중 중단을 시킬 때 사용
- `continue` : 실행은 중단 시키면서 반복이 지속돼게 할 때 사용

```php
<?php
for ($i = 0; $i < 10; $i++){
    if ($i == 5){
        break;
    }
    echo "coding everybody{$i}<br />";
}
?>
```

```php
<?php
for ($i = 1; $i < 10; $i++){
    if ($i == 5){
        continue;
    }
    echo "coding everybody{$i}<br />";
}
?>
```



#### 2-3. 반복문의 중첩

```php
<?php
for ($i = 0; $i < 4; $i++){
    for ($j = 0; $j < 4; $j++){
        echo "coding everybody {$i}{$j} <br />";
    }
}
?>
```
