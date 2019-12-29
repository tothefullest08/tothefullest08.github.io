---
layout: post
title: Laravel 5.7 From Scratch 06 - Routing Conventions / Faking PATCH & DELETE request
category: Laravel
tags: [PHP, Laracast, Laravel]
comments: true
---



#### 1. Routing Conventions Worth Following

Laravel에서 권장하는 Routing Conventions에 따라 routing을 아래와 같이 RESTful하게 설정 할 수 있음.

| Method | URI              | Controller Method |
| ------ | ---------------- | ----------------- |
| GET    | /projects        | index             |
| GET    | /projects/create | create            |
| GET    | /projects/1      | show              |
| POST   | /projects        | store             |
| GET    | /projects/1/edit | edit              |
| PATCH  | /projects/1      | update            |
| DELETE | /projects/1      | destory           |

이에 따라 routing 코드를 web.php에 작성하면 다음과 같을 수 있음. 또한, 아래 코드는 `resource`  라는 스태틱 메서드를 이용하여 shortcut으로 작성 할 수도 있다. `php artisan route:list` 로 등록된 routes를 비교해보면 똑같음을 알 수 있다.

```php
Route::get('/projects', 'ProjectsController@index');
Route::get('/projects/create', 'ProjectsController@create');
Route::get('/projects/{project}', 'ProjectsController@show');
Route::post('/projects', 'ProjectsController@store');
Route::get('/projects/{project}/edit', 'ProjectsController@edit');
Route::patch('/projects/{project}', 'ProjectsController@update');
Route::delete('/projects/{project}', 'ProjectsController@destroy');

//shortcut - resource. first parameter shall be thing we are manipulating
Route::resource('projects','ProjectsController')
```



#### 2. resource 자동 생성

`php artisan make:controller PostController -r` : create all resourceable method for specific controller

`php artisan make:controller PostController -r -m Post` : create Model & allocate model object to the parameter of each method properly

```php
// example of php artisan make:controller PostController -r
<?php

namespace App\Http\Controllers;
use Illuminate\Http\Request;
class PostController extends Controller

{
    public function show($id)
    {
        //
    }
}

// example of php artisan make:controller PostController -r -m Post
<?php

namespace App\Http\Controllers;
// added Model Post
use App\Post;
use Illuminate\Http\Request;
class PostController extends Controller

{
    public function show(Post $post)
    {
        //
    }
}
```



#### 3. Faking PATCH and DELETE Requests

routes(Web.php)에 명시된 Restful하게 설계된 URI에 따라, 수정, 삭제에 대한 로직을 MVC 패턴으로 구현 할 수 있음.

```php
Route::get('/projects/{project}/edit', 'ProjectsController@edit');
Route::patch('projects/{project}', 'ProjectsController@update');
Route::delete('projects/{project}', 'ProjectsController@destroy');
```

ProjectsController.php 내 edit에 대한 메서드 구현. 특정 값에 대하여 수정하는 것이 일반적이므로 기준이 되는 `$id` 를 인자로 받아 `::find` 를 통해 해당되는 오브젝트를 찾아줌. 이후 오브젝트를 `compact()` 로 감싸, 전체 데이터를 템플릿으로 넘김.

`update` 메서드는 edit 템플릿에서 수정된 내용을 받아와 데이터베이스에 저장 & redirect 등을 설정하는 함수.

```php
public function edit($id)
{
  $project = Project::find($id);

  return view('projects.edit', compact('project'))
}

public function update($id)
{
  $project = Project::find($id);
  $project->title = request('title');
  $project->description = request('description');

  $project->save();

  return redirect('/projects');
}
```



#### 4. base template 설정

edit.blade.php 작성에 앞서 베이스 템플릿의 역할을 하는 layout.blade.php를 먼저 작성 한 후, 공통되는 영역을 상속 받음.

- `@yield('content')` : 상속 받은 템플릿의 코드가 삽입되는 부분 in layout.blade.php
- `@extends('layout')` : 상속받을 템플릿 명 입력(최상단에)
- `@section('content') @endsection` : 코드 입력

```php
<!doctype html>
<html lang="en">
<head>
    <title>Document</title>
</head>
<body>
    <div class="container">
        @yield('content')
    </div>
</body>
</html>
```



#### 5. `method_field('PATCH')`

수정 페이지에서는 당연히 기존의 데이터가 보여야하므로, `input` 태그의 `value` 값과, `textarea` 태그 안에 각각의 값을 `{% raw %}{{ }}{% endraw %}` 으로 표현.

수정에 대한 HTTP Method는 PATCH이나, 지원되지 않으므로 폼태그에서는 POST 방식으로 데이터를 보내되, 라라벨이 실제로는 PATCH 라는 것을 알 수 있도록 `{% raw %}{{ method_field('PATCH')}}{% endraw %}` 를 입력해 줌. 또한 CSRF 공격을 막기 위해 `{% raw %}{{ csrf_field() }}{% endraw %}`  입력.

```html
@extends('layout')

@section('content'){% raw %}
<h1 class=""title>Edit Project</h1>

<form method="POST" action="/projects/{{ $project->id }}">
  {{ method_field('PATCH') }}
  {{ csrf_field() }}
  <div>
    <label class="label" for="title">Title</label>
    <input type="text" class="input" name="title" placeholder="Title" value="{{ $project->title }}">
  </div>
  <div>
    <label class="label" for="description">Description</label>
    <textarea name="description" class="textarea" >{{ $project->description }}</textarea>
  </div>

  <div>
    <button type="submit" class="button is-link">Update Project</button>
  </div>
</form>
@endsection{% endraw %}
```



#### 6. Delete Requests

마찬가지로 Delete Requests도 로직 구현이 가능함. Edit.blade.php 내 delete 요청을 보내는 코드 구현. 이후 controller 내에서 `destroy` 메서드 구현. `findOrFail` 은 `find` 의 응용 버젼으로, 존재하지 않는 url으로 요청이 왔을 경우, 404에러를 날려줌.

```html
{% raw %}
<form method="POST" action="/projects/{{ $project->id }}">
  {{ method_field('DELETE') }}
  {{ csrf_Field() }}

  <div>
    <button type="submit">Delete Project</button>
  </div>
</form>
{% endraw %}
```

```php
public function destroy($id)
{
  $project = Project::findOrFail($id)->delete();

  return redirect('/projects');
}
```
