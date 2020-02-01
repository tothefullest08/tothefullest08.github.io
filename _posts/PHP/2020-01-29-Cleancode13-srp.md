---
layout: post
title: Clean code - SOLID 01 - Single Responsibility Principle (SRP)
category: PHP
tags: [clean coders, clean architecture, architecture, OOP, SOLID]
comments: true
---

> 백명석님의 클린코드 [강의](https://www.youtube.com/playlist?list=PLuLb6MC4SOvXCRePHrb4e-EYadjZ9KHyH) 를 듣고 정리한 포스팅 입니다.



#### 1. Single Responsibility Principle

>  클래스는 하나의 책임을 가져야 한다. 책임은 무엇인가?

```php
class Employeelmpl {
  public function calculatePay(){}
  public function save(){}
  public function describeEmployee(){}
  public function findById($id){}
}
```

- Save/findById - 같은 부류 (DB와 연관된 기능)
- 같은 책임을 갖는 기능 - calculatePay
  - calculateDeduction, calculateSalary
  - 메소드가 추가되어도 책임의 수는 변하지 않음
- **같은 부류로 나뉘어 책임이 몇개인가를 결정**
- 부류
  - 메소드의 client에 의해 결정(저 메소드를 누가 사용하는가?)
  - **누가 해당 메소드의 변경을 유발하는 사용자인가?** 
    - (save라는 메소드의 로직 변경이 필요해. 누구때문에 save 로직의 변경이 필요한가?)

 

#### 2. It's About Users

SRP는 사용자에 관한 것. 책임?

- SW의 변경을 요청하는 특정 사용자들에 대해 클래스/함수가 갖는 것
- **변경의 근원** 이라고 볼 수 있음. 변경의 원인. 무엇 때문에 변경이 일어나는가? 변경의 원인이 같은 것은 같은 책임이라고 볼 수 있음
- User들은 그들이 수행하는 Role에 따라 나눠야한다. User가 특정 Role을 수행할 때 Actor라고 부른다. 
  - 책임은 개인이 아니라 Actor와 연결
  - Actor: 서로 다른 Needs, Expectation을 가짐, Employee 클래스의 변경을 요구하는 사용자들
- Employee 클래스에는 3개의 Actor가 있다.
  - Policy, Architect, Operations

<img src="/assets/php/srp01.png" style="zoom:30%;" />



#### 3. Reprise

<img src="/assets/php/srp07.png" style="zoom:50%;" />

- Responsibility - 특정 Actor의 요구사항을 만족시키기 위한 일련의 함수의 집합
- Actor의 요구사항 변경이 일련의 함수들의 변경의 근원이 됨



#### 4. Two Values of S/W

- Secondary value of SW: 현재의 SW가 현재 사용자의 현재 요구사항을 만족하는가?
- Primary value of S/W: 지속적으로 변화하는 요구사항을 수용(tolerate, facilitate) 하는 것. 대부분의 S/W는 현재의 요구사항을 잘 만족하지만 변경하긴 어렵다.



#### 5. Collision

Employee 클래스 처럼 3가지의 책임을 한 클래스에 몰아 넣을 경우, primary value가 저하됨.

- Policy Actors: Business rule의 변경을 필요로 함
- Architector Actors: DB Schema의 변경을 필요로 함
- Operations Actors: Business rule의 변경을 원함
- 동일 모듈 변경. Merge 충돌. Source Repo 충돌



#### 6. Fan Out

- Employee 클래스는 너무 많은 것을 알고 있음(비즈니스 룰, 데이터베이스, 리포팅, 포맷팅)
- 많은 책임을 갖고 있음.
- 각각의 책임은 Employee가 다른 클래스들을 사용하도록 한다.
- Employee 클래스에 거대한 Fan Out이 존재함. 
- 변경에 민감. Employee User는 더 민감함. 따라서 Fan Out을 제한하는 것이 좋음.
  - 좋은 방법은 책임을 최소화 하는 것

<img src="/assets/php/srp02.png" style="zoom:50%;" />



#### 7. Collocation is Coupling

Operations Actor가 새로운 리포트 기능을 필요하다고 가정할 경우 새로운 리포트 기능도 Employee 클래스에 추가함. 기존 책임(Policy, Architecture)에는 변경이 없음에도 새로운 리포트가 추가되어 Employee 클래스가 변경.

- 새로운 리포트 기능이 Employee 클래스에 추가되면 이 기능을 필요로 하지 않는 Employee 클래스를 사용하는 모든 클래스들이 다시 컴파일/배포되어야 함
- 모든 액터들이 영향을 받게 된다(Collocation of responsibilities couples the actors)



#### 8. SRP(Single Responsibility Principle)

하나의 모듈은 반드시 하나의 변경 사유를 가져야 한다 (One and only one responsibility)

- 동일한 이유(같은 사용자, Actor)로 변경(호출)되어야 하는 것들은 동일 모듈에 있어야 한다.
- 다른 이유로 변경되어야 하는 것들은 다른 모듈에 있어야 한다.
- 변경 사유 = Actor = 클래스를 사용하는 사람들. 
- 하나의 클래스는 한 클라이언트들만 서빙을 해야한다는 것을 의미.



#### 9. 시스템 설계

SRP는 시스템을 설계할 때 가장 적용하기 좋음. 

- Actor 파악에 주의해야 함. Actor들을 serve하는 책임들을 식별할 것
- 책임을 모듈에 할당 (각 모듈이 반드시 하나의 책임을 갖도록 유지)
- 유즈 케이스 분리의 이유
  - 다른 이유(다른 Actor)로 인해 변경되고, 다른 때에 변경되기 때문

<img src="/assets/php/srp03.png" style="zoom:40%;" />



#### 10. Solutions

Employee 클래스 내 3개의 Actor, 3개의 Responsibility가 존재함. 어떻게 Responsibility를 분리하나? 정답은 없음



#### 10-1. Inverted Dependencies

의존성 역전. OOP에서 이런 의존성을 다루는 전략. 클래스를 인터페이스와 클래스로 분리시킴
- Actor를 클래스에서 Decouple
그러나, 모든 Actor들이 하나의 인터페이스에 coupled 되어 있으며, 하나의 클래스에 구현되어 구현도 coupled

<img src="/assets/php/srp04.png" style="zoom:50%;" />



#### 10-2. Extract Classes

3개의 책임을 분리하는 방법: 3개의 클래스로 분리

- Actor 들은 분리된 3개의 클래스에 의존
- 3개의 책임에 대한 구현은 분리
- 하나의 책임이 변경되어도 다른 책임에 영향을 미치지 않음

문제점

- transitive dependency (EmployeeGateway/EmployeeReporter -> Employee). 
  - Employee에 변경이 있을 때, EmployeeGateway/EmployeeReporter에 영향을 미칠 수 있음.
- Employee의 개념이 3개의 조각을 분리됨.

<img src="/assets/php/srp05.png" style="zoom:50%;" />



#### 10-3. Inverted Dependencies + Extract Classes

위 두가지 방법을 적절히 배합하면 더 나은 결과를 도출 시킬 수 있음. 우선 인터페이스를 extract 한후, 책임에 따라 인터페이스를 잘게 쪼개어 3개의 인터페이스로 분리시킴.

이후, EmployeeImpl(Employee 클래스)가 3개의 인터페이스를 구현 (또는 EmployeeImpl를 3개로 나눠도 됨) . 이를 통해 특정 Actor에 의한 인터페이스 변경이 일어나더라도 다른 Actor들은 영향을 받지 않게 됨.



#### 10-4. Facade

어디에 구현이 있는지 찾기 쉬워짐.

- Put all 3 function families into a facade class. 
- And, Facade delegates to the 3 different implementations) 

어디에 구현되어있는지 찾기 쉬운 장점이 있지만 여전히 Actor들은 Coupled 되어 있음.

<img src="/assets/php/srp06.png" style="zoom:50%;" />



#### 10-5. Interface Segregation

각 책임에 대하여 3개의 인터페이스를 만든 후, 3개의 인터페이스를 하나의 클래스로 구현

Actor들은 완전히 decoupled되어 있지만, 어디에 구현되어있는지 찾기 어려우며, 하나의 클래스에 구현되어 여전히 구현은 coupled 되어 있음.

<img src="/assets/php/srp07.png" style="zoom:50%;" />



#### 11. Case Study

패키지 다이어그램을 살펴보면 Customer Actor를 위한 책임이 어플리케이션 아키텍처의 중심인 것을 알 수 있음. 각 패키지는 각 액터들을 위한 책임을 구현하였음.

- 패키지 간의 의존성 방향에 주의(모두 gamePlay 패키지로 향하고 있음)
- 이 설계는 좋은 아키텍처 (어플리케이션이 중앙에, 다른 책임들은 어플리케이션에 plug into)

<img src="/assets/php/srp08.png" style="zoom:40%;" />



- 각 클래스는 하나의 액터만을 위한 기능을 제공
  - 하나의 클래스는 반드시 하나의 책임만을 가짐
- 패키지간의 의존성이 한 방향으로 향하고 있음.

<img src="/assets/php/srp09.png" style="zoom:50%;" />

#### 12. Faking It

Waterfall 순서로 한 것 같은가? step by step으로 이 복잡한 것들을 모두 찾아 낸 것 같은가? (액터 → 패키지 다이어 그램 → 클래스 다이어그램 → 코드) 

실제로는 그렇지 않음. 실제로 한 것은

1. 제일 먼저 테스트를 작성하고 통과하도록 했다. 
2. 스코어를 계산하는 동작하는 함수를 하나 만들고, 추측하는 로직을 위한 동작하는 함수를 하나 만들고, 설계가 드러날 때까지 이 함수 저 함수를 리팩토링 했다.
3. 동작하는 전체 게임을 얻을 때까지 모든 동작들이 테스트에 성공하도록 설계를 적용했다.
4. 그리고 아키텍처를 살폈다. 테스트가 당신들에게 보여준 설계의 80%를 유도했다. 
5. 그리고 테스트가 3개의 책임을 식별하는 것을 도왔다.
6. unit test가 확보된 후에 무차별적인 리팩토링을 수행했다(디자인을 향상시키기 위해서)
7. 이런 후에만 3개의 액터들이 식별된다. 그러면 클래스들을 3개의 패키지로 분리시킨다.
8. 마지막으로 이쁜 다이어그램을 그린다. 이쁜 다이어그램을 그리기 가장 좋은 때는 완료된 후이다.

