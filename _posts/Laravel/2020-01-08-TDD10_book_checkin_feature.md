---
layout: post
title: Test Driven Laravel 10 - Book Checkin 테스트 코드 구현 (Feature test)
category: Laravel
tags: [PHP, TDD, Laravel, coderstape]
comments: true
---

#### 1. Checkin 테스트 코드 작성

기본적인 코드는 `a_book_can_be_checked_out_by_a_signed_in_user` 메소드를 토대로 만드나, `reservation` 객체를 생성하려면  `Book@checkout`이 선행되어야함. 

```php
/** @test */
public function a_book_can_be_checked_in_by_a_signed_in_user()
{
  $book = factory(Book::class)->create();
  $user = factory(User::class)->create();
  
  // Reservation 객체를 생성하기 위해 우선 체크아웃 로직 동작시킴
  $this->actingAs($user)->post('/checkout/' . $book->id); 

  $this->actingAs($user)->post('/checkin/' . $book->id);

  $this->assertCount(1, Reservation::all());
  $this->assertEquals($user->id, Reservation::first()->user_id);
  $this->assertEquals($book->id, Reservation::first()->book_id);
  $this->assertEquals(now(), Reservation::first()->checked_in_at);
}
```



아래 코드에서 에러가 발생함.

``` php
// null does not match expected type "object".
$this->assertEquals(now(), Reservation::first()->checked_in_at);
```



`withoutExceptionHandling`으로 살펴보면 라우트 설정이 되지 않은 것이 확인 가능. 라우트를 생성한 후, `CheckinBookController` 컨트롤러를 생성  & store 메소드를 구현함.

```bash
NotFoundHttpException : POST http://localhost/checkin/1
```

```bash
Route::post('/checkin/{book}', 'CheckinBookController@store');

class CheckinBookController extends Controller
{
    public function store()
    {
    }
}
```



정확한 상세 정보를 확인할 수 없는 에러가 발생하나, Unit test를 통해 어떤 로직을 구현해야하는지를 알 수 있음.

- `$book->checkin($user);` 

```bash
null does not match expected type "object".
```

```php
use App\Book;

class CheckinBookController extends Controller
{
    public function store(Book $book)
    {
        $book->checkin(auth()->user());
    }
}
```



나머지 코드는 Unit Test 때와 그대로 동일하게 작성

```php
public function a_book_can_be_checked_in_by_a_signed_in_user()
{
  $this->withoutExceptionHandling();

  $book = factory(Book::class)->create();
  $user = factory(User::class)->create();
  $this->actingAs($user)->post('/checkout/' . $book->id);

  $this->actingAs($user)->post('/checkin/' . $book->id);

  $this->assertCount(1, Reservation::all());
  $this->assertEquals($user->id, Reservation::first()->user_id);
  $this->assertEquals($book->id, Reservation::first()->book_id);
  $this->assertEquals(now(), Reservation::first()->checked_in_at);
}
```



#### 2. Checkin 테스트 케이스 확장 - 인증된 유저만 체크아웃 가능

"인증된 유저만 체크아웃 가능" - `only_signed_in_users_can_checkout_a_book()` 를 기반으로 코드 작성

```php
public function only_signed_in_users_can_checkin_a_book()
{
  $this->withoutExceptionHandling();

  $book = factory(Book::class)->create();

  $this->post('/checkout/' . $book->id)->assertRedirect('/login');
  $this->post('/checkin/' . $book->id)->assertRedirect('/login');

  $this->assertCount(0, Reservation::all());
}
```



테스트를 실행시켜보면 500에러(unauthorized) 발생. 기존의 코드는 로그인이 되어있지 않은 경우에 로그인 엔드포인트로 리다이렉트시키는 코드인데, 지금의 테스트 코드는 체크인을 테스트하는 것임. 그러므로 우선, 로그인은 정상적으로 동작해야함. `actingAs` 메소드와 `factory`  메소드를 활용하여 로그인이 동작하도록 코드 수정

```php
$this->post('/checkout/' . $book->id)->assertRedirect('/login');
```

```php
public function only_signed_in_users_can_checkin_a_book()
{
  $book = factory(Book::class)->create();
  $this->actingAs(factory(User::class)->create())
    ->post('/checkout/' . $book->id);

  $this->post('/checkin/' . $book->id)
    ->assertRedirect('/login');

  $this->assertCount(0, Reservation::all());
}
```



CheckinBookController의 생성자 메소드로 auth 미들웨어를 추가시킴

```php
class CheckinBookController extends Controller
{
    public function __construct()
    {
        $this->middleware('auth');
    }
```



에러가 발생.  로그인 코드를 통해 로그인 상태가 유지되고 있어서 200 코드를 반환시키고 있음. 따라서, 로그 아웃 코드를 추가로 삽입해야함.

```php
//Response status code [200] is not a redirect status code.
//Failed asserting that false is true.

// 문제가 되는 부분
$this->post('/checkin/' . $book->id)->assertRedirect('/login');
```

```php
use Illuminate\Support\Facades\Auth;

/** @test */
public function only_signed_in_users_can_checkin_a_book()
{
  $book = factory(Book::class)->create();

  $this->actingAs(factory(User::class)->create())
    ->post('/checkout/' . $book->id);

  Auth::logout();

  $this->post('/checkin/' . $book->id)
    ->assertRedirect('/login');

  $this->assertCount(0, Reservation::all());
}
```



Reservation 객체가 생성되었으므로 assertCount의 첫번째 인수에는 1을 넣어주며, 체크인이 실제로 되지 않았으므로 그 값이 null이라는 것을 확인시켜주는 코드를 추가로 작성함

```php
$this->assertCount(1, Reservation::all());
$this->assertNull(Reservation::first()->checked_in_at);
```



최종적으로 완성된 코드는 아래와 같음.

```php
public function only_signed_in_users_can_checkin_a_book()
{
  $book = factory(Book::class)->create();

  $this->actingAs(factory(User::class)->create())
    ->post('/checkout/' . $book->id);

  Auth::logout();

  $this->post('/checkin/' . $book->id)
    ->assertRedirect('/login');

  $this->assertCount(1, Reservation::all());
  $this->assertNull(Reservation::first()->checked_in_at);
}
```



#### 3. Checkin 테스트 케이스 확장 - 체크아웃 되지 않은 책을 체크인

체크아웃되지 않은 책을 체크인할 경우 404에러를 반환하는 것이 일반적임. 

-  checkin 테스트 코드를 기반으로 작성하되, `Book@checkout` 을 동작시키는 코드는 삭제시킴.
-  Reservation 객체는 생성되지 않으므로 0으로 만들며 나머지 코드는 삭제시킴
-  그리고 /checkin/ 엔드포인트에 대하여 404를 반환하는 assert 코드 구현

```php
 public function a_404_is_thrown_if_a_book_is_not_checked_out_first()
{
  $book = factory(Book::class)->create();
  $user = factory(User::class)->create();

  $this->actingAs($user)
    ->post('/checkin/' . $book->id)
    ->assertStatus(404);

  $this->assertCount(0, Reservation::all());
}
```



500 에러를 발생시킴. `withoutExceptionHandling` 를 통해 에러 디테일 확인. 에러가 발생하는 `CheckinBookController@store` 수정

```php
public function store(Book $book)
{
  try {
    $book->checkin(auth()->user());
  } catch (\Exception $e) {
    return response([], 404);
  }
}
```

