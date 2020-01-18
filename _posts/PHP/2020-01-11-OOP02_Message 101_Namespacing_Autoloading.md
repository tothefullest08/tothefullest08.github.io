---
layout: post
title: Object-Oriented Bootcamp 02 - Message 101 / Namespacing / Autoloading
category: PHP
tags: [Laracast, PHP, OOP]
comments: true
---

> Laracasts - Object-Oridented Bootcamp [강의](https://laracasts.com/series/object-oriented-bootcamp-in-php)를 듣고 정리한 포스팅 입니다.

#### 1. Messages 101

사람들은 직업을 갖고 있으며 비즈니스를 위해 일을 함. 비즈니스는 사람들을 고용함. 고용된 사람들은 스태프로써 일을 함.

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



#### 2. Namespacing

앞 포스팅에서 작성한 `Person`, `Business`, `Staff` 클래스를 단일 파일로 ungrouping 하도록 하자(1 class per file)

- \src\Person.php
- \src\Business.php
- \src\Staff.php

새로운 디렉토리를 생성하여 각 클래스 별로 파일을 만든 후에 클래스를 호출하는 가장 기본적인 방법은 다음과 같음. 하지만 아래의 방법은 준비하는데 시간이 많이 걸릴 뿐더러 효과적인 방법이 아님.

```php
<?php

require 'src/Person.php';
require 'src/Business.php';
require 'src/Staff.php';

  
$harry = new Person('harry lee');
$staff = new Staff([$harry]);
$laracasts = new Business($staff);

$laracasts->hire(new Person('Ron Wizlie'));

var_dump($staff);
var_dump($laracasts->getStaffMembers());
```



#### 3. Composer

클래스를 Autoloading 하는 방법을 이용할 수 있음( `composer` 사용). 루트 디렉토리에 composer.json 생성 (파일안에는 어떠한 dependency도 선언할 필요는 없음)  

```php
// composer.json
{
}
```

 `composer install` 을 진행하면 vendor 디렉토리가 생성된 것을 알 수 있음. 이후 composer.json에서 autoload 관련 dependecy를 선언

```php
{
  "autoload": {
    "psr-4": {
      "Acme\\": "src"
    }
  }
}
```



src 디렉토리 내 클래스 파일에 각각 namespace 설정한 후 `composer dump-autoload`  입력을 하면 \composer\autoload_psr4.php에 코드가 추가된 것을 알 수 있음.

```php
// Person.php
<?php

namespace Acme;

// autoload_psr4.php

<?php

$vendorDir = dirname(dirname(__FILE__));
$baseDir = dirname($vendorDir);

return array(
    'Acme\\' => array($baseDir . '/src'),
);
```



#### 4. Autoloading

autoloading 설정이 완료되었으므로 base 파일로 돌아가 Acme\를 앞에 붙여주며 클래스를 호출 하면됨. 

```php
<?php

$harry = new Acme\Person('harry lee');
$staff = new Acme\Staff([$harry]);
$laracasts = new Acme\Business($staff);
$laracasts->hire(new Acme\Person('Ron Wizlie'));

var_dump($laracasts->getStaffMembers());
```



그러나 아직까지 에러가 발생함. 

> PHP Fatal error:  Uncaught Error: Class 'Acme\Person' not found in ...

that's because we have an autoloading component, but we haven't yet pulled it into a project. for most frameworks and packages, usually that would be done at the entry point.

for example, index.php would require autoaloading. you only have to do that once for the whole project. => `require 'vendor/autoloading.php` 입력

구조를 좀 더 클린하게 바꾸기 위해 index.php를 만들고 autoloading과 클래스를 호출하는 파일을 import 하자. ex.php에서는 클래스 앞에 붙여진 Acme\를 제거하고 파일 상단에 `use` 로 대체.

```php
// index.php
<?php

require 'vendor/autoload.php';
require 'ex.php';

// ex.php
<?php

use Acme\Person;
use Acme\Staff;
use Acme\Business;

$harry = new Person('harry lee');
$staff = new Staff([$harry]);
$laracasts = new Business($staff);
$laracasts->hire(new Person('Ron Wizlie'));


var_dump($laracasts->getStaffMembers());
```



현재, 이렇게 구조를 바꿨음에도 불구하고, Staff.php의 `add`  메서드의 파라미터에 Person을 그대로 쓸수 있는 것은 동일한 namespace를 공유하고 있기 때문임. 

만약에 src 디렉토리 내에 Users 디렉토리를 생성한 후 Person.php를 집어넣는다면 에러가 발생함. 네임스페이스는 디렉토리 구조를 반드시 따라야함. 아울러, Person 클래스를 파라미터로 받고 있는 Business와 staff는 Person 클래스가 동일한 네임스페이스를 공유하고 있지 않으므로 클래스를 따로 export 해줘야함. 

```php
// Users\Person.php
namespace Acme\Users;

// Business.php
use Acme\Users\Person;

// Staff.php
use Acme\Users\Person;

// ex.php
use Acme\Users\Person;
```



#### 5. Summary

1. composer.json - reference that you are going to use "psr-4" autolading.
   1. as the key, you specify your root namespace  (corresponding to your product)
   2. as its value, you specify what directory should be associated with root namespace.
2. Person.php is in Users directory, thus change the namespace as `namespace Acme\Users;` 
3. if you follow this convention, remember to require autoloader at some point of your project 

