---
layout: post
title: Clean code - SOLID 04 - Interface Segregation Principle (ISP)
category: PHP
tags: [clean coders, clean architecture, architecture, OOP, SOLID]
comments: true
---

> 백명석님의 클린코드 [강의](https://www.youtube.com/playlist?list=PLuLb6MC4SOvXCRePHrb4e-EYadjZ9KHyH) 를 듣고 정리한 포스팅 입니다.
> 



#### 1. Interface Segregation Principle

>  Don't depend on things that you don't need. 

사용하지 않지만 의존성을 가지고 있다면, 그 인터페이스가 변경되면 재컴파일/빌드/배포됨. (독립적인 개발/배포가 불가하다는 것을 의미) 또한, SRP와도 연관이 있음. 

- SRP - 요구사항 변경의 원인은 클라이언트임. 내가 가지고 있는 서브 클래스가 인터페이스의 30%만을 사용한다는 것은 그것 자체가 별개의 기능/책임임. 
- 한 기능에 변경이 발생하면 다른 기능을 사용하는 클라이언트들에도 영향을 미침.

사용하는 기능만 제공하도록 인터페이스를 분리함으로써 한 기능에 대한 변경의 여파를 최소화. 클라이언트 입장에서 인터페이스를 분리하라는 원칙



#### 2. Switch 예제

switch가 activate되면 light를 turn on하는 SW를 설계하라

<img src="/assets/php/isp01.png" style="zoom:80%;" />



#### 2-1. 문제점

아래의 코드는 여러가지의 문제점을 가지고 있음

- Switch가 Light에 의존적임. Switch가 Light뿐만 아니라 Fan, Motor 등에도 turnOn할 수 있음
- Switch는 Light에 대해서 알면 안됨. (의존성이 뒤집어져야함)

```java
public class Switch {
    private Light light;
    
    public Switch(Light light){
      this.light = light;
    }
  
  	public void activate() {
      light.turnOn();
    }    
}
```



#### 2-2. 해결책

Switch는 Light에 대한 의존성을 갖지 않아야 함. Switch와 Switchable이 같은 패키지/배포 단위이어야 함.

추상 Interface(Switchable)는 

- 클라이언트(Switch)에 속해야 한다.
- 구현체(Light)와는 관련이 없어야 함.
- 그러므로 인터페이스의 이름은 클라이언트와 연관된 것이어야 한다. (Not Lightable, but Switchable)

<img src="/assets/php/isp02.png" style="zoom:50%;" />



#### 3. Fat class 예제

Job 클래스가 많은 일을 하고 있어서 많은 Fan In. 각 서브 시스템은 서로 다른 이유로 Job에 의존하고 있는 상황임. 이 경우, rebuild에 많은 시간이 소요되며, 독립적인 배포/개발이 불가능하다는 문제점이 야기됨.

<img src="/assets/php/isp03.png" style="zoom:80%;" />



#### 3-1. 해결책

One Interface for a sub system. Job이라는 굉장히 큰 인터페이스를 분리(Segregation)하여 각각의 클라이언트들을 대변하는 인터페이스를 다 뽑아냄.

- 어떤 인터페이스에 변경이 발생하면 Job 클래스와 해당 인터페이스를 사용하는 서브 시스템만 rebuild하면 됨.

<img src="/assets/php/isp04.png" style="zoom:80%;" />



#### 4. Fat class

Fat class(거대한 클래스)를 만나면 인터페이스를 생성하여 Fat Class를 클라이언트로부터 독립시켜야함. (Fat Class에서 다수의 인터페이스를 구현)

인터페이스는 구현체보다는 클라이언트와 논리적으로 결합되므로 클라이언트가 호출하는 메소드만 인터페이스에 정의되어있다는 것을 확신할 수 있음(ISP 준수)

특정 인터페이스의 변경으로 인한 다른 클라이언트 영향을 없애서

- 재컴파일/재배포를 없앰
- 클라이언트들을 다른 독립된 컴포넌트에 배치(클라이언트 + 인터페이스 = 배치단위)할 수 있고 독립적으로 배포(개발) 가능하게 됨