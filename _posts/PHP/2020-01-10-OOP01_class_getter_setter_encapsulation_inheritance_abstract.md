---
layout: post
title: Object-Oriented Bootcamp 01 - Class / Encapsulation / Inheritance / Abstract class
category: PHP
tags: [Laracast, PHP, OOP, Architecture]
comments: true
---

> Laracasts - Object-Oridented Bootcamp [강의](https://laracasts.com/series/object-oriented-bootcamp-in-php)를 듣고 정리한 포스팅 입니다.

#### 1. Classes

PHP의 클래스는 기본적으로 2가지로 구성되어 있음

-  Property: 멤버변수. 클래스의 변수를 정의
  - `public` : 클래스 외부에서도 프로퍼티에 접근할 수 있게 함.
- Method: 클래스의 특정 함수(동작)을 정의
  - `__construct` : 생성자 메서드로, 클래스 인스턴스가 생성될때 자동으로 실행됨.

클래스 인스턴스를 생성할때는 `new` 를 사용하며, 프로퍼티 & 메서드를 호출할 때는 `->` 를 사용함. 또한, 클래스 내의 `$this` 는 클래스의 인스턴스를 의미함.

```php
<?php

class Task {
    // public: outside of this class, anyone can access to the property
    public $description;

    public $completed = false;

    // method
    // __construct: immediately run when instantiating the class
    public function __construct($description)
    {
        $this->description = $description;
    }

    public function complete()
    {
        $this->completed = true;
    }
}

$task = new Task('Learn OOP');
$task->complete();
var_dump($task->description);
var_dump($task->completed);
```



#### 2. Getters and Setters

이름과 나이에 대한 프로퍼티 & 메서드를 구현한 후, 클래스의 인스턴스를 생성하여 호출하면 인스턴스의 이름과 나이를 쉽게 알 수 있음. 그렇다면 getter과 setter의 역할을 무엇일까?

```php
<?php

class Person {

    public $name;

    public $age;

    public function __construct($name)
    {
        $this->name = $name;
    }
}

$harry = new Person('harry lee');
$harry->age = 30;

var_dump($harry); // name: harry, age: 30

$harry->age++;
var_dump($harry); // name: harry, age: 31
```



getter과 setter를 쓰는 이유는 (little litte bit of ) protection 과 security 때문.

예를 들어 18살 미만인 사람은 허용하지 않는 어플리케이션을 만든다고 하자. 문제는 우리가 age라는 프로퍼티를 사용하므로, 나이와 관계없이 인스턴스의 프로퍼티에 값을 저장할 수 있음(나이 15, 심지어 -5와 같이...) 이처럼 프로퍼티로 바로 접근하는 경우 어떠한 Protection이 존재하지 않음. 이러한 맥락으로 setter & getter를 사용함.

> Setters & Getters are behaviors associated with setting or getting particular property.  In this case, the bahavior is that if you are younger than 18, then that's not allowed.
>
> Convention: `set/get + value`

```php
public function setAge($age)
{
  if ($age < 18)
  {
    throw new Exception("Person is not old enough");
  }

  $this->age = $age;
}

$harry = new Person('harry lee');
$harry->setAge(17); // throw Exception
```



getter도 같은 방식으로 메서드 구현 가능

```php
public function getAge()
{
  return $this->age;
}

$harry = new Person('harry lee');
var_dump($harry->getAge());
```



에러를 던지는 로직을 작성하였지만, 여전히 `setAge` 를 통해 18살 미만일 경우  `setAge`  를 거치지 않고 프로퍼티에 바로 접근하여 인스턴스에 값을 저장시킬 수 있음. 이러한 이유로 `Encapulation` 을 사용함.



#### 3. Encapulation

프로퍼티를 정의할 때 `public` 외에도 `private` 와 `protected` 가 있음. 
`private`  메서드를 호출할 경우 에러 메세지가 발생함 (me only exclusively access from within the class)

> PHP Fatal error:  Uncaught Error: Call to private method LightSwitch::connect() from context '' in /Users/....."

`protected` 도 동일하게 에러 메세지를 발생시키지만, 클래스를 extend 하여 서브클래스에서 해당 메서드를 사용 할 수 있음. 따라서, 이전 강의에서 보았듯이 `setter` 를 거치지 않고 바로 프로퍼티에 접근하는 것을 막기 위해서는 프로퍼티를 `private` 으로 변경하면 됨.



#### 3-1. 객체가 private/protected 멤버를 갖게 하세요

- `public` 메소드와 프로퍼티는 변경에 가장 취약합니다. 그 이유는 어떤 외부 코드가 쉽게 의존할 수 있고, 어떤 코드가 의존하고 있는지 제어할 수 없기 때문입니다. **클래스의 수정은 클래스의 모든 사용자에게 위험합니다.**
- `protected` 제어자는 `public` 만큼이나 위험합니다. 자식 클래스 범위내에서 사용할 수 있기 때문입니다. 이는 public과 protected의 차이점은 접근 매커니즘에만 있다는 것을 의미하나, 캡슐화 보증은 동일하게 유지됩니다. **클래스의 수정은 모든 하위 클래스에 위험합니다.**
- `private` 제어자는 코드가 **단일 클래스의 경계에서만 수정하는 것이 위험함** 을 보증합니다(변경하는 것이 안전하며 [젠가 효과](http://www.urbandictionary.com/define.php?term=Jengaphobia&defid=2494196)를 갖지 않을 것 입니다.).

그러므로 `private` 을 기본으로 사용하고 외부 클래스에 대한 접근 권한을 제공해야 할 때 `public/protected`를 사용하세요. 더 많은 정보를 원하면 이 주제에 대해서 [Fabien Potencier](https://github.com/fabpot)가 작성한 [블로그 포스트](http://fabien.potencier.org/pragmatism-over-theory-protected-vs-private.html)를 읽어볼 수 있습니다.

> 출처: [Clean Code PHP 한글판](https://github.com/yujineeee/clean-code-php)



#### 4. Inheritance

서브 클래스는 상속을 통해 부모 클래스의 메서드, 프로퍼티에 접근이 가능함. 서브 클래스에서 다른 클래스를 상속할 때는 `extends` 를 사용함.

```php
class Mother {

    public function getEyeCount()
    {
        return 2;
    }
}

class Child extends Mother {
}

(new Child)->getEyeCount(); // 2
```



라라벨에서는 `Eloqeunt`  클래스를 상속받음으로써, 라라벨 ORM 문법을 사용할 수 있게 됨.

```php
class Post extends Eloquent {
}

$post->save();
$post->update();
```



클래스를 상속할 때는, 부모 클래스의 functionality/behavior를 오버라이딩 할 수 있음

```php
class Shape {
    protected $length = 4;

    public function getArea()
    {
        return pow($this->length, 2);
    }
}

class Triangle extends Shape{
    protected $base = 4;
    protected $height = 7;

    // override the method of parent class
    public function getArea()
    {
        return .5 * $this->base * $this->height;
    }
}
```



`Shape@getArea` 는 부모 클래스에 구현되어있으나, 사실상 로직이 사각형에서만 사용이 가능함(삼각형, 오각형 등에 사용 불가). 따라서 해당 프로퍼티와 매서드를 서브 클래스인 `Sqaure` 으로 옮기는 것이 더 적합함.

그렇다면 비어있는 `Shape` 클래스를 사용하는 이유 및 Benefit은 무엇일까? 

1. will there ever be any attributes or behaviors that would be shared across every shape? 만약 이 질문의 답이 "그렇다" 라면, functionality를 부모 클래스에 두는 것이 맞음.
2. we can use as a **sole contract**

```php
class Shape {
}

class Square extends Shape {
    protected $length = 4;

    public function getArea()
    {
        return pow($this->length, 2);
    }
}
```



Circle 이라는 새로운 클래스를 생성하여 Shape를 상속 받은 후 `getArea` 메서드를 호출할 경우, 에러가 발생함. "언제 어디서든 Shape을 상속받은 서브클래스에서 `getArea` 메서드를 사용하고 싶다" 라는 contract가 포함되어야함. 이를 위해서는 2가지 방법이 있음.

1. Abstract Class
2.  Interface

```php
class Circle extends Shape {
}

echo (new Circle)->getArea();

// Fatal error: Uncaught Error: Call to undefined method Circle::getArea()
```



#### 4-1. Abstract class

기본 클래스인 `Shape` 의 인스턴스를 생성하는 것은 어떠한 에러도 발생시키지 않음.  `new Shape;`   또한, 일반적인 shape이 없으므로 항상 서브 클래스를 instantiate 하자고 결정내릴 수 있음 (그리고 기본 클래스는 유지하고 싶지 않음). 

이러한 경우 `abstract` 를 사용. 추상화 클래스를 사용하면, 이 클래스의 인스턴스를 만드는 것이 불가능해짐 (에러 발생). 추상화 클래스를 사용하면 여전히 상속은 가능하면서, 기본 클래스의 인스턴스를 만드는것을 막을 수 있음. 

```php
abstract class Shape {
}
```



부모클래스인 `Shape` 에서 `color` 라는 프로퍼티의 값을 부여하는 생성자 메서드를 만든 후, 서브클래스에서 어떠한 파라미터도 넣지 않은 채 인스턴스를 만들어 출력을 해보면, 에러가 발생하는 것을 알 수 있음.

```php
abstract class Shape {
  
    protected $color;

    public function __construct($color)
    {
        $this->color = $color;
    }
}

echo new Square;
// Fatal error: Uncaught ArgumentCountError: Too few arguments to function Shape::
```



`color` 프로퍼티에 default 값(red)을 주는 경우, 그 값을 서브 클래스들이 그대로 사용할 수 있게 됨. 이때 getter를 사용하여 프로퍼티에 접근.  

`getColor` 는 모든 서브 클래스들이 공유하는 shared behavior임. 이후, 각 서브 클래스의 인스턴스를 만들면서 getter 메서드를 호출하는 경우, color를 지정하지 않으면 red가 출력되거나, 인자로 받은 색상이 출력된다. 

```php 
abstract class Shape {
  
    protected $color;

    public function __construct($color = 'red')
    {
        $this->color = $color;
    }

    public function getColor()
    {
        return $this->color;
    }
}

echo (new Square())->getColor() ; // red
echo (new Square('green'))->getColor() ; // green
```



서브클래스인 `Circle` 이 `getArea()` 메서드를 호출하면 에러가 발생함.  만약 각각의 서브클래스가 고유의 메서드를 정의해야 한다면, 추상 메서드를 호출하면 됨.  추상 메서드는 특별히 body ( `{}` )가 필요하지 않음.

```php
abstract class Shape {
    protected $color;

    public function __construct($color = 'red')
    {
        $this->color = $color;
    }

    public function getColor()
    {
        return $this->color;
    }
		
    // 추상화 메서드 정의
    abstract protected function getArea();
}

class Circle extends Shape {

}

echo (new Circle('green'))->getArea() ; 
```



최종적으로 서브 클래스 `Circle` 에서 `getArea`  메서드를 구현

```php
class Circle extends Shape {
    protected $radius = 5;

    public function getArea()
    {
        return pi() * pow($this->radius, 2);
    }
}

$circle = new Circle;
echo $circle->getArea(); // 78.539..
```



**For the purpose of extract common behavior, we use abstract class. **

추상 클래스는 상속을 강제하기 위한 것임. 부모 클래스에는 메서드의 시그니처만 정의해놓고 그 메소드의 실제 동작 방법은 메소드를 상속 받은 하위 클래스의 책임으로 위임하고 있음. 

추상메소드를 정의하면 서브클래스는 반드시 그 메소드를 구현해야함.



#### 5. Messages 101

사람들은 직업을 갖고 있으며 비즈니스를 위해 일을 함. 비즈니스는 사람들을 고용함. 고용된 사람들은 스태프로써 일을 함
- `Person` , `Business` , `Staff` 라는 클래스로 구성 가능

비즈니스 클래스는 사람을 고용해야 하므로 `hire` 메소드 구현. 이때 당연히 사람을 고용하므로 $person을 인자로 넣을 수 있음. 이 개념을 확장하여 PHP에서 지원하는 Type hinting 을 사용하여 오브젝트를 인자로 넣을 수 있음 `Person $person`

```php
class Person {

    protected $name;

    public function _construct($name)
    {
        $this->name = $name;
    }

}

class Business {

    protected $staff;

    public function __construct(Staff $staff)
    {
        $this->staff = $staff;
    }

    public function hire(Person $person)
    {
        $this->staff->add($person);
    }
}

class Staff {
  
    protected $members = [];

    public function add(Person $person)
    {
        $this->members[] = $person;
    }
}

$harry = new Person('harry lee');
$staff = new Staff([$harry]);
$laracasts = new Business($staff);
```



메세지를 보냄으로써 클래스간 상호작용을 하는 것이 가능함. 스태프를 추가하고싶을때는 스태프에 add 메세지를 보내며, 모든 스태프의 리스트를 보고 싶을 때는 스태프에 members 메세지를 보냄

```php
// Business Class
public function hire(Person $person)
{
  $this->staff->add($person);
}

public function getStaffMembers()
{
  return $this->staff->members();
}

// Staff Class
public function add(Person $person)
{
  $this->members[] = $person;
}

public function Members()
{
  return $this->members;
}
```



최종 결과 코드

```php
<?php

class Person {

    protected $name;

    public function __construct($name)
    {
        $this->name = $name;
    }
}

class Business {

    protected $staff;

    public function __construct(Staff $staff)
    {
        $this->staff = $staff;
    }

    public function hire(Person $person)
    {
        $this->staff->add($person);
    }

    public function getStaffMembers()
    {
        return $this->staff->members();
    }
}

class Staff {
    
    protected $members = [];

    public function __construct($members = [])
    {
        $this->members = $members;
    }

    public function add(Person $person)
    {
        $this->members[] = $person;
    }

    public function Members()
    {
        return $this->members;
    }
}

$harry = new Person('harry lee');
$staff = new Staff([$harry]);
$laracasts = new Business($staff);

$laracasts->hire(new Person('Ron Wizlie'));

//$laracasts->hire($harry);

var_dump($staff);
var_dump($laracasts->getStaffMembers());
```
