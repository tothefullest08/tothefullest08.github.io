---
layout: post
title: 생활 코딩 PHP 02 - 문법 - 숫자/문자/변수/상수/비교 
category: PHP
tags: [생활코딩, PHP]
comments: true
---



## 1. 숫자와 문자

#### 1-1. 숫자

```php
<?php
echo 4/2; # echo: 출력
?>

<?php
var_dump(6.1); # var_dump: 데이터 타입 확인 => float(6.1)
?>
```



#### 1-2. 문자

- 따옴표로 묶어줄 것. 따옴표를 사용하지 않을 경우 php에서 상수라는 다른 데이터 타입으로 인식하며 에러가 발생함

> **Parse error**: syntax error, unexpected 'world' (T_STRING), expecting ',' or ';' in **C:\Bitnami\wampstack-7.3.11-0\apache2\htdocs\number_string\string.php** on line **2**

- 문자 결합 시에는 `.` 사용

```php
<?php
echo 'hello world';
?>

<?php
echo var_dump(1234);
echo var_dump("1234");
?>

<!-- 문자 결합 연산자 . -->
<?php
echo "hello"." "."world";
?>

<!-- 따옴표 안에서 따옴표(인용부호) 사용 -->
<!-- 1. 작은 따옴표 사용 -->
<?php
echo '그는 "안녕하세요" 라고 말했다';
?>
<!-- 2. escaping - 역슬래쉬(\) 사용 -->
<?php
echo "그는 \"안녕하세요?\"라고 말했다";
?>
```



## 2. 변수와 상수

#### 2-1. 변수 

- 변수(variable): 변하지 않는 값.
- 변수 선언 시 `$` 를 사용
- `echo` 대신 `print` 사용 가능

```php
<html>
<body>
<?php
$a=1;
echo $a+1; #2
echo "<br />";
$a=2;
print $a+1; #3  print == echo
?>
</body>
</html>
```

```php
<html>
<body>
<?php
$first = "coding";
echo $first." everybody";
?>
</body>
</html>
```



#### 2-2. 변수 사용 이유?

변수를 통해 코드의 재활용성 향상시킬 수 있음.  따라서 코드를 효율적으로 작성할 수 있음.

```php
<html>
<body>
<?php
echo 100+10;
echo '<br />';
echo (100+10)/10;
echo '<br />';
echo ((100+10)/10)-10;
echo '<br />';
echo (((100+10)/10)-10)*10;
echo '<br />';
?>
</body>
</html>
```

```php
<html>
<body>
<?php
$a = 100;
$a = $a + 10;
print $a.'<br />';
$a = $a / 10;
print $a.'<br />';
$a = $a - 10;
print $a.'<br />';
$a = $a * 10 ;    
print $a.'<br />';
?>
</body>
</html>
```



#### 2-3. 상수

- 상수(constant): 변하지 않는 값
- 상수 선언시 `define` 사용. 첫번째 인자로 상수, 두번째 인자로 상수값 입력
- 선언된 상수를 재선언할 경우 에러 발생

> **Notice**: Constant TITLE already defined in **C:\Bitnami\wampstack-7.3.11-0\apache2\htdocs\variable\variable8.php** on line **6**

```php
<html>
<body>
<?php
define('TITLE', 'PHP Tutorial');
echo TITLE;
// define('TITLE', 'JAVA Tutorial');
?>
</body>
</html>
```



#### 2-4. 변수의 데이터 타입 검사 및 변경

- `gettype` : 데이터 타입 검사
- `settype`: 데이터 타입 변경
- `var_dump`는 데이터 타입 검사 및 출력을 강제하므로 활용도가 낮음

```php
<html>
<body>
<?php
$a = 100;
echo var_dump($a); #int(100)
echo gettype($a);  #integer
settype($a, 'double'); #double: 실수를 의미
echo '<br />';
echo gettype($a); #double
?>
</body>
</html>
```

#### 2-5. 가변변수

가변변수(variable variables).  변수의 이름을 가변적으로 변수로 변경 할 수 있게 함.

```php
<html>
<body>
<?php
$title = 'subject';
$$title = 'PHP tutorial';
echo $title;
echo $subject;
?>
</body>
</html>
```



## 3. 비교

-  `==`  ,  `!=`  ,  `===` , `>` , `>=` 등이 있음.
- `===` : 데이터 뿐만 아니라 데이터 타입까지 비교할 경우 사용

```php
<html>
<body>
<?php
echo('<br />');
echo "1==2 : ";
var_dump(1==2);
echo('<br />');
echo "1==1 : ";
var_dump(1==1);
echo('<br />');
echo '"one"=="two" : ';
var_dump("one"=="two");
echo('<br />');
echo '"two"=="two" : ';
var_dump("two"=="two");
echo('<br />');
?>
</body>
</html>
```

```html
<!--결과 -->
1==2 : bool(false)
1==1 : bool(true)
"one"=="two" : bool(false)
"two"=="two" : bool(true)
```

```php
<html>
<body>
<?php
var_dump(10>=20); #false
var_dump(1=='1'); #true
var_dump(1==='1'); #false
?>
</html>
```

