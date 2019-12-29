---
layout: post
title: Laravel 5.7 From Scratch 01 - Basic Routing / Blade Layout Files
category: Laravel
tags: [PHP, Laracast, Laravel]
comments: true
---



#### 1. Basic Routing

`php artisan` : 라라벨 프로젝트에서 사용할 수 있는 명령어

`php artisan serve` : run development server

- Routes/web.php - route(url 경로) 지정
- Resources/views/ - router로 연결된 view 파일 생성

```php
// routes/web.php
<?php

Route::get('/', function () {
    return view('welcome');
});

Route::get('/contact', function () {
    return view('contact');
});

// resources/views/welcome.blade.php & contact.blade.php
<body>
    <h1>here we go</h1>
    <p>
        <a href="/contact">Contact </a> us to learn more.
    </p>
</body>
</html>
```



#### 2. Blade Layout Files

layout.blade.php 를 views 디렉토리 내에 생성하여 공통적으로 쓰이는 base template으로 사용할 수 있음. 다른 템플릿들은 `@yield()`  내 입력된 스트링을 통해 layout.blad.php을 상속 받을 수 있음.

- `@extends('base template 경로')`
- `@section('') @endsection`  : `@yield('')` 내 입력된 스트링을 입력

```php
<!-- layout.blade.php -->
<!doctype html>
<html lang="en">
<head>
    <title>@yield('title', 'laracasts')</title>
</head>
<body>

    <ul>
        <li><a href="/">Home</a></li>
        <li><a href="/about">About Us </a></li>
        <li><a href="/contact">Contact </a> us to learn more.</li>
    </ul>
    @yield('content')
</body>
</html>

<!-- welcome.blade.php -->
@extends('layout')
{{--layouts/app : layouts 디렉토리 내 app 파일을 의미--}}

@section('content')
    <h1>My First Website!!!!!!</h1>
@endsection
```



`@yield()` 의 두번째 인자로 default 값을 설정할 수 있으며, template을 상속받아 입력하는 내용이 길지 않다면 마찬가지로 `@section()`의 두번째 인자로, 입력 값을 간단하게 입력 할 수 도 있음.

```php
//layout.blade.php
<head>
    <title>@yield('title', 'laracasts')</title>
</head>

// about.blade.php
@section('title', 'about')
```
