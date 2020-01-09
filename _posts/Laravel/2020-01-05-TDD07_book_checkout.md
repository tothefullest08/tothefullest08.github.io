---
layout: post
title: Test Driven Laravel 07 - Book Checkout 테스트 코드 구현 (unit test)
category: Laravel
tags: [PHP, TDD, Laravel, coderstape]
comments: true
---

> Coder's tape의 Test Driven Laravel [강의](https://www.youtube.com/playlist?list=PLpzy7FIRqpGAbkfdxo1MwOS9xjG3O3z1y) 를 듣고 정리한 포스팅 입니다.

#### 1. GET STARTED

도서관에서 책을 대여하는 checkout 구현하기 위해, 테스트 파일 생성

```php
php artisan make:test BookReservationsTest
```



가장 기본적으로, 책을 체크아웃하면 시간이 찍히는 로직 구현. 기준이 되는 모델을 유저로 해도 되며, 책으로 해도 됨. 후자를 선택해서 진행

```php
public function a_book_can_be_checked_out()
{
  // $book->checkout($user);
  // $user->checkout($book);
}
```



#### 2. Factory

`factory()` 를 활용하여 Book, User 모델을 토대로 가상의 데이터를 생성하여 각각의 변수에 저장. `Reservation` 모델은 구현 전이나, user_id, book_id, check_out_at 등의 컬럼을 갖고 있을 예정.

```php
use App\Book;
use App\User;
use Tests\TestCase;
use Illuminate\Foundation\Testing\RefreshDatabase;

class BookReservationsTest extends TestCase
{
    use RefreshDatabase;
    /** @test */
    public function a_book_can_be_checked_out()
    {
        $book = factory(Book::class)->create();
        $user = factory(User::class)->create();

        $book->checkout($user);

        $this->assertCount(1, Reservation::all());
        $this->assertEquals($user->id, Reservation::first()->user_id);
        $this->assertEquals($book->id, Reservation::first()->book_id);
        $this->assertEquals(now(), Reservation::first()->checked_out_at);
    }
}
```



이 때, 위의 테스트는 feature test/unit test 인지를 살펴보면, 우리가 체크아웃하는 특정한 책을 토대로 테스트를 하기 때문에 unit test라고 할 수 있음.



#### 3. Feature Test vs Unit Test

Feature Test 중 하나인 `an_author_can_be_created` 를 살펴보면, 

1. The test actually hits a particular endpoint with some data.
2. we are basically simulating the same exact thing that a user would do if they went through our browser, interacting with our website sent in some data.
3. application did what it had to do

Unit Test인 `a_book_can_be_checked_out`  의 경우, "we are at no point doing anything that a user could do directly". 따라서 해당 테스트의 파일 위치를 tests\Feature에서 tests\Unit으로 변경

```php
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
```



#### 4. Factory 생성

테스트를 돌려보면 에러가 발생. 따라서 모델 Book에 대한 팩토리를 만듦.

```bash
InvalidArgumentException : Unable to locate factory with name [default] [App\Book].

php artisan make:factory BookFactory -m Book
```



마이그레이션에 명시된 컬럼명들을 토대로 코드를 작성. 이때 author_id에 대해서는 2가지 방법으로 구현이 가능함.

```php
/** @var \Illuminate\Database\Eloquent\Factory $factory */

use App\Book;
use App\Author;
use Faker\Generator as Faker;

$factory->define(Book::class, function (Faker $faker) {
    return [
        'title' => $faker->sentence,
      	'author_id' => factory(Author::class)->create() // 방법 1
      	'author_id' => factory(Author::class) // 방법 2
    ];
});
```



 후자의 방법의 경우, 

1. 테스트 코드에서 book을 생성할 때 값을 전달시킬 경우, 라라벨은 Author 객체를 생성하지 않음. 
2. 만약에 값을 전달시키지 않을 경우, 라라벨은 Author 객체를 생성시킴.

그러나 전자의 경우, book을 생성할 때마다 author 또한 생성 됨. 만약에 수동으로 author_id를 오버로딩하는 경우에도 author_id가 생성됨. 따라서 후자의 방법을 사용

```php
/** @test */
public function a_book_can_be_checked_out()
{
  $book = factory(Book::class)->create([
    'author_id' => 123
  ]);
```



Author 팩토리가 생성되지 않았다는 에러가 발생하므로 팩토리 생성

```bash
InvalidArgumentException : Unable to locate factory with name [default] [App\Author].
```

```php
/** @var \Illuminate\Database\Eloquent\Factory $factory */

use App\Author;
use Faker\Generator as Faker;

$factory->define(Author::class, function (Faker $faker) {
    return [
        'name' => $faker->name,
        'dob' => now()->subYears(10),
    ];
});
```



#### 3. Return to check out 테스트 코드 구현

Book 모델 내 checkout 메소드 구현

```bash
BadMethodCallException : Call to undefined method App\Book::checkout()
```

```php
class Book extends Model
{
    // 코드 중략
    public function checkout()
    {
    }
}
```



Reservation 모델 생성 & 테스트 파일에 모델 import

```bash
Error : Class 'Tests\Unit\Reservation' not found
php artisan make:model Reservation -m
```



Book 모델에 checkout 메소드를 구현. `a_book_can_be_checked_out`  에서 `Book@checkout` 를 호출하여, Reservation 객체를 생성시킨 후, user_id, checked_out_at의 값을 적용시킴. 이때 book_id도 필요함 (관계 설정이 요구 됨)

```bash
Failed asserting that actual size 0 matches expected size 1.
```

```php
// BookReservatinsTest.php
public function a_book_can_be_checked_out()
{
  $book = factory(Book::class)->create();
  $user = factory(User::class)->create();

  $book->checkout($user);

// Book.php
public function checkout($user) 
{
  Reservation::create([
    'user_id' => $user->id,
    'checked_out_at' => now(),
  ]);
}
```



Reservation은 Book과 관계설정(Book: Reservation = 1: N) 이 적용되므로, 코드를 아래와 같이 수정

```php
public function checkout($user)
{
  $this->reservations()->create([
    'user_id' => $user->id,
    'checked_out_at' => now(),
  ]);
}
```



Book 모델에 관계설정 메소드 구현

```bash
BadMethodCallException : Call to undefined method App\Book::reservations() 
```

```php
public function reservations()
{
  return $this->hasMany(Reservation::class);
}
```



Reservation 모델에 mass assignment 코드 적용

```bash
Illuminate\Database\Eloquent\MassAssignmentException : Add [user_id] to fillable property to allow mass assignment on [App\Reservation].
```

```php
namespace App;

use Illuminate\Database\Eloquent\Model;

class Reservation extends Model
{
    protected $guarded = [];
}
```



Reservation 모델 마이그레이션에 누락된 컬럼들을 추가

```bash
Illuminate\Database\QueryException : SQLSTATE[HY000]: General error: 1 table reservations has no column named user_id
```

```php
public function up()
{
  Schema::create('reservations', function (Blueprint $table) {
    $table->bigIncrements('id');
    $table->unsignedBigInteger('user_id');
    $table->unsignedBigInteger('book_id');
    $table->timestamp('checked_out_at');
    $table->timestamps();
  });
}
```
