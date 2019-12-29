---
layout: post
title: Laravel 5.7 From Scratch 09 - Form Action Consideration
category: Laravel
tags: [PHP, Laracast, Laravel]
comments: true
---

#### 1. Form Action Considerations

태스크를 완료 시킬 수 있는 폼태그 양식을 만들고 그 내용을 업데이트하는 로직을 구성하도록 할 수 있음.

- 태스크 완료 유무를 체크박스로 표시
- submit은 자바스크립트 코드로 구현(`onChange = "this.form.submit()`)
- 삼항 연산자로 checked 속성 적용 유무 확인
- 폼 태그 메서드 및 액션, csrf 적용

```php
<?php foreach ($project->tasks as $task) : ?>
  <div>
    <form method="POST" action="/tasks/{{ $task->id }}">
    @method('PATCH')
    @csrf

      <label class="checkbox" form="completed">
        <input type="checkbox" name="completed" onChange="this.form.submit()"
          {{ $task->completed ? 'checked' : '' }}>
        {{ $task->description }}
      </label>
    </form>
  </div>
<?php endforeach ?>
```



#### 2. 관계 & MVC 설정

web.php에서 업데이트 로직에 대한 라우터 설정을 하며 ProjectTasksController.php에서 로직 구현. 또한 모델(Task.php)에서 mass assignment에 대한 설정도 진행

- `return back();` : 바로 이전 페이지로 리턴
- `request()->has('completed')` : 해당 키의 값이 존재하는지를 확인함.

```php
// web.php
Route:patch('/tasks/{task}', 'ProjectTasksController@update')

// ProjectTasksController.php
class ProjectTasksController extends Controller
{
    public function update(Task $task)
    {
        $task->update([
            'completed' => request()->has('completed')
        ]);

        return back();
    }
}

// Task.php
class Task extends Model
{
    protected $guarded = [];
    public function project()
    {
        return $this->belongsTo(Project::class);
    }
}
```



#### 3. Create New Project Tasks

새로운 태스크를 작성하는 코드를 show.blade.php에서 작성 & web.php에서 store에 해당하는 라우트 설정.

```php
// add a new task form
<form method="POST" action="/projects/{{ $project->id }}/tasks">
    @csrf

    <div>
        <label class="label" for="description">New Task</label>
        <input type="text" class="input" name="description" placeholder="New Task">
    </div>
    <div>
        <button type="submit">Add Task</button>
    </div>
</form>


// option 1
Route::post('/projects/{project}/tasks', 'ProjectTasksController@store');
// option 2: hidden input으로 어떤 프로젝트에 대한 것인지를 함께 넘겨줘야함.
Route::patch('/tasks', 'ProjectTasksController@store');
```



ProjectTasksController@store구현.

첫번째 옵션 처럼 태스크를 생성해도 되나, 해당 로직을 실행하는 메서드를 프로젝트 모델에서 작성하여 클린코드를 지향할 수 도 있음(두번째 옵션 권장). 프로젝트 모델에서도 두가지 방법으로 구현을 할 수 있음.

```php
// ProjectTasksController.php
use App\Project;

class ProjectTasksController extends Controller

public function store(Project $project)
  {

    //option 1
    Task::create([
      'project_id' => $project->id,
      'description' => request('description')
    ]);

    //option 2
    $project->AddTask(request('description'))

      return back();
  }

// Project.php
    public function addTask($task)
    {
				// option 1
        return Task::create([
            'project_id' => $this->id,
            'description' => $description
        ]);

      	// option 2 - 유효성 검증을 통해 array가 반환될 예정이므로
      	// compact()를 사용하지 않고 바로 $task를 넘김.
      	$this->tasks()->create($task);
    }
```



#### 4. 유효성 검증

서버 사이드에서 유효성 검증을 구현하며, 에러 메세지도 뷰에서 보이도록 하자. 에러메세지의 코드는 사실상 동일하므로 해당 코드가 담긴 별도의 파일을 작성한 후, `@include`로 상속 받아옴.

```php
// ProjectTasksController.php

public function store(Project $project)
{
  $attributes = request()->validate(['description' => 'required']);

  $project->AddTask($attributes);

  return back();
}

// errors.blade.php
<?php if ($errors->any()) : ?>
<div>
    <ul>
        <?php foreach($errors->all() as $error) : ?>
        <li> {{ $error }}</li>
        <?php endforeach ?>
    </ul>
</div>
<?php endif; ?>

// show.blade.php
@include ('errors')
```



#### 5. Better Encapsulation

> Encapsulation: Hide internal state and values inside a class

컨트롤러에서 직접 코드를 작성해도 되는데 굳이 모델에서 새로운 메서드를 만드는 이유?

much more readable (`$project->AddTask` vs `$project->tasks()->create(request)` ). Project model can encapsule, what it means to addTask. All we need to know the outside is that "hey, here is the attribute. I want to addTask" then Project model can be fully responsible for what it means to actually do that.

ProjectTasksController.php - `update` 메서드 코드에 encapsulation을 적용 + Task.php에 `complete` 메서드 정의

```php
// ProjectTasksController
public function update(Task $task)
{

  // before
  $task->update([
    'completed' => request()->has('completed')
  ]);

  // after
  $task->complete(request()->has('completed'));

  return back();
}

// Task.php
public function complete($completed=true)
{
  // $this->update(['completed'=>$completed]);
  $this->update(compact('completed'));
}
```



위 코드를 한번 더 발전시켜서, 완료되지 않았을 때에 대한 메서드도 모델에서 구현 가능

```php
public function incomplete()
{
  $this->complete(false);
}
```



모델(Task.php)에서 구현한 `complete` 와 `incomplete`을 활용하여 ProjectTasksController.php의 update 메서드를 더욱 클린하게(encapulation을 적용하여) 구현할 수 있음. 구현 방법은 아래와 같이 분기문, 삼항 연산자 등을 활용 하면 됨.

```php
public function update(Task $task)
{
  //option 1
  if (request()->has('completed')) {
    $task->complete();
  } else {
    $task->incomplete();
  }

  // option 2
  request()->has('completed') ? $task->complete() : $task->incomplete();

  // option 3
  $method = request()->has('completed') ? 'complete' : 'incomplete';
  $task->$method();

  return back();
}
```



#### 6. When in Doubt

이전 강의를 통해 태스크 완료 유무의 로직을 ProjectdTasksController@store에서 구현하였음. 현재 구현된 핵심 코드는 아래와 같음. 현재 코드로도 큰 문제는 없으나, 여전히 request()에 종속되어 분기문을 통해 Task 모델의 메서드를 호출하고 있음. 하기 로직은 테스크 완료 유무 로직이 동작하는 별도의 컨트롤러를 생성하여 발전 시킬 수 있음.

```php
$method = request()->has('completed') ? 'complete' : 'incomplete';
$task->$method();
```



CompletedTasksController.php 생성 후,  태스크 완료, 태스크 완료 해제에 대한 메서드 `store` , `destroy` 구현. 기존에 ProjectTasksController.php에 구현되었던 `update` 메서드는 삭제할 수 있음. 아래와 같이 컨트롤러를 구분함으로써, 동일한 엔드포인트에 대해 다른 메서드를 호출하도록 routes를 설정할 수 있으며, request()에도 종속되지 않으며 분기문도 사용하지 않는 매우 클린한 코드가 완성됨.

```php
// CompletedTasksController.php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use App\Task; // Task 메서드 사용을 위해 export

class CompletedTasksController extends Controller
{
    public function store(Task $task)
    {
        $task->complete();

        return back();
    }

    public function destroy(Task $task)
    {
        $task->incomplete();

        return back();
    }
}

// web.php
Route::post('/completed-tasks/{task}', 'CompletedTasksController@store');
Route::delete('/completed-tasks/{task}', 'CompletedTasksController@destroy');
```



show.blade.php - 폼태그의 action 속성 값(url 경로)를 새롭게 지정한 endpoint로 변경시킴. 이때, 동일한 엔드포인트를 기반으로  "테스크 완료" & "테스크 완료 해제"를 적용시키기 위해 `@method` 값을 변경해주는 분기문 작성

```php
<?php if($project->tasks->count()) : ?>
  <div>
    <?php foreach ($project->tasks as $task) : ?>
      <div>
        <form method="POST" action="/tasks/{{ $task->id }}">
          @if ($task->completed)
          	@method('DELETE')
          @endif
          @csrf

        <label class="checkbox" form="completed">
          <input type="checkbox" name="completed" onChange="this.form.submit()"
              {{ $task->completed ? 'checked' : '' }}>
            {{ $task->description }}
    		</label>
      </form>
    </div>
    <?php endforeach ?>
  </div>
<?php endif ?>
```
