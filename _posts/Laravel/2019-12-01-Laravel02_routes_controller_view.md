---
layout: post
title: Udemy Laravel 02 - Routes, Controller, View
category: Laravel
tags: [PHP, Laravel]
comments: true
---



## 1. Routes

Django의 urls.py처럼 url을 정하여 View와 연동시키는 설정을 하는 영역. Routes\web.php에서 코드 입력

#### 1.1 Routes\web.php

- `Route` : 클래스 명
- `::`  : static method를 의미함
- 첫번째 파라미터로 url 주소를 입력, 두번째 파라미터로 closure function또는 controller 등을 입력할 수 있음. `view()`를 return시키거나,  text를 return하는 것도 가능함.

404 Error가 발생할 경우, `php artisan serve` 를 terminal에서 실행 시켜 접속하자

```php
<?php

Route::get('/', function () {
    return view('welcome');
});

Route::get('/welcome', function () {
    return "Hi You";
});

Route::get('/admin/post', function() {
    return "Admin is here";
});
```



#### 1.2 variable routing

url 주소를 입력하는 첫번째 파라미터에 `variable routing`으로 쓸 변수를 `{}` 로 묶어준 다음 변수를 두번째 파라미터 함수의 인자로 넘겨주면 `variable routing` 설정이 가능함.

두번째 예제처럼 `variable routing`을 여러개 설정할 수 도 있음.

```php
Route::get('/post/{id}', function($id) {
    return "This is post number ".$id;
});


Route::get('/post/{id}/{name}', function($id, $name) {
    return "This is post number ".$id . " ". $name;
});
```



#### 1.3 naming

지정한 url에 대하여 naming 설정도 가능함. `Route` 클래스의 두번째 파라미터로 `array()` 메서드를 사용. naming 명으로 해당 url 호출 가능

- 첫번째 파라미터: `as=> naming 명` 
- 두번째 파라미터:  closure 함수 입력

```php
Route::get('admin/posts/example', array('as'=>'admin.home', function() {
    
    $url = route('admin.home');

    return "This url is". $url;
}));
```

- `php artisan route:list` : 프로젝트의 route list를 반환함

```bash
$ php artisan route:list
+--------+----------+---------------------+------------+---------+--------------+
| Domain | Method   | URI                 | Name       | Action  | Middleware   |
+--------+----------+---------------------+------------+---------+--------------+
|        | GET|HEAD | /                   |            | Closure | web          |
|        | GET|HEAD | about               |            | Closure | web          |
|        | GET|HEAD | admin/posts/example | admin.home | Closure | web          |
|        | GET|HEAD | api/user            |            | Closure | api,auth:api |
|        | GET|HEAD | contact             |            | Closure | web          |
|        | GET|HEAD | post/{id}/{name}    |            | Closure | web          |
+--------+----------+---------------------+------------+---------+--------------+
```



## 2. Controller

Django의 views.py의 역할을 함. Database에서 데이터를 갖고와 Template으로 넘겨줄 수 있음. `App\Http\Controllers` 디렉토리 내에서 파일 생성 & 코드 입력

#### 2.1 PostsController.php

Laravel에서는 Terminal을 통해 손쉽게 controller를 생성할 수 있음. 

- `php artisan make:controller controller 이름`

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;

class PostController extends Controller
{
    //
}

```

CRUD 및 기본적인 로직이 담겨있는 resource가 저장된 controller를 불러와 만들 수도 있음.

- `php artisan make:controller --resource PostsController`

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;

class PostsController extends Controller
{
    /**
     * Display a listing of the resource.
     *
     * @return \Illuminate\Http\Response
     */
    public function index()
    {
        //
    }

    /**
     * Show the form for creating a new resource.
     *
     * @return \Illuminate\Http\Response
     */
    public function create()
    {
        //
    }

    /**
     * Store a newly created resource in storage.
     *
     * @param  \Illuminate\Http\Request  $request
     * @return \Illuminate\Http\Response
     */
    public function store(Request $request)
    {
        //
    }

    /**
     * Display the specified resource.
     *
     * @param  int  $id
     * @return \Illuminate\Http\Response
     */
    public function show($id)
    {
        //
    }

    /**
     * Show the form for editing the specified resource.
     *
     * @param  int  $id
     * @return \Illuminate\Http\Response
     */
    public function edit($id)
    {
        //
    }

    /**
     * Update the specified resource in storage.
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  int  $id
     * @return \Illuminate\Http\Response
     */
    public function update(Request $request, $id)
    {
        //
    }

    /**
     * Remove the specified resource from storage.
     *
     * @param  int  $id
     * @return \Illuminate\Http\Response
     */
    public function destroy($id)
    {
        //
    }
}
```



#### 2.2 Routing controller

Route 클래스의 두번째 파라미터로 생성한 controller를 입력함으로써 controller를 routing 할 수 있음. 

-  `@`를  활용하여 controller의 특정 메서드 호출 가능
- `variable routing`을 설정하여 데이터를 넘겨주는 것 또한 가능

```php
# web.php(routes)

Route::get('/post/{id}', 'PostsController@index'); # PostController의 index 메서드에 연결

#PostsController.php

    public function index($id)
    {
        //

        return "its working :: ".$id;
    }

```



#### 2.3 resource

`resource` 메서드를 이용하면 CRUD의 경로를 한줄의 코드로 할당할 수 있음.

```php
# web.php(routes)
Route::resource('post', 'PostsController');

#PostsController.php

    public function create()
    {
        //

        return "I am the method that creates stuff :)";
    }

    public function show($id)
    {
        //

        return "this is the show methods yayyyyyyy :: ". $id;
    }
```

```bash
$ php artisan route:list
+--------+-----------+------------------+--------------+---------------------------------
| Domain | Method    | URI              | Name         | Action                           
+--------+-----------+------------------+--------------+---------------------------------
|        | GET|HEAD  | post             | post.index   | PostsController@index           
|        | POST      | post             | post.store   | PostsController@store           
|        | GET|HEAD  | post/create      | post.create  | PostsController@create          
|        | GET|HEAD  | post/{post}      | post.show    | PostsController@show             
|        | PUT|PATCH | post/{post}      | post.update  | PostsController@update           
|        | DELETE    | post/{post}      | post.destroy | PostsController@destroy         
|        | GET|HEAD  | post/{post}/edit | post.edit    | PostsController@edit             
+--------+-----------+------------------+--------------+---------------------------------
```



## 3. Views

#### 3.1 Get Started

- resources/views/contact.blade.php
  - Laravel Blate Template Engine을 사용하면 `<?php echo $this ?>` 의 문법을 쉽게 `{{}}` 으로 변경하여 사용할수 있음. Simple하게 welcome.blade.php 파일의 내용을 갖고와 코드를 붙여놓도록 하자.

- Controller & Routes
  - 해당 view로 연결시키는 메서드를 PostsController.php 내에 생성 & Routes 또한 생성

```php
Route::get('/contact', 'PostsController@contact');

public function contact(){

    return view('contact');

}
```



#### 3.2 view에 data 넘기기

- Routes & Controller
  - `{id}`를 variable routing으로 controller로 넘김.
  - controller 메서드의 return 값으로 view 파일을 연결시키며, `->with()` 메서드를 통해 데이터를 넘김
  - `compact()` 메서드로 `->with()` 를 대체할 수 있음 (여러개의 파라미터를 넘길 때 유용)

```php
Route::get('/post/{id}', 'PostsController@show_post');

public function show_post($id){

    return view('post')->with('id', $id);
    #return view('post', compact('id'));
}
```

- post.blade.php

```php
<body>
    <div class="container">
    	<h1> Post {{ $id }}</h1>
    </div>
</body>
```
