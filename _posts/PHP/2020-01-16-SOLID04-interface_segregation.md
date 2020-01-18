---
layout: post
title: SOLID Principles in PHP 04 - Interface Segregation Principle (ISP)
category: PHP
tags: [Laracast, PHP, OOP, SOLID]
comments: true
---

> Laracasts - SOLID Principles in PHP [강의](https://laracasts.com/series/solid-principles-in-php)를 듣고 정리한 포스팅 입니다.

#### 1. GET STARTED

> A client should not be forced to implement an interface that it doesn't use.  
> knowledge that one object has another object.

스타트렉의 세계관에 있다고 가정할 때 캡틴이라는 클래스가 있을 것이다. 캡틴의 주요 역할 중 하나는 선원들을 고용하는 것이며 코드로 작성하면 다음과 같을 것이다.

- Captain has knowledge of the worker. Captain is depend upon worker, it means that it is all depend up anything that worker is depend upon.
- it is not ideal. too many knowledge looking out here.

```php
class Captain {
    public function hire(Worker $worker)
    {

    }
}
```



Worker 클래스 구성 후 Captain 클래스에서 호출

```php
class Worker {
    public function work(){}
    public function sleep(){}
}

class Captain {
    public function manage(Worker $worker)
    {
        $worker->work();
        $worker->sleep();
    }
}
```



#### 2. A client should not be forced to implement an interface that it doesn't use.  

만약에 이제 캡틴이 안드로이드 선원을 뽑는다고 하면, 인터페이스 `WorkerInterface` 를 만들어 안드로이드 클래스 `AndroidWorker` 가 인터페이스를 구현하도록 할 것이다. 여기서, 안드로이드는 잠을 자지 않으므로 `sleep ` 메소드는 `null` 을 반환하게 코드를 작성할 것임. 

- 여기서 ISP을 위반하게 된다. (client should not be forced to implement an interface that it doesn't use)
- `AndroidWorker`  클래스는 `sleep` 메소드를 강제 받고 있음.

```php
interface WorkerInterface {
    public function work();
    public function sleep();
}

class HumanWorker implements WorkerInterface {
    public function work(){}
    public function sleep()
    {
        return 'human sleeping';
    }
}

class AndroidWorker implements WorkerInterface {
    public function work(){}
    public function sleep()
    {
        return null; // violates the ISP
    }
}
```



`WorkerInterface` 를 다음과 같이 재 분류

-  `WorkerableInterface`  
-  `SleepableInterface` 

 `HumanWorker`  클래스는 위 2개의 인터페이스를 모두 받으며, `AndroidWorker` 클래스는 `WorkerableInterface` 만 호출하는 걸로 ISP를 준수 할 수 있게 됨.

```php
interface WorkerableInterface {
    public function work();
}

interface SleepableInterface {
    public function sleep();
}

class HumanWorker implements WorkerableInterface, SleepableInterface {
    public function work(){}

    public function sleep()
    {
        return 'human sleeping';
    }
}

class AndroidWorker implements WorkerableInterface {
    public function work(){}
}
```



#### 3. Open Closed Principle 적용

캡틴 클래스로 돌아와 코드를 다시 다듬고나면 이제 `sleep` 메소드에 대한 고민을 해야함. 아래처럼 조건문을 걸어주는 방식은 적절한 방법이 아닐 것이다. (OCP 위반)

```php
class Captain {
    public function manage(WorkerableInterface $worker)
    {
        $worker->work();
        if ($worker instanceof AndroidWorker) return;

        $worker->sleep();
    }
}
```



이제 생각할 수 있는 방법은 `ManagableInterface`를 만드는 것.

- `ManageInterface` has a method called `beManaged`
- `HumanWorker` & `AndroidWorker` class implements `ManagableInterface`
- `Captain` class now calls `beManaged` method from `ManagableInterface` only 

이 방법을 통해 `ManagableInterface`  는 더이상 `$worker` 오브젝트에 의존하지 않게 되어, 디자인을 향상시키며 결합도를 낮출 수 있게 됨. 또다른 말로는, `$worker` 가 갖고 있는 어떠한 implementation에 의존하지 않으면서 클라이언트 (`manage` 메소드) 는 `ManagableInterface` 를 준수하는 어떠한 오브젝트 사용할 수 있게됨.

```php
interface ManagableInterface {
    public function beManaged();
}

class HumanWorker implements WorkerableInterface, SleepableInterface, ManagableInterface {
    public function work(){}
    public function sleep()
    {
        return 'human sleeping';
    }

    public function beManaged()
    {
        $this->work();
        $this->sleep();
    }
}

class AndroidWorker implements WorkerableInterface, ManagableInterface {
    public function work(){}
    public function beManaged()
    {
        $this->work();
    }
}

class Captain {
    public function manage(ManagableInterface $worker)
    {
        $worker->beManaged();
    }
}
```



