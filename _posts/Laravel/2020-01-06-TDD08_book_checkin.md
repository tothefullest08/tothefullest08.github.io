---
layout: post
title: Test Driven Laravel 08 - Book Checkin 테스트 코드 구현 / 테스트 케이스 확장 (unit test)
category: Laravel
tags: [PHP, TDD, Laravel, coderstape]
comments: true
---

> Coder's tape의 Test Driven Laravel [강의](https://www.youtube.com/playlist?list=PLpzy7FIRqpGAbkfdxo1MwOS9xjG3O3z1y) 를 듣고 정리한 포스팅 입니다.

#### 1. Checkin 테스트 코드 구현

테스트 기본 코드 작성. `Book@checkin`  를 통해 checkin을 동작시키며, checked_in_at 시간을 적용함.

```php
public function a_book_can_be_returned()
{
  $book = factory(Book::class)->create();
  $user = factory(User::class)->create();
  $book->checkout($user);

  $book->checkin($user);

  $this->assertCount(1, Reservation::all());
  $this->assertEquals($user->id, Reservation::first()->user_id);
  $this->assertEquals($book->id, Reservation::first()->book_id);
  $this->assertNotNull(Reservation::first()->checked_in_at);
  $this->assertEquals(now(), Reservation::first()->checked_in_at);
}
```



`Book@checkin`  구현. 체크인을 진행하려면 checked_out_at은 null이 아니어야하며, checked_in_at은 null값인 객체가 있다는 것이 선행되어야 함.

```bash
BadMethodCallException : Call to undefined method App\Book::checkin()
```

```php
public function checkin($user)
{
  $reservation = $this->reservations()->where('user_id', $user->id)
    ->whereNotNull('checked_out_at')
    ->whereNull('checked_in_at')
    ->first();

  $reservation->update([
    'checked_in_at' => now()
  ]);
}
```



체크인 컬럼이 존재하지 않아 에러가 발생. Reservation 마이그레이션에 checked_in_at 컬럼 추가. `book@checkout` 로직이 동작할 시에는 체크아웃시간이 null값이어야 하므로 `nullable()` 적용

```bash
Error : Call to a member function update() on null
```

```php
public function up()
{
  Schema::create('reservations', function (Blueprint $table) {
    $table->bigIncrements('id');
    $table->unsignedBigInteger('user_id');
    $table->unsignedBigInteger('book_id');
    $table->timestamp('checked_out_at');
    $table->timestamp('checked_in_at')->nullable();
    $table->timestamps();
  });
}
```



#### 2. 테스트 케이스 확장 - 체크아웃 2회

체크인을 하자마자 다시 체크아웃을 했을 경우의 상태는 다음과 같음

- Reservation 객체는 2개 (체크아웃 2번)
- Reservation의 id가 2인 객체와 비교
- Checked_in_at의 값은 null
- Reservation 객체의 checked_out_at에는 현재 시간이 저장 

```php
/** @test */
public function a_user_can_check_out_a_book_twice()
{
  $book = factory(Book::class)->create();
  $user = factory(User::class)->create();
  $book->checkout($user);
  $book->checkin($user);
  $book->checkout($user);

  $this->assertCount(2, Reservation::all());
  $this->assertEquals($user->id, Reservation::find(2)->user_id);
  $this->assertEquals($book->id, Reservation::find(2)->book_id);
  $this->assertNull(Reservation::find(2)->checked_in_at);
  $this->assertEquals(now(), Reservation::find(2)->checked_out_at);
}
```



마찬가지로, 이후 다시 체크인을 했을 경우 비교하는 코드도 추가 가능

```php
/** @test */
public function a_user_can_check_out_a_book_twice()
{
  $book = factory(Book::class)->create();
  $user = factory(User::class)->create();
  $book->checkout($user);
  $book->checkin($user);
  $book->checkout($user);

  $this->assertCount(2, Reservation::all());
  $this->assertEquals($user->id, Reservation::find(2)->user_id);
  $this->assertEquals($book->id, Reservation::find(2)->book_id);
  $this->assertNull(Reservation::find(2)->checked_in_at);
  $this->assertEquals(now(), Reservation::find(2)->checked_out_at);

  $book->checkin($user);
  $this->assertCount(2, Reservation::all());
  $this->assertEquals($user->id, Reservation::find(2)->user_id);
  $this->assertEquals($book->id, Reservation::find(2)->book_id);
  $this->assertNotNull(Reservation::find(2)->checked_in_at);
  $this->assertEquals(now(), Reservation::find(2)->checked_in_at);
}
```



#### 3. 테스트 케이스 확장 - 체크아웃 X

체크아웃을 하지 않고 체크인을 한 경우에 대한 테스트 기본 코드를 작성

```php
use Exception;

/** @test */
public function if_not_checked_out_exception_is_thrown()
{
  $this->expectException(Exception::class);
  $book = factory(Book::class)->create();
  $user = factory(User::class)->create();

  $book->checkin($user);
}
```



테스트를 돌리면 에러가 발생. `Book@checkout` 을 통해 Reservation 객체가 생성되는데 바로 이를 거치지 않고, `Book@checkin` 을 호출하면 객체가 존재하지 않기 때문임. 따라서 객체가 존재하지 않을 경우 exception을 던지는 코드 작성

```bash
Failed asserting that exception of type "Error" matches expected exception "Exception". Message was: "Call to a member function update() on null" at
```

```php
use Exception;

public function checkin($user)
{
  $reservation = $this->reservations()->where('user_id', $user->id)
    ->whereNotNull('checked_out_at')
    ->whereNull('checked_in_at')
    ->first();

  if (is_null($reservation)) {
    throw new Exception();
  }

  $reservation->update([
    'checked_in_at' => now()
  ]);
}
```