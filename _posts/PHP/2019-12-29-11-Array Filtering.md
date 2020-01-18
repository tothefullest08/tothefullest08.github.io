---
layout: post
title: The PHP Practitioner 11 - Array Filtering
category: PHP
tags: [Laracast, PHP]
comments: true
---

> Laracasts - The PHP Practitioner [강의](https://laracasts.com/series/php-for-beginners)를 듣고 정리한 포스팅 입니다.
#### 0. GET STARTED

기본 코드는 다음과 같음

```php
class Post
{
    public $title;
    public $author;
    public $published;

    public function __construct($title, $author, $published)
    {
        $this->title = $title;
        $this->author = $author;
        $this->published = $published;
    }
}

$posts = [
    new Post('My first post', 'harry', true),
    new Post('My second post', 'harry',  true),
    new Post('My third post', 'mat', true),
    new Post('My fourth post', 'simon', false),
];
```

> PHP에서 array를 활용할 수 있는 메서드들이 매우 많음. 그 중 대표적인 것이 아래.

#### 1. `array-filter`

filter down the array - `array_filter(array, closure function)`

```php
$unpublishedPosts = array_filter($posts, function ($post) {
    // return $post -> published === false;
    return ! $post -> published;

});

$publishedPosts = array_filter($posts, function ($post) {
    return $post -> published;

});

var_dump($unpublishedPosts);
var_dump($publishedPosts);
```



#### 2. `array_map`

map over the array(change the status). It is useful when you need to modify what gets refunded

-  `array_map(closure function, array)`

```php
// map over the array -> change the status - array_map(closure, array)
// when you need to modify what gets returned.
$modified = array_map(function($post) {
    $post->published = true;
    return $post;
}, $posts);

var_dump($modified);

// 응용 1. (array) -> return each element as array ( this case, obj -> array)
$modified = array_map(function($post){
    return (array) $post;

}, $posts);

var_dump($modified);

// 응용 2.
$modified = array_map(function($post){
    return ['title' => $post->title];
}, $posts);

var_dump($modified);

// 응용 3. map 메서드는 foreach 로도 표현 가능
foreach ($posts as $post) {
    $post->published = true;
}

var_dump($posts);
```



#### 3. `array_column`

extract specific column from the array(object)

-  `array_column(array, column)`

```php
// 특정 컬럼만을 추출할 때 유용. 오브젝트, 배열 모두에 사용 가능
$titles = array_column($posts, 'title');

// 응용) 3번째 인자로, key 값에 해당하는 컬럼을 지정할 수 있음.
// 'my first post' : "harry"
$titles = array_column($posts, 'author', 'title');

var_dump($titles);
```