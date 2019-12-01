---
layout: post
title: 생활 코딩 PHP 06 - include와 namesapce
category: PHP
tags: [생활코딩, PHP]
comments: true
---



## 1. include와 namespace

#### 1-1. include/require

다른 PHP 파일을 코드 안으로 불러와서 사용할 수 있게 함

`include` 와 `require`은 외부의 php 파일을 로드할 때 사용하는 명령.  두 명령어의 차이는 `include`는 warning을 일으키며, `require`은 fatal error를 일으킴. (`require`이 더 엄격한 기준).  `_once`는 파일을 로드할때 단 한번만 로드시킬때 사용

- `include`
- `include_once`
- `require`
- `require_once`

```php
# greeting.php
<?php
function welcome(){
    return 'Hello world';
}
?>
    
#2.php
<?php
include 'greeting.php';
echo welcome();
?>
    
```



#### 1-2. namespace

디렉토리와 같은 개념으로 하나의 어플리케이션 내에 여러개의 모듈을 사용하면서 같은 이름이 사용될 경우 충돌이 발생 할 수 있음. 이러한 경우 네임스페이스를 지정하여 충돌을 방지함.

아래의 경우 로드한 두개의 파일 모두 `welcome` 이라는 함수를 선언했기 때문에 중복 선언으로 인하여 에러가 발생함.

```php
# greeting_en.php
<?php
function welcome(){
    return "Hello world";
}
?>

# greeting_kr.php
<?php
function welcome(){
    return "안녕 세계";
}
?>
    
<?php
require_once 'greeting_kr.php';
require_once 'greeting_en.php';
echo welcome();
echo welcome();
?>

Fatal error: Cannot redeclare welcome() (previously declared in C:\Bitnami\wampstack-7.3.11-0\apache2\htdocs\include\greeting_kr.php:3) in C:\Bitnami\wampstack-7.3.11-0\apache2\htdocs\include\greeting_en.php on line 3
```



아래와 같이 네임스페이스를 설정하여 함수 중복 선언에러를 방지할 수 있음.

```php
<?php
namespace language\en;
function welcome(){
    return "Hello world";
}
?>

<?php
namespace language\kr;
function welcome(){
    return "안녕 세계";
}
?>

<?php
require_once 'greeting_kr.php';
require_once 'greeting_en.php';
echo language\kr\welcome();
echo language\en\welcome();
?>
```



하나의 단일 파일 내에서도 여러개의 네임스페이스를 지정할 수 있음.

```php
#greeting_all.php
<?php
namespace language\en\rev;
function welcome() {
    return 'Hello world';
}

namespace language\ko\rev;
function welcome() {
    return "안녕 세계";
}
?>

#4.php
<?php
require_once 'greeting_all.php';
echo language\en\rev\welcome();
echo language\ko\rev\welcome();
?>
```

