---
layout: post
title: Laravel 5.7 From Scratch 11 - Service Provider
category: Laravel
tags: [PHP, Laracast, Laravel]
comments: true
---



#### 1. GET STARTED

App\Providers를 보면 라라벨에서 제공하는 기본 providers를 볼 수 있음. App\config\app.php를 보면 Autoloaded Service Providers 목록을 볼 수 있으며, 이 목록을 어플리케이션에 요청이 올때마다 자동적으로 불러옴.



#### 2. `boot` & `register`

서비스 프로바이더는 `boot()` 와 `register()` 두 가지의 메서드를 갖고 있음.

- `register` : 서비스 컨테이너에 바인딩. 라라벨은 App\config\app.php - providers에 있는 서비스 프로바이더들을 돌면서 `register` 메서드를 호출함.
- `boot` : register이 끝난 이후, 라라벨은 다시한번 pp\config\app.php - providers에 있는 서비스 프로바이더들을 돌면서 `boot` 메서드를 호출함.

```php
class AppServiceProvider extends ServiceProvider
{

    public function register()
    {
      $this->app->singleton('foo', function() {
        return 'bar';
      });
    }

    public function boot()
    {

    }
}
Route::get('/', function() {
    dd(app('foo'));

    return view('welcome');
});
```



#### 2-1. 응용

```php
//AppServiceProvider.php
public function register()
{
  $this->app->singleton(Twitter::class, function() {
    return new Twitter('api-key');
  });
}

//web.php
use App\Services\Twitter;

Route::get('/', function(Twitter $twitter) {
    dd($twitter);

    return view('welcome');
});
```



#### 3. Create new service provider

새로운 서비스 프로바이더 생성하여 위의 로직을 새로운 프로바이더에 연결시킬 수 있음

- `php artisan make:provider SocialServiceProvider`
- app\config\app.php에 서비스 프로바이더 추가

```php
// SocialServiceProvider
public function register()
{
  $this->app->singleton(Twitter::class, function() {
    return new Twitter('api-key');
  });
}

// app\config\app.php
    'providers' => [
        /*
         * Application Service Providers...
         */
        App\Providers\AppServiceProvider::class,
        App\Providers\AuthServiceProvider::class,
        // App\Providers\BroadcastServiceProvider::class,
        App\Providers\EventServiceProvider::class,
        App\Providers\RouteServiceProvider::class,
        App\Providers\SocialServiceProvider::class,

    ],
```



#### 4. Service Provider 활용

`AppServiceProvider@register`에서 새로운 클래스 바인딩 코드 구현

```php
public function register()
{
  $this->app->bind(
    \App\Repositories\UserRepository::class,
    \App\Repositories\DbUserRepository::class
  );
}
```

\App\Repositories 내에서 UserRepository & DbUserRepository 파일 생성

```php
// UserRepository.php
<?php

namespace App\Repositories;

interface UserRepository
{
    public function create($attributes);
}

// DbUserRepository.php

<?php

namespace App\Repositories;

class DbUserRepository implements UserRepository
{
    public function create($attributes)
    {
        User::create();
        dd('creating the user');
    }
}
```



이후, web.php에서 클래스를 호출하면 됨.

```php
use App\Repositories\UserRepository;

Route::get('/', function(UserRepository $users) {
    dd($users);

    return view('welcome');
});
```



AppServiceProvider.php에서 register 메서드의 내용을 주석처리할 경우 아래처럼 인스턴스 에러가 발생함.

```php
Illuminate\Contracts\Container\BindingResolutionException
Target [App\Repositories\UserRepository] is not instantiable.
```
