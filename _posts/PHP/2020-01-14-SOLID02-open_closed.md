---
layout: post
title: SOLID Principles in PHP 02 - Open Closed Principle (OCP)
category: PHP
tags: [Laracast, PHP, OOP, SOLID]
comments: true
---



#### 1. GET STARTED

> Open-Closed: Entities(class, method, function etc..) should be open for extension, but closed for modification

확장에는 개방되어야하며 변경에는 폐쇄되어야한다. 기존의 코드를 변경하지 않으면서 기능을 추가할 수 있도록 설계되어야 함.

- open for extension:  it should be simple to change the behavior of a particular entity (class)
- closed for modification: **Goal** (very difficult to follow perfectly). something you should strive for its goal. **Change behavior without modifying original source code** .



#### 2. Modify behavior by doing it from extension

회사의 보스가 사각형을 준비하라고 지시를 내렸음

- 이에 따라 `squre` 이라는 클래스 생성

이후, 보스가 사각형의 면적을 계산해야한다고 지시를 내림 

- Single Responsibility Principle에 따라, 면적만을 계산하는 클래스 `AreaCalculator` 를 따로 생성
- 가장 심플한 형태로 사각형들을 배열로 받아 면적의 합을 누적으로 저장시킴

```php
<?php namespace Acme;

class Square {
    public $width;
    public $height;

    function __construct($height, $width)
    {
        $this->height = $height;
        $this->width = $width;
    }
}
```

```php
<?php namespace Acme;

class AreaCalculator {

    public function calculate($squares)
    {
        $area = 0;
        foreach ($squares as $square)
        {
            $area += $square->width * $square->height;
        }
      
      	return $area;
    }
}
```



이후, 보스가 원에 대해서도 준비해달라고 요청을 하였고, 원의 면적도 고려해서 계산할 수 있는 로직을 구현해달라고 추가 요청을 함. 그러나 현재 `AreaCalculator`  클래스는 사각형만을 고려해서 구현되어있으며 원에 대해서는 적용이 불가능한 상태임. 따라서 클래스를 전면적으로 수정해야함. 이는 Open-Closed Principle을 위반하는 것.

```php
<?php namespace Acme;

class Circle {
    public $radius;

    function __construct($radius)
    {
        $this->radius = $radius;
    }
}
```



OCP를 준수하기 위해 먼저, `AreaCalculator@calculator`  내 반복문 변수명을 `$squares` 을  `$shapes` 으로 변경시킬 수 있음. 그런데 도형에 따라 면적을 계산하는 계산식이 다른 상황임. 이때 우리가 우선적으로 생각할 수 있는 것은 분기문임. 

```php
<?php namespace Acme;

class AreaCalculator {

    public function calculate($shapes)
    {
        foreach ($shapes as $shape)
        {
            if (is_a($shape, 'square')) // if ($shape instanceof Square)
            {
                $area[] = $shapes->width * $shapes->height;
            }
            else
            {
                $area[] = $shapes->radius * $shapes->radius * pi();
            }
        }

        return array_sum($area);
    }
}
```



또 다시, 삼각형 클래스를 추가하여 면적 계산 클래스에 반영해야하는 경우라면 또다시 분기문을 통해 코드를 수정해야 하는 것일까? 이처럼 변화가 생길 때마다 코드를 매번 수정하는것이 맞는 것일까? This is sort of things that lead to code rot.

*how can we extend this behavior while keeping the class closed from modification?* 



#### 3. Seperate extensible behavior behind an interface, and flip the dependencies

`Shape`  인터페이스 생성후 인터페이스를 각각의 클래스에 implement 시킴.

```php
//ShapeInterface.php
<?php namespace Acme;

interface Shape {
    public function area();
}

// Sqaure.php
<?php namespace Acme;

class Square implements Shape {
    public $width;
    public $height;

    function __construct($height, $width)
    {
        $this->height = $height;
        $this->width = $width;
    }

    public function area()
    {
        return $this->width * $this->height;
    }
}
// Circle.php
<?php namespace Acme;

class Circle implements Shape {
    public $radius;

    function __construct($radius)
    {
        $this->radius = $radius;
    }

    public function area()
    {
        return $this->radius * $this->radius * pi();
    }
}
```

 `AreaCalculator@calculate`  는 이제 면적을 계산한 값을 파라미터로 받아 `$area` 에 저장만 하면 됨. 만약에 삼각형에 대하여 추가적인 계산이 필요하더라도 아래 코드는 수정할 필요가 없으며, 삼각형 클래스만 만들어 `area` 메소드를 implement하여 면적 계산만 하면 됨.

```php
<?php namespace Acme;

class AreaCalculator {

    public function calculate($shapes)
    {
        foreach ($shapes as $shape)
        {
            $area[] = $shape->area();
        }

        return array_sum($area);
    }
}
```



#### 4. Practices

제품을 사고 체크아웃 하는 프로세스 로직을 갖고 있는 `Checkout` 클래스를 정의해보자. 

-  `begin` : 영수증을 받아 시작을 함. 그리고 payment를 accept 하는 `acceptCash` 를 호출함
- `acceptCash` : 현금을 받음

```php
class Checkout{
    public function begin(Receipt $receipt)
    {
        $this->acceptCash($receipt);
    }

    public function acceptcash($receipt)
    {
        // accept the cash
    }
}
```



만약에 현금이 아니라 신용카드, 카카오 페이로 계산을 하기 원하는 경우는 어떻게 해야할까? 기존의 코드는 현금을 받아 계산하는 경우만을 고려하여 디자인 되어있음. 이러한 경우는 Open Closed Principle을 고려하지 않은 경우임. Open Closed Principle을 적용할 경우 코드는 다음과 같음.

```php
interface PaymentMethodInterface {
    public function acceptPayment($receipt);
}

class CashPaymentMethod implements PaymentMethodInterface {
    public function acceptPayment($receipt)
    {
    }
}

class Checkout{
    public function begin(Receipt $receipt, PaymentMethodInterface $payment)
    {
        $payment->acceptPayment();
    }
}
```
