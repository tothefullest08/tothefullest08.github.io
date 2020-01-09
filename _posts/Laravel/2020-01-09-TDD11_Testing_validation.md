---
layout: post
title: Test Driven Laravel 11 - 최종 리팩토링
category: Laravel
tags: [PHP, TDD, Laravel, coderstape]
comments: true
---

> Coder's tape의 Test Driven Laravel [강의](https://www.youtube.com/playlist?list=PLpzy7FIRqpGAbkfdxo1MwOS9xjG3O3z1y) 를 듣고 정리한 포스팅 입니다.

#### 1. Feature Test: Author - create(store) 테스트 코드 리팩토링

기존 코드

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

        $author = Author::all();

        $this->assertCount(1, $author);
        $this->assertInstanceOf(Carbon::class, $author->first()->dob);
        $this->assertEquals('1998/14/05', $author->first()->dob->format('Y/d/m'));
    }
}

```



엔드포인트를 복수형으로 변경 (/author -> /authors ) & 라우트도 그에 맞게 수정

```php
$this->post('/authors', [
  'name' => 'Author Name',
  'dob' => '05/14/1998'
]);

Route::post('/authors', 'AuthorsController@store');
```



데이터와 함께 엔드포인트에 도달하는 로직을 추출하여 새로운 메소드로 분리시킴

```php
public function an_author_can_be_created()
{
  $this->withoutExceptionHandling();

  $this->post('/authors', $this->data());

  $author = Author::all();

  $this->assertCount(1, $author);
  $this->assertInstanceOf(Carbon::class, $author->first()->dob);
  $this->assertEquals('1998/14/05', $author->first()->dob->format('Y/d/m'));
}

private function data()
{
  return [
    'name' => 'Author Name',
    'dob'  => '05/14/1998'
  ];
}
```



#### 2. Feature Test - Author 유효성 검증(작가 이름 필요)

기본 코드 작성

```php
/** @test */
public function a_name_is_required()
{
  $response = $this->post('/authors', array_merge($this->data(), ['name' => '']));

  $response->assertSessionHasErrors('name');
}
```



에러발생. `AuthorsController@store `는 현재 유효성 검증을 하고 있지 않음. 따라서 유효성 검증 코드 작성한 후, 테스트를 돌리면 정상적으로 테스트가 동작함.

```php
// Session is missing expected key [errors].

class AuthorsController extends Controller
{
    public function store()
    {
        $data = request()->validate([
           'name' => 'required',
           'dob' => '',
        ]);

        Author::create($data);
    }
}
```



#### 3. Feature Test - Author 유효성 검증(Date Of Birth 필요)

```php
/** @test */
public function a_dob_is_required()
{
  $response = $this->post('/authors', array_merge($this->data(), ['dob' => '']));

  $response->assertSessionHasErrors('dob');
}
```



동일한 에러가 발생함. 테스트가 동작할 수 있도로 유효성 검증을 통해 해당 필드가 필수가 되도록 수정

```php
// Session is missing expected key [errors].

class AuthorsController extends Controller
{
    public function store()
    {
        $data = request()->validate([
           'name' => 'required',
           'dob' => 'required',
        ]);

        Author::create($data);
    }
}
```



#### 4. 유효성 검증 코드 리팩토링

`BooksController@validateRequest` 메소드를 통해 유효성 검증하는 부분을 추출하여 따로 뺀 것처럼, `AuthorController@validateRequest`  메소드를 동일한 방식으로 구현하여 리팩토링을 진행

```php
class AuthorsController extends Controller
{
    public function store()
    {
        Author::create($this->validateRequest());
    }

    private function validateRequest()
    {
        return request()->validate([
            'name' => 'required',
            'dob'  => 'required',
        ]);
    }
}
```

