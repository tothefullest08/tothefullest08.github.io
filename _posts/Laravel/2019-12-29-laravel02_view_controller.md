---
layout: post
title: Laravel 5.7 From Scratch 02 - Views / Controllers
category: Laravel
tags: [PHP, Laracast, Laravel]
comments: true
---



#### 1. Sending Data to your Views

`Route:get()` 내에서 배열을 선언한 후, 연관배열을 return하는 것도 가능하다. 이때 `<?php : ?>` & `<?= ?>` 와 같이 traditional한 PHP 문법을 사용해도 되지만, Laracasts에서 지원하는 shorthands를 사용 할 수도 있음.

- `<?php : ?>`   => `@`
- `<?= ?>` => `{{ }}`

```php
Route::get('/', function() {
    $tasks = [
    'Go to the store',
    'Go to the market',
    'Go to the work',
];

    return view('welcome', [
        'tasks' => $tasks
    ]);
});
```

```php
@extends('layout')

@section('content')
    <h1>My First Website!!!!!!</h1>

    // 기존 방식
    <ul>
      	<?php foreach ($tasks as $task) : ?>
          	<li><?= $task; ?></li>
      	<?php endforeach; ?>
    </ul>

    // Laravel shorthand: laravel이 코드를 읽으면서 @를 <?php의 형태로 대체시킴.
    <ul>
        @foreach ($tasks as $task)
            <li>{{ $task }}</li>
        @endforeach
    </ul>

@endsection
```



#### 2. HTML 엘리먼트 route로 넘기기

routes의 return 값으로 url에서 들어오는 쿼리스트링을 넘길 수도 있다. 이때 html의 태그를 같이 넣을 경우 그대로 스트링으로 인식되되어 엘리먼트로 적용되지는 않는다. 만약 엘리먼트로 사용하고 싶을 경우, `{{ }}` 대신 `{!! !!}` 으로 묶어줘야 한다.

```php
Route::get('/', function() {
    $tasks = [
    'Go to the store',
    'Go to the market',
    'Go to the work',
    'Go to concert',
];

    return view('welcome', [
        'tasks' => $tasks,
        'foo' => request('title'),
        'foo' => '<script>alert("foobar")</script>'
    ]);
});
```

```php
@section('content')
    <h1>My {!! $foo !!} Website</h1>

    <ul>
        @foreach ($tasks as $task)
            <li>{{ $task }} </li>
        @endforeach
    </ul>
@endsection
```



#### 3. 응용

Laravel에서 지원하는 custom method 중 하나인 `with` 를 사용하여 좀 더 코드를 단축시킬 수 있음.

- `With"view에서사용하는변수명"->("정의된 변수명")`

```php
Route::get('/', function() {
    $tasks = [
    'Go to the store',
    'Go to the market',
    'Go to the work',
    'Go to concert',
];
    return view('welcome')->withTasks($tasks)->withFoo('foo');

//    return view('welcome', [
//        'tasks' => $tasks,
//        'foo' => 'foobar',
//    ]);
});
```



하나의 인자만 넘길 경우, 변형 가능

```php
Route::get('/', function() {
    return view('welcome')->withTasks([
        'Go to the store',
        'Go to the market',
        'Go to the work',
        'Go to concert',
    ]);
});
```



또다른 응용

```php
Route::get('/', function() {
    return view('welcome')->with([
        'foo' => 'bar',
        'tasks' => [
            'Go to the store',
            'Go to the market',
            'Go to the work',
            'Go to concert',
        ]
    ]);
});
```



#### 4. Controllers 101

`php artisan make:controller 컨트롤러이름` 으로 컨트롤러를 간단히 만들 수 있음. routes 코드를  다시 한번 다듬어, web.php에서는 컨트롤러를 호출하고, 엔드 포인트 설정을 컨트롤러 내 메서드를 통해 정의

- PagesController 생성 후, 각각 home, about, contact 페이지로 이동하는 메서드를 만듦.
- web.php에서는 컨트롤러와 대응되는 메서드를 호출 `PagesController@home`

```php
Route::get('/', 'PagesController@home');
Route::get('about', 'PagesController@about');
Route::get('contact', 'PagesController@contact');


namespace App\Http\Controllers;

use Illuminate\Http\Request;

class PagesController extends Controller
{
    public function home()
    {
        return view('welcome', [
            'foo' => 'bar',
        ]);
    }

    public function about()
    {
        return view('about');
    }

    public function contact()
    {
        return view('contact');
    }
}
```



