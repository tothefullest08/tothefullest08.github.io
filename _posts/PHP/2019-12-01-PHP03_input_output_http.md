---
layout: post
title: 생활 코딩 PHP 03 - 입/출력 그리고 form과 http
category: PHP
tags: [생활코딩, PHP]
comments: true
---



## 입/출력 그리고 form과 http

#### 1-1. 입력과 출력

- PHP 어플리케이션은 URL을 통해 데이터를 입력 받을 수 있음.
- `http://localhost/IO_form/25.php?id=harry` 로 입력할 경우, `id`의 값이 출력됨.
- `http://localhost/IO_form/25.php` : 주소
- `id=harry` : 입력 데이터 영역

```php
<?php
echo $_GET['id']."님 안녕하세요."; #사용자가 입력된 정보를 치환하여 줌.
?>
    
#출력값: harry 님 안녕하세요. 가 출력됨.
```

```php
<?php
echo $_GET['id'].','.$_GET['name'];
?>

#입력값: http://localhost/IO_form/25.php?id=harry&name=potter
#출력값: harry,potter
```



#### 1-2. HTML FORM

- HTML FORM 태그를 활용하여 데이터를 전달 시킬 수 있음.

```html
<!-- 1.html -->
<html>
<body>
    <form method="get" action="25.php">
        id :  <input type="text" name="id" />
        password :  <input type="text" name="password" />
        <input type="submit" />
    </form>
</body>
</html>
```

```php
# 25.php
<?php
echo $_GET['id'].','.$_GET['password'];
?>
```



#### 1-3. GET vs POST

- GET 방식으로 데이터 전송 시, URL에 데이터를 포함 시킴
- POST 방식은 데이터를 URL에 포함시키지 않고 전송 가능

```html
<!-- 2.html -->
<html>
<body>
    <form method="POST" action="21.php">
        id :  <input type="text" name="id" />
        password :  <input type="text" name="password" />
        <input type="submit" />
    </form>
</body>
</html>
```

```php
# 21.php
<?php
echo $_POST['id'].', '.$_POST['password'];
?>
```

