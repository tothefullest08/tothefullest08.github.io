---
layout: post
title: SOLID Principles in PHP 05 - Dependency Inversion Principle (DIP)
category: PHP
tags: [Laracast, PHP, OOP, SOLID]
comments: true
---

> Laracasts - SOLID Principles in PHP [강의](https://laracasts.com/series/solid-principles-in-php)를 듣고 정리한 포스팅 입니다.

#### 1. Depedency Inversion(의존성 역전)

-  High level modules should not depend upon low level modules. Instead, they should depend upon abstractions(All of this is about decoupling code)
-  Depend on abstractions, not on concretions.
-  Dependency Inversion does not equal Dependency Injection



#### 2. High level & Low Level

- High Level Code - isn't as concerned with detail. is not concerned with specific details.
- Low Level Code - is more concerned with details and specifics.
- Higher level code should never have to depend upon low level code. One class should never be forced to depend upon a specific implementation. Instead, it should depend upon a contract or abstractions.



전등, 텔레비전 등의 장치들을 갖고 있다고 가정할 때, 우선적으로 전력을 공급해야만 함. (plug for). 대신에 멀티탭(인터페이스)으로 각 장치들을 묶어서 전력을 공급한다고 해보자. 장치들은 사실 어떻게 전력을 공급받는지는 관심이 없으며, 멀티탭에 연결만 하면 그들이 원하는(전력 공급) 것만 받으면 되는 것임.

좀 더 개념을 확장하면, 집은 멀티탭(인터페이스 = 추상화, abstraction)를 소유하고 있다고 볼 수 있음. 그리고 전력 공급이 필요한 어떠한 장치도 멀티탭(인터페이스)의 규칙을 따라야만 함. 개별 장치(전등, 텔레비젼, 닌텐도)는 인터페이스를 갖고 있는 것이 아니라 인터페이스를 따르고 있음. 이 경우 하이 레벨 코드와 로우 레벨 코드 둘다 abstraction에 의존하고 있음. 이제, 어떤 파트가 abstraction을 갖고 있냐는 것이 흥미로운 질문이 될 수 있음.



#### 3-1. Practices

`PasswordReminder` 클래스를 구현하고 생성자 메소드로, 데이터베이스에 연결하는 클래스를 파라미터로 받는다고 가정해보자. 이때 Typehint로  데이터베이스 연결 클래스는 `MySQLConnection` 라고 정의하자. 이러한 디자인 설계(코드 작성)는 적절하지 못한 방법이다.

-  `PasswordReminder` 클래스는  `$dbconnection` 가 무엇인지 왜 신경써야하는가? Depedency Injection의 관점에서는 코드상의 문제가 없음 (클래스를 파라미터로 의존성 주입하고 있음). 그러나 Dependency Inversion의 관점에서 볼 땐 DIP를 위반하고 있음. 
- 하이 레벨 모듈  `PasswordReminder`  은 로우 레벨 모듈 `MySQLConnection`  에 의존하면 안됨. abstraction에 의존해야하며 concrete implementation에 의존하면 안됨. 
- `PasswordReminder` 는 어떻게 데이터베이스에 연결하는지에 대한 지식이 필요 없음. 그저 데이터베이스가 연결되고 작동만 하면 됨 →  인터페이스 생성

```php
class PasswordReminder {

    /**
     * @var MySQLConnection
     */

    private $dbConnection;

    public function __construct(MySQLConnection $dbconnection)
    {
        $this->dbConnection = $dbconnection;
    }
} 
```



#### 3-2. Depends on abstraction

abstraction에 의존하도록 코드를 아래와 같이 변경시킬 수 있음.

- 하이 레벨 모듈( `PasswordReminder` ) : `\ConnectionInterface` 인터페이스를 생성하고 `connect` 메서드 구현. 이후, `PasswordReminder` 의 파라미터로는 인터페이스를 넘김(abstraction에 의존)
- 로우 레벨 모듈(`DbConnection` ) : abstraction에 의존하도록 클래스 생성 후 인터페이스 구현

```php
interface ConnectionInterface {
    public function connect();
}

class DbConnection implements ConnectionInterface {    
  public function connect()    
  {          
  }
}
class PasswordReminder {

    /**
     * @var MySQLConnection
     */

    private $dbConnection;

    public function __construct(ConnectionInterface $dbconnection)
    {
        $this->dbConnection = $dbconnection;
    }
}
```



> Dependency Injection과 Dependency Inversion는 동일한 개념이 아님. 그러나 Dependency Injection는 Dependency Inversion을 준수 할 수 있는 방법론을 제공함.



#### 4. IoC(Inversion of Control) - Who owns Abstraction?

하이 레벨 모듈과 로우 레벨 모듈 중 어떤 것이 abstraction( `ConnectionInterface` ) 을 소유하고 있는지를 확인해보면,  결과론적으로는 둘다 소유하고 있지 않음. 
