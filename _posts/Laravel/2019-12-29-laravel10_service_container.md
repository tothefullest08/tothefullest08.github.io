---
layout: post
title: Laravel 5.7 From Scratch 10 - Service Container
category: PHP
tags: [Laracast, Laravel]
comments: true
---

#### 1. GET STARTED

ProjectsController.php - show 메서드는 Project Model을 파라미터로 받음 이는 Route Model Binding을 이용한 것으로 유사한 개념으로 Service Container Component가 있음

```php
public function show(Project $project)
{
  return view('projects.show', compact('project'));
}
```



#### 2. Service Container and Auto-Resolution

 예를 들어, 라라벨은 `Filesystem` 이라는 클래스를 갖고 있음. 클래스를 파라미터로 받아 `dd($file)`  을 입력하면 인스턴스가 출력 결과를 볼 수 있음. 이처럼, 클래스를 파라미터로 받으면 라라벨이 자동적으로 클래스를 사용하게 해줌. behind scene에는 2가지의 컴포넌트가 존재함.

1. Auto-Resolution: "Okay.. you typed in Filesystem here, it seems like you want instance of that.."
2. Service Container:  Imagine there's key-value pairs in a container.  Laravel is gonna look into container and check "Do we have Filesystem here?" "Oh there it is! That's probably what the user wants. I'm going to resolve fetch, or get them from the container and give it to user!" 

```php
use Illuminate\Filesystem\Filesystem; // import to us Filesystem

// ... skip

public function show(Filesystem, $file)
{
  dd($file)
}
```



#### 3. 예시 (1)

web.php에서 다른 예시를 들어보자.  Laravel application 자체가 사실상 서비스 컨테이너임(we can get things out of here by using helpful function)

-  `app()` 또는 `resolve()` 으로 호출 가능.
-  `Filesystem` 을 기본페이지로 호출하면, 위의 show 메소드에서 출력한 값과 동일한 값이 출력되는 것을 알 수 있음.

```php
use Illuminate\Filesystem\Filesystem;

Route::get('/', function(){
  dd(app(Filesystem::class));
})
```



#### 4. 예시 (2) - put something into container

`app()->bind()` 으로 서비스 컨테이너에 등록이 가능함.

- key( `example` )으로 value(`Example` 클래스의 인스턴스 생성)을 호출 할 수 있음.  
- 이후 out of container에서 `app('eaxmple')` 을 호출하면 동일하게 `Example` 인스턴스를 출력한 결과를 볼 수 있음.

```php
// fetch 'example' out of container, what is value of accicated key.
app()->bind('example', function() {
  return new \App\Example;
})
  
Route::get('/', function(){
  dd(app('example'));
})

// 두번 호출할 경우 두개의 별도의 인스턴스가 생성됨.
Route::get('/', function(){
  dd(app('example'), app('example'));
})
```



#### 5. `singleton`

위의 마지막 예시처럼 두번 호출할 경우 두개의 별도의 인스턴스가 생성됨. 만약 단 하나의 인스턴스만 생성되길 원할 경우, `singleton` 메서드를 사용함. 

> `singleton` 메소드로 클래스나 인터페이스를 바인딩 하면 컨테이너는 한 번만 해당 의존성을 해결합니다. 싱글톤 바인딩으로 의존성이 해결되면, 컨테이너의 다른 부분에서 호출될 때 동일한 객체 인스턴스가 반환될 것입니다.

```php
app()->singleton('example', function() {
  return new \App\Example;
}
```



#### 6. Summary

컨테이너 외부에서 요청이 들어왔을 경우,  

1. 서비스 컨테이너에 존재하는지를 확인
2. 어플리케이션 내에 존재하는 클래스인지 확인 (Full relative address를 입력하면 됨)

```php
Route::get('/', function() {
    dd(app('App\Example'));
});
```



#### 7. Service container with constructor

서비스 컨테이너가 호출하는 클래스가 고유한 `__construct` 를 갖고 있는 경우, 그안의 값까지 같이 호출함

```php
<?php

namespace App;

class Example
{
    protected $foo;

    public function __construct(Foo $foo)
    {
        $this->foo = $foo;
    }
}

<?php

namespace App;

class Foo
{

}
```

아래와 같이 응용이 가능함.

```php
//web.php
app()->singleton('Twitter', function() {
   return new \App\Services\Twitter('abdsfsfdf');
});

// ProjectsController.php
public function show(Project $project)
{
  $twitter = app('twitter');
  dd($twitter);
}

// \App\Services\Twitter.php
<?php

namespace App\Services;

class Twitter
{
    protected $apiKey;

    public function __construct($apiKey)
    {
        $this->apiKey = $apiKey;
    }
}
```



#### 8. Service Container as parameter of methods

key를 Full Relative path로 변경해준 후 클래스를 호출하면, 서비스 컨테이너에 등록된 클래스를 메서드의 파라미터로 받아 올 수 있음.

```php
app()->singleton('App\Services\Twitter', function() {
   return new \App\Services\Twitter('abdsfsfdf');
});


use App\Services\Twitter;

public function show(Project $project, Twitter $twitter)
{
  dd($twitter);
}
```
