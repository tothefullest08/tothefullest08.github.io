---
layout: post
title: SOLID Principles in PHP 01 - Single Responsibility Principle (SRP)
category: PHP
tags: [Laracast, PHP, OOP, SOLID, Architecture]
comments: true
---

> Laracasts - SOLID Principles in PHP [강의](https://laracasts.com/series/solid-principles-in-php)를 듣고 정리한 포스팅 입니다.

#### 1. GET STARTED

>  A Class Should have one, and only one, reason to change

현재 아래의 예시는 여러 방면으로 Single Responsibility Principle을 준수하고 있지 않다. 사용자 인증, DB 접근, 결과 반환 등 너무 많은 책임과 역할을 수행하고 있음.

```php
// SalesReporter.php
class SalesReporter {
  
  public function between($startDate, $endDate)
  {
    //perform authentication
    if ( ! Auth::check()) throw new exception('Authentication reqiured for reporting');
    
    // get sales from db
    $sales = $this->queryDBForSalesBetween($startDate, $endDate)
      
    // return results
    return $this->format($slaes);
  }
  
  //quering database
  protected function queryDBForSalesBetween($startDate, $endDate)
  {
    return DB::table('sales')->whereBetween('created_at', [$startDate, $endDate])->sum('charged') / 100;
  }
}

	protected function format($sales)
  {
    return "<h1>Sales: $sales</h1>";
  }
}
```

```php
// routes.php

Route::get('/', function()
{
	$reporter = new Acme\Reporting\SalesReporter();
  
  $begin = Carbon\Carbon::now()->subDays();
  $end = Carbon\Carbon::now();
  
  return $report->between($begin, $end);
})
```



#### 2. SalesReporter과 user Authentication을 왜 신경써야하는가? 

That's application logic. It does not belong in here→ perform authentication 코드 삭제



#### 3. querying data 코드에서  `SalesReporter`가 너무 많은 responsibility를 갖고 있음 

*"too many reasons to change, or too many consumer of this class"*

예를 들어, persistence layer(데이터 처리 담당 계층)가 향후 변경된다면 아래 코드를 변경해야만 할 것. 또한, output의 포맷을 바꿔야하는 경우도 아래 코드를 변경해야 할 것이다. 이러한 두가지의 이유로 기 클래스는 SRP를 준수하고 있지 않다고 할 수 있다.

```php
return DB::table('sales')->whereBetween('created_at', [$startDate, $endDate])->sum('charged') / 100;
```

 persistance layer가 무엇이고 어떻게 정보를 갖고올건지는 `SalesReporter` 클래스의 responsibility가 아님. 

- 이러한 역할을 하는 인터페이스를 생성자 메소드에 주입(`SalesRepository` ; 이론적으로 `SalesRepositoryInterface` 라 지어야 하나, 이부분은 뒤부분에서 다룰 예정)
- 사용을 위해 `use Acme\Repositories\SalesRepository;` 후, 디렉토리 & 파일 생성

```php
// SalesReporter.php
use Acme\Repositories\SalesRepository;
```

`SalesRepository`  클래스가 이제 database specific interaction 을 담당함.  

- `SalesReporter@queryDBForSalesBetween` 를  `SalesRepository`  클래스로 이동
- 접근 제어자는 public으로 변경
- 메소드 명은 좀 더 친화적으로 `queryDBForSalesBetween` 에서 `between` 으로 변경
- `SalesReporter@between` 에서 불러오는 메소드 명 변경

```php
// Repositories\SalesRepository.php

namespace Acme\Repositories;

class SalesRepository {
  protected function between($startDate, $endDate)
  {
    return DB::table('sales')->whereBetween('created_at', [$startDate, $endDate])->sum('charged') / 100;
  }
}

// SalesReporter.php
public function between($startDate, $endDate)
{
  $sales = $this->repo->between($startDate, $endDate)
```



#### 4. why should this class care or be this class's responsibility to ouput/format/print the result?

현재 코드에서 다른 포맷으로 아웃풋을 넘기는 것이 가능한가? 현재 코드는 HTML을 assume 하고 있는 상태임. 만약에 Json으로 포맷을 바꾸고 싶다면? 또는 다른 포맷들을 함께 넘기고 싶다면? 그럴때마다 코드를 업데이트 해줘야하는 번거로움이 발생함. 이에 대한 방안은 여러개가 있음.

```php
protected function format($sales)
{
  return "<h1>Sales: $sales</h1>";
}
```

1. we are going to leave the formatting to the consumer of the class
2. class based formatting을 원할 경우. 포맷팅을 describe하는 인터페이스 생성
   - 이후, 인터페이스를 호출하여 코드 업데이트
   - `format` 메소드는 삭제가능
   - routes.php 최종 다듬기(인스턴스 넘기기)

```php
// SalesOutputInterface.php
namespace Acme\Reporting;

interface SalesOutputInterface {
  public function output($sales);
  
}

// HtmlOutput.php
namespace Acme\Reporting;
use Acme\Reporting\SalesOutputInterface;

class HtmlOutput implements SalesOutputInterface{
  public function output($sales)
  {
    return "<h1>Sales: $sales</h1>";
  }
}

// SalesReporter.php
public function between($startDate, $endDate, SalesOutputInterface $formatter)
{
  $sales = $this->repo->between($startDate, $endDate)
  
  $formatter->output($sales);
  
// routes.php
Route::get('/', function()
{
	$reporter = new Acme\Reporting\SalesReporter(new \Acme\Repositories\SalesRepository);
  
  $begin = Carbon\Carbon::now()->subDays();
  $end = Carbon\Carbon::now();
  
  return $report->between($begin, $end, new Acme\Reporting\HtmlOutput);
})  
```



#### 5. Summary

routes.php

```php
Route::get('/', function()
{
	$reporter = new Acme\Reporting\SalesReporter(new \Acme\Repositories\SalesRepository);
  
  $begin = Carbon\Carbon::now()->subDays();
  $end = Carbon\Carbon::now();
  
  return $report->between($begin, $end, new Acme\Reporting\HtmlOutput);
})  
```

SalesReporter.php

```php
use Acme\Repositories\SalesRepositories;
use Auth, DB, Exception;

class SalesReporter {
  
	private $repo;
  
  public function __construct(SalesRepository $rep)
  {
    $this->repo = $repo
  }
  public function between($startDate, $endDate, SalesOutputInterface $formatter)
  {
    $sales = $this->repo->between($startDate, $endDate)

    $formatter->output($sales);
  }
}
```

Repositories\SalesRepository.php

```php
namespace Acme\Repositories;

class SalesRepository {
  protected function between($startDate, $endDate)
  {
    return DB::table('sales')->whereBetween('created_at', [$startDate, $endDate])->sum('charged') / 100;
  }
}
```

SalesOutputInterface.php

```php
namespace Acme\Reporting;

interface SalesOutputInterface {
  public function output($sales);
  
}
```

HtmlOutput.php

```php
namespace Acme\Reporting;
use Acme\Reporting\SalesOutputInterface;

class HtmlOutput implements SalesOutputInterface{
  public function output($sales)
  {
    return "<h1>Sales: $sales</h1>";
  }
}
```

