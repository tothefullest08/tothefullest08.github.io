---
layout: post
title: Test Driven Laravel 04 - Author:create 테스트 코드 구현 / Carbon Instance
category: Laravel
tags: [PHP, TDD, Laravel, coderstape]
comments: true
---



#### 1. Author - Create 테스트 코드 작성 

테스트 파일 생성

```php
php artisan make:test AuthorManagementTest
```

author 생성하는 테스트 케이스 기본 코드 작성

```php
class AuthorManagementTest extends TestCase
{
    use RefreshDatabase;
    /** @test */
    public function an_author_can_be_created()
    {
        $this->withoutExceptionHandling();

        $this->post('/author', [
            'name' => 'Author Name',
            'dob' => '05/14/1998'
        ]);

        $this->assertCount(1, Author::all());
    }
}
```

라우트 & 컨트롤러 추가

```php
Route::post('/author', 'AuthorsController@store');

class AuthorsController extends Controller
{
    public function store()
    {
    }
}
```



모델 생성 & 마이그레이션 적용

```php
php artisan make:model Author -m
```



모델에 mass assignment 설정

```php
class Author extends Model
{
    protected $guarded = [];
}
```



마이그레이션 파일 수정

```php
Schema::create('authors', function (Blueprint $table) {
  $table->bigIncrements('id');
  $table->string('name');  // 추가
  $table->timestamp('dob'); // 추가
  $table->timestamps();
});
```



모델을 테스트 파일과컨트롤러 파일에 import 시킨 후, 기본 코드 작성

```php
use App\Author;

class AuthorsController extends Controller
{
    public function store()
    {
        Author::create(request()->only([
            'name', 'dob',
        ]));
    }
}
```



#### 2. Carbon Instance

Carbon은 PHP에서 개발한 날짜 관리 클래스로, Carbon을 이용하면 날짜를 쉽게 다룰 수 있음. 따라서 dob(Data of Birth)가 Carbon Instance로 오는지를 확인

```php
use Carbon\Carbon;

public function an_author_can_be_created()
{
  $this->withoutExceptionHandling();

  $this->post('/author', [
    'name' => 'Author Name',
    'dob' => '05/14/1998'
  ]);

  $author = Author::all();
  $this->assertCount(1, $author);
  $this->assertInstanceOf(Carbon::class, $author->first()->dob);
}
```



현재는 Carbon Instance가 아니라고 에러가 발생함.

```bash
Failed asserting that '05/14/1998' is an instance of class "Carbon\Carbon".
```



라라벨 엘로퀸트와 연동시켜, Model 클래스의 `$dates` 변수에 날짜/시간 컬럼을 명시하면 자동으로 Carbon 객에로 생성이 됨.

```php
class Author extends Model
{
    protected $guarded = [];

    protected $dates = ['dob'];
}
```



또다른 에러 발생... 포맷팅을 커스텀으로 설정해줘야함.

```bash
InvalidArgumentException : Unexpected data found.
Unexpected data found.
Data missing
```



포맷팅을 맞추기 위해서는 모델 클래스에서 메소드를 구현하되, 컨벤션 룰을 정확히따라줘야함. 인수명을  `$attribute`이라고 하는 것이 통상적이나, 우리는 이 상황에서 `$dob`가 들어오는 것을 알고있으므로 간단히 `$dob` 라고 명명.

- 컨벤션 : set + 컬럼명(첫글자 대문자로 시작) + Attribute

```php
class Author extends Model
{
    protected $guarded = [];

    protected $dates = ['dob'];

    public function setDobAttribute($dob)
    {
        $this->attributes['dob'] = Carbon::parse($dob);
    }
}
```



다른 포맷으로 들어오더라도, 포맷팅만 잘 비교해주면 상관없음

```php
$this->assertEquals('1998/14/05', $author->first()->dob->format('Y/d/m'));
```
