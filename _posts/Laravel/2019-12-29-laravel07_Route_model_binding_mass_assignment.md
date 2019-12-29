---
layout: post
title: Laravel 5.7 From Scratch 07 - Route Model Binding / Mass Assignment
category: PHP
tags: [Laracast, Laravel]
comments: true
---



#### 1. Route Model Binding

> **Laravel route model binding** provides a convenient way to automatically inject the **model** instances directly into your **routes**. For example, instead of injecting a user's ID, you can inject the entire User **model** instance that matches the given ID.

route model binding을 이용하여, `$id` 를 파라미터로 받아, `$project = Project::find($id);` 를 입력하지 않아도 오브젝트를 찾는 것이 가능함.

- 함수(모델명, 변수명)

```php
public function edit(Project $project)
```

또한,  `ProjectController@show` 에도 route model binding을 적용할 수 있음.

```php
public function show(Project $project)
{
  return view('projects.show', compact('project'));
}
```

```html
@extends('layout')

@section('content')
    <h1>{{$project->title}}</h1>
    <div>{{$project->description}}</div>
    <p>
        <a href="/projects/{{$project->id}}/edit">edit </a>
    </p>
@endsection
```



#### 2. Mass Assignment

모델인 Project.php에서 massive assignment에 대한 설정을 할 수 있음. massive assignment를 통해 HTML으로 입력받을 수 있는 데이터베이스의 필드를 설정 할 수 있음. 이러한 설정을 하지 않을 경우, 브라우저의 개발자 도구를 악용하여 hidden으로 요청을 보내 데이터베이스를 조작하는 것이 가능해짐. 

massive assignment를 설정하는 방법은 크게 아래와 같이 2가지 방법이 있음.

- `protected $fillable = []`  : 입력이 가능한 필드명을 배열안에 저장
- `protected $guarded = []` : 입력이 불가능한 필드명을 배열에 저장. 빈 배열일 경우 모든 필드를 허용함

```php
class Project extends Model
{
    protected $fillable = [
        'title', 'description'
    ];

//    protected $guarded = [
//    ];
}
```



#### 3. Clean up the codes

마지막으로 ProjectController.php - 메서드 (`store` , `update` ) 내 짜여진 코드를 좀 더 clean하게 변경 시킬 수 있음

```php
// before
public function store()
{
  $project = new Project();
  $project->title = request('title');
  $project->description = request('description');
  $project->save();

  return redirect('/projects');
}

public function update($id)
{
  $project = Project::find($id);
  $project->title = request('title');
  $project->description = request('description');
  $project->save();
  return redirect('/projects');
}

// after
public function store()
{
  Project::create(request(['title','description']));

  return redirect('/projects');
}

public function update(Project $project)
{
  $project->update(request(['title', 'description']));

  return redirect('/projects');
}
```


