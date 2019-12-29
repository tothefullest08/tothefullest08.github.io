---
layout: post
title: The PHP Practitioner 01 - Variable / PHP and HTML / Seperate Logic
category: PHP
tags: [Laracast, PHP]
comments: true
---



#### 1. Variable

```bash
php -h #php 관련 명령어 확인가능
php -S <addr>:<port> # built-in web server 실행
php -S localhost:8888
```

basic formular:  `$변수명`

```php
<?php
$greeting = 'hello universe';
echo $greeting;
```

```php
<?php
$greeting = 'Hello';
$name = 'Harry Lee';
echo $greeting. ' '. $name;
echo "$greeting, $name";
echo "{$greeting}, {$name}";
```



#### 2. PHP and HTML

PHP와 HTML을 같이 쓸 경우에는 닫는 태그 `?>` 가 반드시 필요함

```php+html
<!DOCTYPE HTML>
<html lang="en">
<head>
    <meta charset=UTF-8">
    <title>Document</title>
    <style>
        header {
            background: #e3e3e3;
            padding: 2em;
            text-align: center;
        }
    </style>
</head>
<body>
    <header>
        <h1><?php echo 'Hello world';?></h1>
    </header>
</body>
</html>
```

쿼리스트링이 url에 포함되는 경우(`http://localhost:8000/?name=harry`)

- `$_GET` 을 사용하여 value값을 갖고 올 수 있음.
- 해당 방법은 다음과 같이 확장/활용 가능. `<?php` 는 `<?=`와 같이 shorthand로도 나타 낼 수 있음.

```php
<h1>
  <?php
  	$name = $_GET['name']; # harry
  	echo "Hello, $name";
  ?>
</h1>
<h1>
  <?php echo "Hello, ". $_GET['name']; ?>
</h1>
<h1>
  <?= "Hello, ". $_GET['name']; ?>
</h1>
```



보안상의 문제? 누군가가 태그 자체를 입력하여 HTML을 수정하려 한다면? (`http://localhost:8000/?name=<small>harry</small>`)

- `Htmlspecialchar()` 메서드를 사용

```php+HTML
<h1>
  <?= "Hello, ". htmlspecialchars($_GET['name']); ?> 
  <!-- 출력 값: Hello, <small>harry</small> -->
</h1>
```



#### 3. Seperate PHP Logic From Presentation

Html: render Page &  PHP: connect to DB, Add new data.

따라서 역할에 따라 두개의 파일들을 분리시키는 것이 가장 최적해임.

```php
# index.html
<?php

$greeting = "Hello World";

require 'index.view.php'; 
```

```html
<!-- index.view.php -->
<!DOCTYPE HTML>
<html lang="en">
<head>
    <meta charset=UTF-8">
    <title>Document</title>
    <style>
        header {
            background: #e3e3e3;
            padding: 2em;
            text-align: center;
        }
    </style>
</head>
<body>
<header>
    <h1>
        <?= $greeting; ?>
    </h1>
</header>
</body>
</html>
```
