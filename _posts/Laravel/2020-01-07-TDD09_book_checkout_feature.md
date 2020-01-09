---
layout: post
title: Test Driven Laravel 09 - Book Checkout 테스트 코드 구현 (Feature test)
category: Laravel
tags: [PHP, TDD, Laravel, coderstape]
comments: true
---

> Coder's tape의 Test Driven Laravel [강의](https://www.youtube.com/playlist?list=PLpzy7FIRqpGAbkfdxo1MwOS9xjG3O3z1y) 를 듣고 정리한 포스팅 입니다.

#### 1. Checkout 테스트 코드 작성

Unit Test에서 구현한 book checkout, checkin을 Feature Test에서 구현

```bash
php artisan make:test BookCheckoutTest
```



Unit Test의 로직을 기본적으로 따라감

```php
// Unit Test
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
```



Feature Test의 경우 라우트의 엔드포인트를 통해 인증된 유저가 들어올 것이라고 가정을 하고 진행하므로 접근방법이 조금 다름.  book_id를 함께 엔드포인트로 넘김. 만약 제대로 동작할 경우, 나머지 코드는 Unit Test의 코드를 동일하게 사용

-  `actingAs` 헬퍼 메소드는 특정사용자를 현재 사용자로 인증하는 단순한 방법을 제공함.

```php
class BookCheckoutTest extends TestCase
{
    use RefreshDatabase;

    /** @test */
    public function a_book_can_be_checked_out_by_a_signed_in_user()
    {
        $book = factory(Book::class)->create();
        $user = factory(User::class)->create();

        $this->actingAs($user)->post('/checkout/' . $book->id);

        $this->assertCount(1, Reservation::all());
        $this->assertEquals($user->id, Reservation::first()->user_id);
        $this->assertEquals($book->id, Reservation::first()->book_id);
        $this->assertEquals(now(), Reservation::first()->checked_out_at);
    }
}
```



에러가 발생. 상세하게 어떠한 에러인지를 알기 위해 `withoutExceptionHandling` 사용

```bash
Failed asserting that actual size 0 matches expected size 1.
```



라우트 설정

```php
// Exception\NotFoundHttpException : POST http://localhost/checkout/1
Route::post('/checkout/{book}', 'CheckoutBookController@store');
```



`CheckoutBookController` 컨트롤러 생성 후 store 메소드 구현

```php
// BindingResolutionException : Target class [App\Http\Controllers\CheckoutBookController] does not exist.
class CheckoutBookController extends Controller
{
    public function store()
    {
    }
}
```



Unit Test를 통해서 바로 다음 어떠한 로직/코드를 짜야하는지 쉽게 알 수 있음. Route Model Binding을 통해 Book 객체를 저장시키며, `auth()->user()` 를 통해 인증된 유저를 인자로 받아 `Book@checkout` 를 실행시킴.

```php
class CheckoutBookController extends Controller
{
    public function store(Book $book)
    {
        $book->checkout(auth()->user());
    }
}
```



나머지 코드는 Unit Test 때와 그대로 동일하게 작성

```php
public function a_book_can_be_checked_out_by_a_signed_in_user()
{
  $book = factory(Book::class)->create();
  $user = factory(User::class)->create();

  $this->actingAs($user)->post('/checkout/' . $book->id);

  $this->assertCount(1, Reservation::all());
  $this->assertEquals($user->id, Reservation::first()->user_id);
  $this->assertEquals($book->id, Reservation::first()->book_id);
  $this->assertEquals(now(), Reservation::first()->checked_out_at);
}
```



#### 2. Checkout 테스트 케이스 확장 - 인증된 유저만 체크아웃 가능

인증되지 않은 유저가 체크아웃을 진행할 경우, 로그인 페이지로 리다이렉트하는 테스트 케이스 작성.  Reservation은 이루어지지 않았으므로 객체의 수는 0개여야 함.

```php
/** @test */
public function only_signed_in_users_can_checkout_a_book()
{
  $book = factory(Book::class)->create();

  $this->post('/checkout/' . $book->id)
    ->assertRedirect('/login');

  $this->assertCount(0, Reservation::all());
}
```



테스트를 돌려보면 500 에러가 발생함. `withoutExceptionHandling` 를 통해 출력을 해보면 user_id가 존재하지 않음을 알수 있음. (`Book@checkout`에서 reservation 객체를 생성할 때 user_id가 필요하나, 누락되어있음)

```php
// Response status code [500] is not a redirect status code.

// Book.php
public function checkout($user)
{
  $this->reservations()->create([
    'user_id' => $user->id,
    'checked_out_at' => now(),
  ]);
}
```



생성자 메소드를 통해 라라벨의 미들웨어중 하나인 `Authenticate` 를 동작시킴.  `Authenticate`  파일을 살펴보면 인증에 실패했을 경우, 로그인 페이지로 리다이렉트 시키는 로직이 구현되어있음.

```php
// Http\Middleware\Authenticate.php
class Authenticate extends Middleware
{
    protected function redirectTo($request)
    {
        if (! $request->expectsJson()) {
            return route('login');
        }
    }
}
```

```php
class CheckoutBookController extends Controller
{
    public function __construct()
    {
        $this->middleware('auth');
    }
    public function store(Book $book)
    {
        $book->checkout(auth()->user());
    }
}
```



테스트를 동작시키면 500에러가 발생함. 현재 Laravel authentication을 이용하고 있지만,  그에 따른 라우트 설정을 해주지 않았음(로그인, 로그아웃 등의 라우트가 존재하지 않음). 

```php
//  if  laravel version below 6.0
php artisan make:auth 
// else
composer require laravel/ui --dev 
php artisan ui vue --auth
```

이후 테스트를 돌려보면 성공적으로 작동하는 것을 알 수 있음.



#### 3. Checkout 테스트 케이스 확장 - 책이 존재할 때만 체크아웃 가능

존재하지 않는 book_id를 하드코딩하여 입력한 후, `assertStatus` 가 404(not found)가 뜨도록 코드 작성

```php
/** @test */
public function only_real_books_can_be_checked_out()
{
  $this->actingAs(factory(User::class)->create())
    ->post('/checkout/123')
    ->assertStatus(404);

  $this->assertCount(0, Reservation::all());
}
```

