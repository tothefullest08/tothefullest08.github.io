---
layout: post
title: Laravel 5.7 From Scratch 05 - Directory Structure / Form Handling / CSRF
category: Laravel
tags: [PHP, Laracast, Laravel]
comments: true
---



#### 1. Directory Structure Review

`editorconfig` : configuration for editor

`.env` : store important configration items (db password)

`artisan` : when `php artisan` is typed, this file is triggered

`composer.json` : configuration file that specifies all of dependencies (maybe only for development purposes)

`composer.lock` : taks all of dependencies you pulled in, locks them to specific versions - ensure the consistency

`package.json` : help with front-end compilation. dependencies are installed via Node.js and represented in javascript.

`webpack.mix.js` : wrapper on webpack. compiling javascript, scss, less. compile output to go.

`yarn.lock` : similar to `composer.lock`

`\vender` : where all dependencies are installed.

`\tests` : execture test.

`\storage`: storage files.

`\routes` : store routes.

- `console.php` : possible to make custom artisan command.
- `channels.php`: broadcasting channels. how will server side communicate with front end.
- `api.php` : for the client routes specified for api.

`\resources` : views, js, sass etc that could be compiled down are stored.

`\public` : where compiled file goes. CSS, JS are coming from other files in `\resources` that are complied down using `web pack.mix.js`

`\database` : store all of migrations, database seeding, factories.

`\config` : various configuration settings are stored.

`\bootsrap` : framework bootstrap itself.

`\app` : where your app lives. all of your model/controller/more complicated artisan command(\\console directory) go,

- `Http\middleware` : layers of onion...  when request comes in, user visits page. it is gonna go throw all of these layer(middleware). literally loop. loop throw all of these layers and trigger necessary layer. each of layers in these onion has the opportunity to respond to request before hit controller. in kerne file, you can find `$middleware` array  that all of middleware are run during every single request to your application.
- `Providers`



#### 2. Form Handling and CSRF Protection

> create.blade.php에서 폼 태그 내용을 POST 방식으로 지정된 URI(`/projects`)로 보내는 로직 구현

Routes\web.php 내에 routing 코드 입력

```php
//post 방식으로 /projects라는 url로 요청이 들어왔을 경우, ProjectsController@store메서드 호출
Route::post('/projects', 'ProjectsController@store');
//마찬가지로, get방식으로 하기 주소로 요청이 들어왔을 경우.
Route::get('/projects/create', 'ProjectsController@create');
```



Resources\views\projects\create.blade.php 내 템플릿 코드 작성

- POST 방식으로 /projects에 대한 경로로 요청을 보낼 경우, web.php에 따라 정의된 url패턴에 따라 `ProjectsController@store` 가 호출 됨.
- 이때, `{{ csrf_field() }}` 을 통해 CSRF TOKEN을 생성하여 세션에 저장된 토큰값과 일치하는지 검증. 이를 통해 CSRF 공격을 막을 수 있음.

```html
<h1>Create New Project</h1>

<form method="POST" action="/projects">
  {{ csrf_field() }}
  <div>
    <input type="text" name="title" placeholder="Project title">
  </div>
  <div>
    <textarea name="description" placeholder="Project description"></textarea>
  </div>
  <div>
    <button type="submit">Create Project</button>
  </div>
</form>
```



ProjectsController.php 내 메서드 구현

- `return redirect('/projects')` : return to this url
- `return request()->all()` : return all data
- `return request('title')` : return the value of this specific key(title)

```php
public function create()
{
  return view('projects.create');
}

public function store()
{
  $project = new Project();

  $project->title = request('title');
  $project->description = request('description');

  $project->save();

  return redirect('/projects');
  //        return request()->all();
  //        return request('title');
}
```

