---
layout: post
title: Laravel 5.7 From Scratch 13 - Authentication / Middleware
category: PHP
tags: [Laracast, Laravel]
comments: true
---



#### 1. GET STARTED

라라벨 새 프로젝트 생성.  `Laravel new auth-example` 

웹 서비스에는 누구나 접근 가능하기보다는, authentification layer를 통해 허용된 유저만 접근이 가능해야함.

- `php artisan make:auth` : Scaffold basic login and registration views and routes(only available below Laravel 6.0)
- 라라벨 6.0 버젼 이후로는 아래의 명령어로 authentication을 위한 routes와 views를 생성할 수 있음

```bash
composer require laravel/ui --dev
php artisan ui vue --auth
```



#### 2. Database 설정

이후 기본페이지에서 회원가입을 하면 database credential 설정을 하지 않았으므로 에러가 발생함.

> Illuminate\Database\QueryException
>
> SQLSTATE[HY000] [1045] Access denied for user 'root'@'localhost' (using password: NO) (SQL: select count(*) as aggregate from `users` where `email` = foo@example.com)



간단한 설정을 위해 `.env` 파일의 데이터베이스 설정을 sqlite로 변경후 database 디렉토리 내 database.sqlite 생성. 마지막으로 `php artisan migrate`  로 데이터베이스에 마이그레이션 설정한 후 다시 회원가입을 진행해보도록 하자.

```php
DB_CONNECTION=sqlite
DB_HOST=127.0.0.1
DB_PORT=3306
```

  

#### 3. Auth 살펴보기

`php artisan tinker` 로 유저목록을 확인해보면 회원가입된 회원정보를 알 수 있음.

```php
>>> App\User::all();
=> Illuminate\Database\Eloquent\Collection {#3032
     all: [
       App\User {#3033
         id: "1",
         name: "foo",
         email: "foo@example.com",
         email_verified_at: null,
         created_at: "2019-12-09 06:24:34",
         updated_at: "2019-12-09 06:24:34",
       },
     ],
   }
```



Routes\web.php를 보면 `Auth::routes()` 가 추가된 것을 볼 수 있음. authentication에 대한 라우팅이 기본적으로 설정된 것으로 아래와 같이 두가지 방법으로 상세 정보를 확인 할 수 있음.

1. `php artisan route:list`
2. App\controllers\auth 내 생성된 컨트롤러 확인



Routes\web.php에 보면 /home 엔드포인트에 대하여 HomeController@index 를 호출하는 라우팅이 설정되어 있음. 이에 따라 HomeController.php를 살펴보면, 생성자 메서드로 middleware 메서드를 호출하는 것을 알 수 있음.  

```php
Route::get('/home', 'HomeController@index')->name('home');

<?php

namespace App\Http\Controllers;
use Illuminate\Http\Request;

class HomeController extends Controller
{
    public function __construct()
    {
        $this->middleware('auth');
    }
)
```



#### 4. Middleware

미들웨어는 layers of Onion 처럼 이해하면 됨. 제일 안에 있는 layer로 가기 위해서는 바깥에 있는 layer들을 모두 거쳐야 함. App\Http\Kernel.php 에는 라라벨에 저장되어 있는 미들웨어의 목록을 볼 수 있음.

App\Http\Middleware\Authenticate.php를 살펴보면

-  Illuminate\Auth\Middleware\Authenticate as Middleware를 export 하는 것을 알수 있음. 
- Illuminate\Auth\Middleware\Authenticate.php는 미들웨어로서, 인증에 관한 로직을 구현해놓음. 
- 특히 `handle` 메서드를 통해 다음 미들웨어로 넘길 수 있음.

```php
// Illuminate\Auth\Middleware\Authenticate.php

public function handle($request, Closure $next, ...$guards)
{
  $this->authenticate($request, $guards);

  return $next($request);
}
```



web.php를 보면 /home은 HomeController로 연결되어있으며, 이 클래스의 생성자는  `middleware('auth')`를 호출함. 여기서 `middleware('auth')` 는 App\Http\Kernel.php 내 `$routeMiddleware` 배열에 있는 `auth` 의 값을 호출 함.

```php
// web.php
Route::get('/home', 'HomeController@index')->name('home');

// HomeController.php
public function __construct()
{
  $this->middleware('auth');
}
```

`php artisan make:middleware` 로 새로운 미들웨어를 생성하는 것도 가능함.  이후 kernel.php 내 미들웨어 배열안에 클래스를 추가하면 미들웨어로써 동작을 하게 됨.



#### 5. You May Only View Your Projects

인스타그램, 페이스북 등과 같이, 내가 업로드한 프로젝트 리스트만 볼 수 있도록 설정하도록 하자. 유저 관련해서 다양한 메서드를 사용할 수 있음. 아래처럼 유저별로 프로젝트를 보여주기 위해서는 마이그레이션 & 스키마를 다시 설정해줘야함.

- `auth()->id()`  ex) 4
- `auth()->user() `   ex) user
- `auth()->check()`  ex) boolean
- `auth()->guest()`  ex) guest user

```php
// ProjectsController.php
public function index()
{
  // $projects = Project::all();
  auth()->id(); // 4
  auth()->user(); // user
  auth()->check(); // boolean
  auth()->guest(); // guest user

  // select * from projects where owner_id = 4
  $projects = Project::where('owner_id', auth()->id())->get(); 

  return view('projects.index', compact('projects'));
}
```



#### 6. Migration - Foreign key 

\database\migration\create_project_table.php 내에서 `owner_id` 라는 새로운 컬럼 추가하며 방법은 2가지가 있음

1. `$table->unsignedBigInteger('owner_id')->index();`
2. `$table->unsignedBigInteger('owner_id')` & `$table->foreign('owner_id')->references('id')->on('users');`   `owner_id` 라는 외래키는 user 테이블의 id를 참조함.
   - 추가로 `onDelete` 설정도 가능함.

이후 `php artisan migrate:fresh` 를 통해 마이그레이션을 새로고침.

```php
public function up()
{
  Schema::create('projects', function (Blueprint $table) {
    $table->bigIncrements('id');
    $table->unsignedBigInteger('owner_id');
    $table->string('title');
    $table->text('description');
    $table->timestamps();
    
    $table->foreign('owner_id')->references('id')->on('users')->onDelete('cascade');
  });
}
```



#### 7. Authentication 설정

`http://127.0.0.1:8000/projects/create` 에 대한 사용자 인증을 진행하기 위해 기본 설정 진행. 로그인 사람만 해당 엔드포인트에 도달할 수 있도록 미들웨어를 추가할 수 있음.

```php
composer require laravel/ui --dev
php artisan ui vue --auth
```

```php
class ProjectsController extends Controller
{
  	// 생성자 메서드를 만들어, middleware('auth')를 추가시킴.
    public function __construct()
    {
        $this->middleware('auth');
    }
```



클래스의 특정 메서드에만 사용자 인증이 진행되도록 설정을 할 수 있음. 

```php
public function __construct()
{
  $this->middleware('auth');
  $this->middleware('auth')->only(['store','update']);
  $this->middleware('auth')->except(['show'])
}
```



클래스의 생성자 메서드에서 미들웨어를 호출하지 않고, 라우터에서 바로 호출하는 것도 가능함.

```php
Route::post('/projects/{project}/tasks', 'ProjectTasksController@store')
  ->middleware('auth');
```



#### 8. Controller - Add Foreignkey attribute

`ProjectController@store` 메서드에서 `owner_id` 를 추가해야함. 방법은 아래와 같이 2가지임

1. `Project::create($attribute + ['owner_id' => auth()->id()]);` 
2. `$attribute['owner_id'] = auth()->id();` 

```php
// before
public function store()
{
  $attribute = request()->validate([
    'title' => ['required', 'min:3'],
    'description' => ['required','min:3'],
  ]);

  Project::create($attribute);

  return redirect('/projects');
}

// after
// option 1
public function store()
{
  $attribute = request()->validate([
    'title' => ['required', 'min:3'],
    'description' => ['required','min:3'],
  ]);

  Project::create($attribute + ['owner_id' => auth()->id()]);

  return redirect('/projects');
}

// option 2
public function store()
{
  $attribute = request()->validate([
    'title' => ['required', 'min:3'],
    'description' => ['required','min:3'],
  ]);

  $attribute['owner_id'] = auth()->id();

  Project::create($attribute);

  return redirect('/projects');
}
```

