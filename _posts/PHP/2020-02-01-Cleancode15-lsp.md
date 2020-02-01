---
layout: post
title: Clean code - SOLID 03 - Liskov Substitution Principle (LSP)
category: PHP
tags: [clean coders, clean architecture, architecture, OOP, SOLID]
comments: true
---

> 백명석님의 클린코드 [강의](https://www.youtube.com/playlist?list=PLuLb6MC4SOvXCRePHrb4e-EYadjZ9KHyH) 를 듣고 정리한 포스팅 입니다.
> 



#### 1. Liskov Substitution Principle

> 서브 타입(자식 클래스) 은 언제나 자신의 기반타입(부모 클래스) 으로 교체 할 수 있어야 한다. 

객체지향에서 많이들 이야기하는 것이 슈퍼 클래스에 많은 기능을 넣어 넣은 후, 상속받아 서브 클래스에서 사용하는 것을 재 사용성이라고 한다. 하지만 이는 down casting으로 LSP에 위배됨. 타입에 대한 의존성은 아주 강한 의존성으로 반드시 지양 해야함.



#### 2. OCP vs LSP

OCP의 경우 

- abstraction, polymorphism(inheritance)을 적용하여 구현 
- 어떤 인터페이스나 추상 클래스가 있을 때, 실제 구현체가 무엇인지 모름. 어떠한 형태가 담겨있는지 모른 채 수행 됨. 
- 여러 디테일한 부분은 추상화하여 하나의 인터페이스로 표현

LSP의 경우

- OCP를 받쳐주는 polymorphism에 관한 원칙을 제공함. (서브타입은 언제나 자신의 슈퍼타입으로 교체될 수 있어야 한다.)
- instanceof / downcasting을 사용하는 것은 전형적인 LSP 위반의 징조 (instanceof / downcasting를 통해 정확한 타입을 알아야 하는 일이 있어서는 안된다)
- (백명석님 사견으로는) 서브클래스에서는 슈퍼클래스를 사용해서는 안됨. (지저분하고 난잡해짐)
- LSP가 위반되면 OCP도 위반됨. LSP를 위반하면 서브타입이 추가될때마다 클라이언트들이 수정되어야 함.



#### 3. Rectangle 예제

<img src="/assets/php/lsp01.png" style="zoom:80%;" />



예) Rectangle은 시스템의 여러곳에 퍼져있음. 이러한 상황에서 정사각형(Square)을 서브타입으로 추가하려고 한다.

- Square IS-A Rectangle (정사각형은 사각형이다. IS-A 관계 성립 ➔ 상속 관계로 표현이 가능함)
- setWidth와 setHeight에서 난항을 겪음 
  - setWidth는 가로와 세로를 똑같이 width로 셋팅을 해야하나? 
  - setHeight는 가로세로를 똑같이 height로 셋팅을 해야하나?

<img src="/assets/php/lsp02.png" style="zoom:80%;" />



Failed RectangleTest

- instanceof를 사용하게되어 LSP위반이 발생함

<img src="/assets/php/lsp03.png" style="zoom:80%;" />



#### 4. The Representative Rule

대리인은 자신이 대리하는 어떤 것들에 대한 관계까지 대리(공유)하지는 않는다.

- 이혼 소송 변호사들(대리인)이 이혼하는 부부들의 관계(부부)를 대리(공유)하지 않음
- 따라서 기하학적에 따르면 Square IS-A Rectangle이지만, 이들을 표현/대리(represent)하는 S/W는 꼭 그들의 관계(IS-A)를 공유하지 않음

코드에 downcast, instanceof, 서브클래스에서 슈퍼클래스를 호출하는 경우가 발생한다면 상속관계를 끊고 composition 관계로 옮길 수 있는지를 고민해야함.

