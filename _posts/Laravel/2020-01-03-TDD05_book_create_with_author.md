---
layout: post
title: Test Driven Laravel 05 - Book:create with author & Unit test (1)
category: Laravel
tags: [PHP, TDD, Laravel, coderstape]
comments: true
---

> Coder's tape의 Test Driven Laravel [강의](https://www.youtube.com/playlist?list=PLpzy7FIRqpGAbkfdxo1MwOS9xjG3O3z1y) 를 듣고 정리한 포스팅 입니다.

#### 1. GET STARTED

책(데이터)을 생성할 때, 작가를 자동적으로 생성하는 테스트 케이스를 작성해봄. 책 작성 시 작가를 자동적으로 생성하므로, AuthorManagementTest.php가 아닌 BookManagementTest.php에서 작성

```php
/** @test */
public function a_new_author_is_automatically_added()
{
  $this->post('/books', [
    'title' => 'cool book title',
    'author' => 'harry'
  ]);

  $book = Book::first();
  $author = Author::first();

  $this->assertEquals($author->id, $book->author_id);
  $this->assertCount(1, Author::all());
}
```



#### 2. `firstOrCreate`

현재 코드로는 'harry'라는 string이 입력되므로, author 객체가 생성되지 않음. 따라서 author(string)값이 생성되면, 이 값을 기반으로 (존재한다면) 매칭되는 객체를 가지고 오거나, 새로운 객체를 생성하는 메소드를 모델에서 구현 

- 라라벨 헬퍼 메소드 `firstOrCreate` 활용

```php
public function setAuthorAttribute($author)
{
  $this->attributes['author_id'] = Author::firstOrCreate([
    'name' => $author,
  ]);
}
```



하지만 여전히 에러가 발생함. 빈 객체의 id값을 갖고오는 이유는 author을 생성할 때 필요한 dob값을 지정해주지 않아 데이터베이스에 저장이 되지 않고 있기 때문임. 따라서 `setAuthorAttribute($author)` 메소드도 에러가 발생해야함.

```bash
ErrorException: trying to get property 'id' of non-object
```



에러 메세지를 확인하기 위해 `withoutExceptionHandling` 메소드를 찍어보아도 동일한 에러메세지가 뜸.

```php
/** @test */
public function a_new_author_is_automatically_added()
{
  $this->withoutExceptionHandling();
```



#### 3. Unit Test - only_name_is_required_to_create_an_author

 현재 feature test를 진행하고 있음. Feature test를 라우트를 통해 진행되나, 어떤 에러가 발생하는지 디테일하게 확인을 못하고 있는 상황임 (when you are testing too high up in terms of layers, it occurs). 이 때 "drop down level"을 이용하며, unit test를 사용.

unit test는 lowest level로 클래스와 객체들과 직접적으로 상호작용이 가능함. Test\Unit 내 AuthorTest.php 생성 후 기본 코드 작성

```php
namespace Tests\Unit;

use App\Author;
use Illuminate\Foundation\Testing\RefreshDatabase;
use Tests\TestCase;

class AuthorTest extends TestCase
{
    use RefreshDatabase;

    /** @test */
    public function only_name_is_required_to_create_an_author()
    {
        Author::firstOrCreate([
            'name' => 'John Doe',
        ]);

        $this->assertCount(1, Author::all());
    }
}
```

이후, 테스트를 진행하면 드디어, NULL constraint violation 에러가 발생함. Feature에서 진행한 테스트의 결과로는 구체적인 에러 메세지를 확인할 수 없었음. (never be able to reach down lower level) . 따라서 unit에서 테스트를 진행한 것임.

date of birth 값을 추가하지 않았으므로, 데이터베이스 저장에 실패함. 따라서 이 필드가 null값이 가능하도록 마이그레이션 파일 수정

```php
public function up()
{
  Schema::create('authors', function (Blueprint $table) {
    $table->bigIncrements('id');
    $table->string('name');
    // nullable
    $table->timestamp('dob')->nullable();
    $table->timestamps();
  });
}
```



위의 경우는 왜 발생했는지를 직관적으로 알 수 있으므로 개발을 진행하면서 직접 마이그레이션 내 코드를 수정하여 에러를 제거 시킬 수 있지만 이는 TDD의 step(아래) 대로 진행하지 않은 것임.

1. we start with green test
2. we change the test
3. we make it fail
4. we change the code on a fail test to turn it into green



#### 4. Unit Test - an_author_id_is_recorded

이후 다시 feature test로 돌아오면 여전히 에러가 발생하는데, 다음 단계로 `$book` 이 `author_id`를 갖고 있지 않다는 것을 알고 있음.

```php
/** @test */
public function a_new_author_is_automatically_added()
{
  $this->withoutExceptionHandling();

  $this->post('/books', [
    'title' => 'cool book title',
    'author' => 'harry'
  ]);

  $book = Book::first();
  $author = Author::first();

  $this->assertEquals($author->id, $book->author_id);
  $this->assertCount(1, Author::all());
}
```



다시 drop down a level 한 후 에러를 수정시켜 봄.

```bash
php artisan make:test BookTest --unit
```



현재 우리는 모델을 통해 데이터를 at the very root level of our project에서 생성하고 있음. 엔드포인트를 통해서 테스트를 진행하는 것이 아님. Unit test라고 말 할 수 있음 (we are reaching in and really grabbing the stuff in the database and manipulating)

```php
namespace Tests\Unit;

use App\Book;
use Illuminate\Foundation\Testing\RefreshDatabase;
use Tests\TestCase;

class BookTest extends TestCase
{
  use RefreshDatabase;
  
    /** @test */
    public function an_author_id_is_recorded()
    {
        Book::create([
            'title' => 'cool title',
            'author_id' => 1,
        ]);

        $this->assertCount(1, Book::all());
    }
}
```



에러 발생

```bash
table books has no column named author_id 
```



book migration 수정 & author_id 컬럼 추가

```php
public function up()
{
  Schema::create('books', function (Blueprint $table) {
    $table->bigIncrements('id');
    $table->string('title');
    $table->string('author');
    $table->unsignedBigInteger('author_id');
    $table->timestamps();
  });
}
```



Constraint Violation 에러 발생

```bash
NOT NULL constraint failed: books.author 
```



book migration 파일에서 더이상 사용하지 않는 author 칼럼 제거한 후 유닛 테스트를 동작시키면 제대로 동작함.

```php
public function up()
{
  Schema::create('books', function (Blueprint $table) {
    $table->bigIncrements('id');
    $table->string('title');
    // $table->string('author');
    $table->unsignedBigInteger('author_id');
    $table->timestamps();
  });
}
```

