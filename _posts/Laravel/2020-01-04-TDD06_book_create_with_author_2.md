---
layout: post
title: Test Driven Laravel 06 - Book:create with author & Unit test (2)
category: Laravel
tags: [PHP, TDD, Laravel, coderstape]
comments: true
---

> Coder's tape의 Test Driven Laravel [강의](https://www.youtube.com/playlist?list=PLpzy7FIRqpGAbkfdxo1MwOS9xjG3O3z1y) 를 듣고 정리한 포스팅 입니다.

#### 1. Returned to Feature Test

Feature test로 돌아와, `a_new_author_is_automatically_added`  테스트를 돌리면 또 다른 에러가 발생함.

```bash
table books has no column named author
```



이는 엔드포인트를 author로 hit하고 있기 때문임. 컨트롤러 수정

```php
public function validateRequest()
{
  return request()->validate([
    'title'  => 'required',
    // author -> author_id로 변경
    'author_id' => 'required',
  ]);
}
```



Author 모델 내에서 작성한 메소드를 Book 모델으로 이동시킨 후, 메소드명을 수정(Author->AuthorId). 그리고 attirbutes의 key 또한 author -> author_id로 변경시킴. 이 메소드를 통해 string으로 들어온 $author을 Author 객체로 변환시킴.

```php
public function setAuthorIdAttribute($author)
{
  $this->attributes['author_id'] = Author::firstOrCreate([
    'name' => $author,
  ]);
}
```



validation exception 에러 발생

```php
The given data was invalid.
```



라우트에 넘기는 데이터를 'author' 에서 'author_id'로 변경. 여기서 author_id 값에 왜 author의 name에 해당하는 string값을 넣어주는것에 대해 의문이 생길 수 있으나,  모델의 mutator 메소드 `setAuthorIdAttribute` 에서 이 값을 기반으로 author 객체를 만든 후, id값을 저장시키는 로직을 작성할 것임.

```php
/** @test */
public function a_new_author_is_automatically_added()
{
  $this->withoutExceptionHandling();

  $this->post('/books', [
    'title' => 'cool book title',
    // author -> author_id 로 변경
    'author_id' => 'harry' 
  ]);
```



에러 발생

```php
Failed asserting that '{"name":"harry","updated_at":"2019-12-16 05:37:37","created_at":"2019-12-16 05:37:37","id":1}' matches expected 1.
Expected :1
Actual   :{"name":"harry","updated_at":"2019-12-16 05:37:37","created_at":"2019-12-16 05:37:37","id":1}
```



이는 모델의 mutator 메소드에서 author_id 값으로 객체 전체의 정보를 저장시키고 있기 때문임. id값이 저장되도록 변경

```php
public function setAuthorIdAttribute($author)
{
  $this->attributes['author_id'] = (Author::firstOrCreate([
    'name' => $author,
  ]))->id;
}
```



이제 성공적으로 테스트가 동작함

```php
// BookManagementTest.php - test
/** @test */
public function a_new_author_is_automatically_added()
{
  $this->withoutExceptionHandling();

  $this->post('/books', [
    'title' => 'cool book title',
    'author_id' => 'harry'
  ]);

  $book = Book::first();
  $author = Author::first();

  $this->assertEquals($author->id, $book->author_id);
  $this->assertCount(1, Author::all());
}

// Book.php - Model
public function setAuthorIdAttribute($author)
{
  $this->attributes['author_id'] = (Author::firstOrCreate([
    'name' => $author,
  ]))->id;
}

// BooksController.php - controller
public function validateRequest()
{
  return request()->validate([
    'title'  => 'required',
    'author_id' => 'required',
  ]);
}

// create_books_table.php - migration
public function up()
{
  Schema::create('books', function (Blueprint $table) {
    $table->bigIncrements('id');
    $table->string('title');
    $table->unsignedBigInteger('author_id');
    $table->timestamps();
  });
}
```



#### 2. Test Code 수정

테스트를 전체로 돌려보면 여러 추가 에러가 발생함.

```php
Failed asserting that actual size 0 matches expected size 1.
 Desktop/TDD/library/tests/Feature/BookManagementTest.php:26
 

Session missing error: author
Failed asserting that false is true.
 Desktop/TDD/library/vendor/laravel/framework/src/Illuminate/Foundation/Testing/TestResponse.php:1028
 Desktop/TDD/library/tests/Feature/BookManagementTest.php:50
 

Error : Call to a member function path() on null
 Desktop/TDD/library/tests/Feature/BookManagementTest.php:65
 

Failed asserting that actual size 0 matches expected size 1.
 Desktop/TDD/library/tests/Feature/BookManagementTest.php:84
```





#### 3. Book - store 코드 수정

`BookManagementTest@a_book_can_be_added_to_the_library` (store 메소드)에서 에러가 발생

```php
public function a_book_can_be_added_to_the_library()
{
  $response = $this->post('/books', [
    'title' => 'cool book title',
    'author' => 'harry'
  ]);

  $book = Book::first();

  $this->assertCount(1, Book::all());
  $response->assertRedirect($book->path());
}
```



코드 수정에 앞서, 데이터를 넘기는 부분이 반복적으로 사용되고 있음. 이부분을 extract하여 새로운 메소드로 구현. 그리고 author이 아니라 author_id를 넘김.

```php
// before
$response = $this->post('/books', [
  'title' => 'cool book title',
  'author' => 'harry'
]);

// after
private function data()
{
  return [
    'title'  => 'cool book title',
    'author_id' => 'harry'
  ];
}
```

```php
/** @test */
public function a_book_can_be_added_to_the_library()
{
  $this->withoutExceptionhandling();

  $response = $this->post('/books', $this->data());

  $book = Book::first();

  $this->assertCount(1, book::all());
  $response->assertRedirect($book->path());
}
```



#### 4. Book - author validation 코드 수정

기존 코드

```php
public function an_author_is_required()
{
  $response = $this->post('/books', [
    'title' => 'cool book title',
    'author' => ''
  ]);

  $response->assertSessionHasErrors('author');
}
```



PHP에서 지원하는 `array_merge` 를 활용함(두 배열을 합쳐주는 기능을 함). `assertSessionHasErrors` 의 값도 author에서 author_id로 변경

```php
/** @test */
public function an_author_is_required()
{
  $response = $this->post('/books', array_merge($this->data(), ['author_id' => '']));

  $response->assertSessionHasErrors('author_id');
}
```



#### 5. Book - delete 코드 수정

엔드포인트를 때릴때 데이터를 넘기는 부분을 새롭게 구현한 메소드로 변경

```php
public function a_book_can_be_deleted()
{
  // $this->post('/books', [
  // 'title' => 'cool book title',
  // 'author' => 'harry'
  // ]);
  
  $this->post('/books', $this->data());

  $book = Book::first();
  $this->assertCount(1, Book::all());

  $response = $this->delete('/books/' . $book->id);

  $this->assertCount(0, Book::all());
  $response->assertRedirect('/books');
}
```



#### 6. Book - Update 코드 수정

엔드포인트를 때릴 때 넘기는 데이터는 새로운 메소드로 변경. 그리고 새로운 데이터로 업데이트할 때, author_id를 당연히 넘겨줘야한다. 마지막으로 assertEquals에는 author_id값을 비교하되, 1이 아니라 2를 비교함. (새로운 author을 만들었으므로 그 author의 id 값은 2가 될 것)

```php
public function a_book_can_be_updated()
{
  $this->post('/books', $this->data());

  $book = Book::first();

  $response = $this->patch($book->path(), [
    'title' => 'New Title',
    'author_id' => 'victor',
  ]);

  $this->assertEquals('New Title', Book::first()->title);
  $this->assertEquals(2, Book::first()->author_id);
  $response->assertRedirect($book->fresh()->path());
}
```

