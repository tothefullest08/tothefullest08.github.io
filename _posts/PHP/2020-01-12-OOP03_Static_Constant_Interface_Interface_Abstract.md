---
layout: post
title: Object-Oriented Bootcamp 03 - Static, Constant / Interface / Interface vs Abstract
category: PHP
tags: [Laracast, PHP, OOP]
comments: true
---



#### 1. Statics

간단하게 들어온 숫자들의 합을 구하는 메소드를 구현해보도록 하자.

```php
// option 1
class Math {
    public function add()
    {
        return array_sum(func_get_args());
    }
}

// option 2. applicable for recent version of PHP
class Math {
    public static function add(...$nums)
    {
        return array_sum($nums);
    }
}

$math = new Math;

var_dump($math->add(1, 2, 3, 4));
```



현재, `add` 메소드는 다른 클래스에서 호출하는 등 dynamic할 필요가 없음 (input을 받아서 계산 후 output을 반환하기만 하면 됨). 이처럼 dynamic 하지 않게 메소드를 구현하고 싶을 경우, 함수를 정의할 때  `static`  을 추가하면 됨. 

메서드를 호출할때는 `클래스명::메소드명` 으로 호출.  스태틱 메소드는 인스턴스를 생성하지 않고 바로 어디서든 사용이 가능함. 이러한 관점에서 볼 때, 스태틱 메소드는 `Global function ` 이라고 할 수 도 있음. 

만약 스태틱 메소드가 다른 클래스를 반환/호출하는 경우는 유지/테스트 하기에 매우 어려우므로 권장되는 방법이 아님.

```php
class Math {
    public static function add(...$nums)
    {
        return array_sum($nums);
    }
}

echo Math::add(1,2,3);
```



#### 2. Static with Example

`Person` 이라는 클래스에 프로퍼티 `$age` 를 스태틱하게 1로 정의해보도록 하자. 이론적으로 문제 없이 동작하나, 객체 & 클래스의 관점에서 볼 때는 적절하지 못함. "사람" 이라는 클래스를 만들었다면 모든 사람들이 동일한 나이를 Share(=static) 한다는 것은 적절하지 않음.

```php
class Person {
    public static $age = 1;
}

echo Person::$age;
```



또한, 아래의 예시에서도 문제가 생길 수 있음. 스태틱 프로퍼티인 `age` 를 1씩 증가시키는 `haveBirthday` 메소드를 구현하고, `harry` 와 `ron` 의 나이를 개별적으로 증가시켜보자. 

- 우리는 `harry` 는 해당 메소드를 2번 호출하여 3이라는 값을 갖게 하고
- `ron` 은 한번만 호출하여 2이라는 값을 갖게 하고 싶음. 
- 그러나 스태틱 프로퍼티는 특정 오브젝트에만 할당되는 것이 아니라 글로벌로 공유됨.
- 따라서  `ron` 의 나이는 4라는 값을 갖게 된다 → **break encapsulation**

```php
class Person {
  
    public static $age = 1;

    public function haveBirthday()
    {
        static::$age += 1;
    }
}

$harry = new Person;
$harry->haveBirthday(); // 2
$harry->haveBirthday(); // 3

echo $harry::$age; // 3

$ron = new Person;
$ron->haveBirthday(); // expected 2 but returns 4

echo $ron::$age; // 4
```



#### 3. Constants

스태틱 프로퍼티는 어떠한 경우에 사용 될 수 있을까? 인스턴스마다 개별적인 값을 가지지 않고 모든 인스턴스에게 동일하게 적용되는 값들에게 스태틱 프로퍼티를 적용할 수 있을 것이다 (예를 들어, 세율).

아래의 예시의 경우,  `tax` 는 `public` 으로 정의된 스태틱 프로퍼티이므로 값을 변경할 수 있다. 만약에 반드시 고정된 값을 가지며 값을 변하게 하기 위해서는  `priave` 으로 변경할 수 도 있지만, 다른 방법으로 `const` (상수)를 사용할 수 있음.

```php
class BankAccount {
    // public static $tax = .09;
  	// private static $tax = .09;
  	const TAX = 0.9;

}

// echo BankAccount::$tax = 1.5;
echo BankAccount::TAX;
```

마찬가지로, 스태틱 메소드 또한 전역으로 사용하며 인스턴스 생성 없이 바로 사용하기 위해 쓴다.



#### 4. Interfaces

**"Think of interface as contract" ** 인터페이스 안에서 실제 로직을 작성하지 않음. 인터페이스는 어떠한 term이 적용되어야 하는지를 작성하는 개념. 

아래 예시에서 `Animal`  이라는 인터페이스 내에 "모든 동물은 의사소통을 한다" 라는 term을 적용시키기 위해 `communication` 함수를 작성하였음. 인터페이스는 실제 로직을 작성하지 않으므로 함수의 body( `{}` )를 입력하지 않음.

```php
interface Animal {
    public function communicate();
}
```



we want to make sure that any implementation that we have will at this contract. on the other words, I want any type of animal to offer communicate method. 

`Animal` 인터페이스를 implements 함으로써, 인터페이스(계약서)에 작성된 메서드 `communicate` 를 반드시 사용해야함.

```php
interface Animal {
    public function communicate();
}

class Dog implements Animal {
    public function communicate()
    {
        return 'bark';
    }
}

class Cat implements Animal {
    public function communicate()
    {
        return 'meow';
    }
}
```



#### 5-1. Interfaces vs Abstract Classes (1)

인터페이스와 추상 클래스의 차이는 무엇일까? 로그 기능을 구현한다고 가정해보자. 로그에는 파일/데이터베이스/온라인 서비스 등 다양한 영역에 접근할 수 있음. 이에 따라 기본적인 클래스를 구현하면 다음과 같음. 그리고 로그 데이터에 접근 & 출력에 대한 로직을 구현하는 컨트롤러도 작성해보자.

```php
class LogToFile {
    public function execute($message)
    {
        var_dump('log the message to a file: '. $message);
    }
}

class LogToDatabase {
    public function execute($message)
    {
        var_dump('log the message to a database: '. $message);
    }
}

class UsersController {

    protected $logger;

  	// LogToFile 클래스를 하드코딩으로 파라미터에 입력
    public function __construct(LogToFile $logger)
    {
        $this->logger = $logger;
    }

    public function show()
    {
        $user = 'harrylee';

        // log this information
        $this->logger->execute($user);
    }
}

$controller = new UsersController(new LogToFile);
$controller->show();
```



이제, 회사의 보스가 말하길 "파일"에는 더이상 로깅하지 않으니 "데이터베이스"에 로깅하는 코드로 변경하라, 고 새로운 지시를 내렸다고 가정해보자. 이제 문제는 `LogToFile`  이라는 specific implementation을 하드코딩하여 여러 군 데 사용했다는 것임. (과장해서, 여러 어플리케이션에 적용했다고 가정) => 코드를 찾아서 일일이 다 변경해줘야함.

> *"The problem was that we were too specific. on another word, the problem was that we assumed an implementation. we didn't think the possibilty that it might be changed"*
>
>  **“Coding to interfaces, not implementation(concretion)”**

이 말은 위의 예시와 정확히 일치한다. 위의 예시에서  `UserController` 클래스의 생성자 메소드를 작성할 때 우리는 implementation(concrete class)을 사용하였음. 그리고 이것을 다른 것으로 변경해야할 때 broke down 되었음.

if there are ever classess or tasks or you could imgine having multiple implementations (multiple different message excuting this task/behavior, then that is a sign that you need to create interface)

따라서 두개의 클래스를 인터페이스와 연결한 후, `UserController` 클래스의 생성자 메서드의 인자로 클래스가 아니라 인터페이스인 `Logger` 를 넘김.

```php
interface Logger {
    public function execute($message);
}

class LogToFile implements Logger {
    public function execute($message)
    {
        var_dump('log the message to a file: '. $message);
    }
} 

class LogToDatabase implements Logger {
    public function execute($message)
    {
        var_dump('log the message to a database: '. $message);
    }
}

class UsersController {

    protected $logger;

    public function __construct(Logger $logger)
    {
        $this->logger = $logger;
    }
```

인자로 인터페이스를 넘기는 것과 특정 클래스를 넘기는데는 커다란 차이가 있음.  

1. 클래스를 넘긴 경우, 유저 컨트롤러는 바깥 세상에 "기능이 동작하려면 이 특정 인스턴스가 필요해" 라고 말함.
2. 인터페이스를 넘긴 경우, 유저 컨트롤러는 "기능이 동작하려면 어떤 종류의 함수가 필요해. 정확히 어떤 클래스인지는 알 필요 없어. 너가 정해. 난 그냥 유저 컨트롤러를 동작할 때 어떤 functionality가 있기만 하면 돼!" 

이제 `LogToDatabase` 클래스로 일일이 하드하게 변경할 필요가 없으며 `UserController`  또한 그대로 둬도 됨. 그저 인스턴스를 생성할 때 `LogToDatabase` 를 호출하기만 하면 됨.

```php
$controller = new UsersController(new LogToFile);
$controller->show();

$controller = new UsersController(new LogToDatabase);
$controller->show();
```

인터페이스를 활용한 추가 예시

```php
interface Repository {
    public function save($data);
}

class MongoRepository implements Repository {
    public function save($data)
    {
    }
}

class FileRepository implements Repository {
    public function save($data)
    {
    }
}
```

```php
interface CanBeFiltered {
    public function filter();
}

class Favorited implements CanBeFiltered{
    public function filter()
    {
    }
}

class Unwatched implements CanBeFiltered{
    public function filter()
    {
    }
}

class Difficulty implements CanBeFiltered{
    public function filter()
    {
    }
}
```



#### 5-2. Interfaces vs Abstract Classes (2)

어플리케이션에서 Github으로 로그인하는 함수를 구현한다고 가정해보자. 다른 SNS(페이스북, 카카오톡) 등으로 로그인을 시도하기 전까지는 문제가 없음. 

```php
function login(GithubProvider, $provider)
{
    $provider->authorize();
}
```



위의 예시는 provider implementation을 하드코딩한 상태임. 만약 페이스북의 경우는? 혹은 카카오톡의 경우는? 분기문을 따로 작성하여 프로바이더의 종류에 따라 다르게 동작하도록 코드를 작성해야할까? 

이처럼 오브젝트의 타입을 체크해야하는 경우, 99%의 경우 you should be leveraging polymorphism(다형성). 다형성의 관점에서 접근하여, 우리는 특정한 클래스를 참조할 것이 아니라 인터페이스를 활용하여 코드를 재작성할 수 있음. 이 경우, `login` 함수는 더이상 어떠한 프로바이더가 오는지 신경쓸 필요가 없게 됨. (다형성)

```php
interface Provider {
    public function authorize();
}

function login(Provider $provider)
{
    $provider->authorize();
}
```

인터페이스는 퍼블릭 메소드만 정의 가능함. 이를 통해 어떠한 호출가능한 클래스는 인터페이스의 메소드를 호출 할 수 있게 됨.

추상 클래스는 인스턴스를 만들 수 없음. 대신 서브 클래스에서 상속받아 인스턴스를 생성 할 수 있음.

```php
abstract class Provider {

    abstract Protected function getAuthorizationUrl();
}

class FacebookProvider extends Provider{

    protected function getAuthorizationUrl()
    {
    }
}
```



PHP는 기본적으로 다중상속을 지원하지 않음.

#### 5-3. Summary

Interface defines public API. It defines contract that any implementation has to buy it. however No logic will ever be stored within interface.

Abstract class, something is in common. I can enforce contract by creating abstract method. so, subclass must buy it. However we're also having result to inheritance.



#### 6. Method Injection vs Constructor Injection

Src\AuthController 생성 - receiving Http request and return response.

- Method Injection: if it is the only place that dependency is referenced, if it is single controller method, use method injection 

- Constructor Injection: if you are going to reference this object in multiple places of your class, use constructor injection.