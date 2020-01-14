---
layout: post
title: SOLID Principles in PHP 03 - Liskov substitution Principle (LSP)
category: PHP
tags: [Laracast, PHP, OOP, SOLID]
comments: true
---



#### 1. GET STARTED

> "컴퓨터 프로그램에서 자료형 S가 자료형 T의 하위형이라면 필요한 프로그램의 속성(정확성, 수행하는 업무 등)의 변경 없이 자료형 T의 객체를 자료형 S의 객체로 치환 할 수 있어야 한다"
>
> - Let q(x) be a property provable about objects x of type T.
>
> - Then q(y) should be provable for objects y of type S where S is a subtype of T
>

서브 타입(자식 클래스) 은 언제나 자신의 기반타입(부모 클래스) 으로 교체 할 수 있어야 한다. Derived(sub) classes must be substitutable for their base classes

-  `dosomething`  함수가 클래스 A를 인자로 받아 동작을 한다면 A의 서브 클래스인 B가 넣어도 문제 없이 그대로 동작해야만 함. 

```php
class A {
    public function fire() {}
}

class B extends A {
    public function fire() {}
}

function doSomething(A $obj)
{
    // do something with it
}
```



#### 2. Pre-condition of the subclass is too great

비디오 플레이어에 대한 `VideoPlayer` 클래스가 있으며 플레이를 시키는 `play` 메소드가 있다고 가정하자. 그리고 .avi 파일을 플레이하는 경우를 고려했을 때, 가장 먼저 떠오르는 생각은 `VideoPlayer` 를 상속받아 서브클래스를 만드는 것일 것이다. 그 다음 코드 구현은 다음과 같다.

- `play` 메소드를 오버라이드 
- 확장자를 체크하는 분기문 작성. 확장자가 avi가 아닐 경우 Exception을 던짐

그러나 이 코드는 LSP를 위반하는 코드이다. 이유는 서브 클래스의 전제조건이 더 클 수없기 때문이다.(pre-conditions of the subclass can't be greater). 부모 클래스인 `VideoPlayer`를 `AviVideoPlayer` 로 교체한다고 했을 경우, 반환값이 달라 질 수 있기 때문에(확장자에 따라) LSP를 준수하지 못하게 된다

이러한 경우, 우리가 떠올릴 수 있는 방법은 contract(interface)를 활용하는 것이다. 그러나 이 방법은 input만을 검증하며 output은 검증하지 못한다.

```php
class VideoPlayer {
    public function play($file)
    {
        // play the video
    }
}

class AviVideoPlayer extends VideoPlayer {
    public function play($file)
    {
        if(pathinfo($file, PATHINFO_EXTENSION) !== 'avi')
        {
            throw new Exception; // violates the LSP
        }
    }
```



`LessonRepositoryInterface` 에는 모든 데이터를 갖고오는 `getAll` 메소드가 있음. 이에 따라, 파일시스템/데이터베이스로부터 데이터를 갖고와 인터페이스를 구현하는 코드가 있다고 하자. 이 경우 각각의 클래스 내 `getAll` 메소드는 다음과 같이 다른값을 반환한다. 이 경우 역시 LSP를 준수하지 못하는 코드가 됨.

- `FileLessonRepository@getAll`  - 배열을 반환
- `DbLessonRepository@getAll`  - collection 을 반환

```php
interface LessonRepositoryInterface {
    public function getAll();
}

class FileLessonRepository implements LessonRepositoryInterface {

    public function getAll()
    {
        // return through filesystem
        return [];
    }
}

class DbLessonRepository implements LessonRepositoryInterface {

    public function getAll()
    {
        // return via eloquent model
        return Lesson::all();
    }
}
```



위 코드를 LSP를 준수하는 방향으로 변경하고 싶을 경우 인터페이스의 메소드위의 힌트를 넣을 수 있음. PHP가 강제로 이 주석을 준수해라고 하진 않지만 일종의 약속으로, 반드시 array를 반환하게 코드를 바꿔야 함.

```php
interface LessonRepositoryInterface {
    /**
     * Fetch all records
     *
     * @return array
     */
    public function getAll();
}

class DbLessonRepository implements LessonRepositoryInterface {

    public function getAll()
    {
        return Lesson::all()->toArray();
    }
}
```



인터페이스를 인자로 받아 동작하는 함수를 예시로 들어 보자. 만약에 인터페이스를 구현하는 클래스들의 반환값이 다르다면, 값의 데이터 타입을 체크하여 분기문을 작성할 것이다. 이는 반환값이 다를 것을 예상한다는 것으로 LSP를 지키지 않았다는 의미이다. 

```php
function foo(LessonRepositoryInterface $lesson)
{
    $lessons = $lesson->getAll();
}
```



#### 3. Summary

- Signature must match
- Preconditions can't be greater
- Post conditions at least equal to
- Exception types must match