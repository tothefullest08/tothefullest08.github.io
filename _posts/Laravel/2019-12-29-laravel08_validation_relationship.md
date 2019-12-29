---
layout: post
title: Laravel 5.7 From Scratch 08 - Validation / Eloquent Relationship(1:N)
category: PHP
tags: [Laracast, Laravel]
comments: true
---



#### 1. Two Layers of Validation

폼 태그로 입력받은 데이터에 대한 유효성 검증(예. 입력하지 않은채 제출할 경우) 은 크게 2가지 방법으로 클라이언트 사이드와 서버 사이드에서 각각 활용이 가능하다. 

클라이언트단에서 간단하게 입력 태그에 `required`를 입력하면 된다. 하지만 개발자 도구로 `required`를 삭제할 수 도 있다. 이런 경우, 서버사이드 단에서 추가로 유효성 검증을 아래와 같이 진행한다.



#### 2. `request()->validate([])`

`request()->validate([])`는 유효성 검증의 attribute를 반환한다.

```php
request()->validate([
  'title' => 'required', // 반드시 필요한 경우
  'description' => ['required', 'min:3'] // min string length: 3
  'password' => ['required', 'confirmed'] // 패스워드에 대한 유효성 검증
]);
```

 `request(['title', 'description'])` 으로 대체 시킬 수 있음.

```php
public function store()
{
  $attribute = request()->validate([
    'title' => ['required', 'min:3'], 
    'description' => ['required','min:3'],
  ]);

  Project::create($attribute);

  return redirect('/projects');
}
```



#### 3. `$errors & old()`

이 후, 에러메세지를 브라우저에서 보여주고 싶다면, 라라벨에서 기본적으로 렌더하는 `$errors` 를 활용할 수 있음. 

```php
<?php if ($errors->any()) : ?>
  <div>
  	<ul>
  		<?php foreach($errors->all() as $error) : ?>
    		<li> {{ $error }}</li>
    	<?php endforeach ?>
    </ul>
  </div>
<?php endif; ?>
```



마지막으로 유효성 검증이 실패했을 경우, 라라벨은 자동적으로 해당 페이지를 리다이렉트 시키는데, 이때 기존에 입력된 값을 브라우저에 남기고 싶을 경우에는 `{{old('title')}}` 을 입력하면된다.

```html
<div>
  <input type="text" name="title" placeholder="Project title" 
         required value="{{ old('title') }}">
</div>

<div>
  <textarea name="description" placeholder="Project description" required> 
    {{ old('description') }} 
  </textarea>
</div>
```



#### 4. Your First Eloquent Relationships

`php artisan help make:model` : 모델 생성에 대한 도움말 확인 가능

```bash
Description:
  Create a new Eloquent model class

Usage:
  make:model [options] [--] <name>

Arguments:
  name                  The name of the class

Options:
  -a, --all             Generate a migration, factory, and resource controller for the model
  -c, --controller      Create a new controller for the model
  -f, --factory         Create a new factory for the model
      --force           Create the class even if the model already exists
  -m, --migration       Create a new migration file for the model
  -p, --pivot           Indicates if the generated model should be a custom intermediate table model
  -r, --resource        Indicates if the generated controller should be a resource controller
  -h, --help            Display this help message
  -q, --quiet           Do not output any message
  -V, --version         Display this application version
      --ansi            Force ANSI output
      --no-ansi         Disable ANSI output
  -n, --no-interaction  Do not ask any interactive question
      --env[=ENV]       The environment the command should run under
  -v|vv|vvv, --verbose  Increase the verbosity of messages: 1 for normal output, 2 for more verbose output and 3 for debug
```



#### 5. Model, Migration, Factory 생성

`project cupist$ php artisan make:model Task -m -f` : `Task` 모델, 마이그레이션, 팩토리 생성

\database\migrations\2019_12_07_090008_create_tasks_table.php에서 스키마 재 구성한 후 `php artisan migrate` 명령어를 통해 마이그레이션을 데이터베이스에 적용.

```php
class CreateTasksTable extends Migration
{
    public function up()
    {
        Schema::create('tasks', function (Blueprint $table) {
            $table->bigIncrements('id');
            $table->unsignedBigInteger('project_id'); // 외래키 설정
            $table->string('description'); // 일의 설명
            $table->boolean('completed')->default(false); // 완료 유무
            $table->timestamps();
        });
    }
```



#### 6. 관계 설정

Project.php & Task.php 모델에서 관계설정을 진행

- `$this->hasMany(Task::class)`   - 1:N 관계 설정
- `$this->belongsTo(Project::class)` - N에서 1로 접근할 때, 부모의 모델 설정

```php
// Project.php
class Project extends Model
{
    protected $guarded = [];

    public function tasks()
    {
        // 1:N관계설정 시 hasMany 사용
        return $this->hasMany(Task::class); 
    }
}

// Task.php

class Task extends Model
{
  	// 1:N 관계설정 후, N에서 1로 접근시에는 belongsTo 사용
    public function project()
    {
        return $this->belongsTo(Project::class);
    }
}
```

MySQL에 접속하여 tasks 테이블에 샘플 데이터를 입력

```mysql
INSERT INTO tasks(project_id, description, created_at, updated_at) VALUES(2, 'purchase map', NOW(), NOW());
INSERT INTO tasks(project_id, description, created_at, updated_at) VALUES(2, 'Inform school', NOW(), NOW());
```



#### 7. php artisan tinker 연습

`php artisan tinker`를 통해 관계 설정된 데이터들의 형태를 볼 수 있음. 

- 1에서 N으로 접근시, `App\Project::first()->tasks;`  모델의 메서드를 호출하는 것이지만 `()`은 입력하지 않음.
- 마찬가지로, N에서 1로 접근 시에도 `APP\Task::first()->project;` 으로 접근

```php
>>> App\Project::first();
=> App\Project {#3015
     id: 2,
     title: "My Second Project22",
     description: "Lorem ipsum",
     created_at: "2019-12-05 12:48:01",
     updated_at: "2019-12-07 06:46:38",
   }

>>> App\Project::first()->tasks;
=> Illuminate\Database\Eloquent\Collection {#3004
     all: [
       App\Task {#3015
         id: 2,
         project_id: 2,
         description: "purchase map",
         completed: 0,
         created_at: "2019-12-07 18:15:55",
         updated_at: "2019-12-07 18:15:55",
       },
       App\Task {#3027
         id: 3,
         project_id: 2,
         description: "Inform school",
         completed: 0,
         created_at: "2019-12-07 18:16:39",
         updated_at: "2019-12-07 18:16:39",
       },
     ],
   }
>>> APP\Task::first()
=> App\Task {#3020
     id: 2,
     project_id: 2,
     description: "purchase map",
     completed: 0,
     created_at: "2019-12-07 18:15:55",
     updated_at: "2019-12-07 18:15:55",
   }
>>> APP\Task::first()->project;
=> App\Project {#3000
     id: 2,
     title: "My Second Project22",
     description: "Lorem ipsum",
     created_at: "2019-12-05 12:48:01",
     updated_at: "2019-12-07 06:46:38",
   }
```



#### 8. Views에 관계 나타내기

Show.blade.php에서도 마찬가지로 메서드이지만 프로퍼티를 호출하는 방식으로 프로젝트에 대한 태스크를 보여 줄 수 있다. 추가로, 태스크가 존재할 경우에만 `div` 태그가 보이도록 `cout()`로 분기를 적용함.

```php
<?php if($project->tasks->count()) : ?>
  <div>
  	<?php foreach ($project->tasks as $task) : ?>
    	<li> {{ $task->description }}</li>
    <?php endforeach ?>
  </div>
<?php endif ?>
```

