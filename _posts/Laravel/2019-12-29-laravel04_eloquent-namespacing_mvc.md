---
layout: post
title: Laravel 5.7 From Scratch 04 - Eloquent / Namespacing / MVC
category: Laravel
tags: [Laracast, Laravel]
comments: true
---

#### 1. Eloquent

`php artisan make:model Project` : 모델 생성

`php artisan tinker` : 플레이 그라운드 (laraval shell의 개념)

```php
php artisan tinker
Psy Shell v0.9.11 (PHP 7.2.25 — cli) by Justin Hileman
>>> App\Project::all();
=> Illuminate\Database\Eloquent\Collection {#3006
     all: [],
	}
>>> App\Project::first();
=> null
>>> App\Project::latest()->first();
=> null
>>> $project = new App\Project;  // Project 클래스의 인스턴스 생성
=> App\Project {#2997}

// Migration 에서 생성한 create_projects_table을 적용시킴
>>> $project->title = 'My First Project';
=> "My First Project"
>>> $project->description = 'Lorem ipsum';
=> "Lorem ipsum"
>>> $project
=> App\Project {#2997
     title: "My First Project",
     description: "Lorem ipsum",
   }
>>> $project->save(); // DB에 저장
=> true

>>> App\Project::first();
=> App\Project {#3019
     id: 1,
     title: "My First Project",
     description: "Lorem ipsum",
     created_at: "2019-12-05 12:43:28",
     updated_at: "2019-12-05 12:43:28",
   }
>>> App\Project::first()->title;
=> "My First Project"
>>> App\Project::first()->description;
=> "Lorem ipsum"
>>> App\Project::all();
=> Illuminate\Database\Eloquent\Collection {#3003
     all: [
       App\Project {#3022
         id: 1,
         title: "My First Project",
         description: "Lorem ipsum",
         created_at: "2019-12-05 12:43:28",
         updated_at: "2019-12-05 12:43:28",
       },
     ],
   }
```



두번째 Project 인스턴스 생성 후 데이터베이스에 저장

```php
>>> $project = new App\Project;
=> App\Project {#3006}
>>> $project->title = 'My Second Project';
=> "My Second Project"
>>> $project->description = 'Lorem ipsum';
=> "Lorem ipsum"
>>> $project->save();
=> true
```



App\Project 데이터를 갖고오면 `collection`이 생성되어 있음을 알 수 있음.

- Collections are like arrays on steroids

```php
>>> App\Project::all();
=> Illuminate\Database\Eloquent\Collection {#3025
     all: [
       App\Project {#3026
         id: 1,
         title: "My First Project",
         description: "Lorem ipsum",
         created_at: "2019-12-05 12:43:28",
         updated_at: "2019-12-05 12:43:28",
       },
       App\Project {#3027
         id: 2,
         title: "My Second Project",
         description: "Lorem ipsum",
         created_at: "2019-12-05 12:48:01",
         updated_at: "2019-12-05 12:48:01",
       },
     ],
   }
```



모든 배열 메서드를 동일하게 적용 시킬 수 도 있음.

```php
>>> App\Project::all()[0];
=> App\Project {#3030
     id: 1,
     title: "My First Project",
     description: "Lorem ipsum",
     created_at: "2019-12-05 12:43:28",
     updated_at: "2019-12-05 12:43:28",
   }
>>> App\Project::all()[1];
=> App\Project {#3016
     id: 2,
     title: "My Second Project",
     description: "Lorem ipsum",
     created_at: "2019-12-05 12:48:01",
     updated_at: "2019-12-05 12:48:01",
   }
>>> App\Project::all()[1]->title;
=> "My Second Project"
>>> App\Project::all()->map->title;
=> Illuminate\Support\Collection {#3002
     all: [
       "My First Project",
       "My Second Project",
     ],
   }
```



#### 2. MVC Pattern

MVC 패턴에 따라 어플리케이션을 사용할 수 있으며 각 컴포넌트들의 위치는 다음과 같다.

- Model: app\Project.php
- View: resources\views\welcome.blade.php
- Controller: app\Http\Controllers\Controller.php
- +Route



#### 3. MVC 패턴 적용

ProjectsController를 생성 한 후, 라우팅 연결 & view 파일을 만들어 주도록 하자.

```php
// web.php
Route::get('/projects', 'ProjectsController@index');

//ProjectsController.php
class ProjectsController extends Controller
{
    public function index()
    {
        return view('projects.index');
    }
}

// views\projects\index.blade.php
<!doctype html>
<html lang="en">
<head>
    <title>Document</title>
</head>
<body>
    <h1>projects</h1>
</body>
</html>
```



#### 4. 모델 연결 & namespaces

라라벨은 PSR-4(an autoloading specification)이라는 컨벤션 룰에 따라 네임스페이스를 설정해줘야함.

- `$projects = \App\Project::all();`

model 은 app 디렉토리 내에 있으므로 App 으로 시작해야함. 이때, controller는 App\Http\Controllers 디렉토리 내에 있기 때문에 루트 디렉토리에서 시작하기 위해서는 네임스페이스 시작을 \으로 해야함. 그렇게 하지 않을 시, 현재 디렉토리를 기준으로 인식해버림.


```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;

class ProjectsController extends Controller
{
    public function index()
    {
        $projects = \App\Project::all();

        return view('projects.index');
    }
}
```



다른 방법으로는 아래와 같이 또한 사용 가능 함.

```php
<?php

namespace App\Http\Controllers;

use App\Project;

use Illuminate\Http\Request;

class ProjectsController extends Controller
{
    public function index()
    {
        $projects = Project::all();

        return view('projects.index');
    }
}
```



모델에 따른 데이터를 view로 넘겨줌

```php
class ProjectsController extends Controller
{
    public function index()
    {
        $projects = Project::all();

//        return view('projects.index', ['projects' => $projects]);
        return view('projects.index', compact('projects'));

    }
}
```

```php
<!doctype html>
<html lang="en">
<head>
    <title>Document</title>
</head>
<body>
    <h1>projects</h1>
    @foreach ($projects as $project)
        <li>{{ $project->title }}</li>
    @endforeach
</body>
</html>
```
