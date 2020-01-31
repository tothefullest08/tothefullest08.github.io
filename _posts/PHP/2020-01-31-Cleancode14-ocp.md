---
layout: post
title: Clean code - SOLID 02 - Open Closed Principle(OCP)
category: PHP
tags: [clean coders, clean architecture, architecture, OOP, SOLID]
comments: true
---

> 백명석님의 클린코드 [강의](https://www.youtube.com/playlist?list=PLuLb6MC4SOvXCRePHrb4e-EYadjZ9KHyH) 를 듣고 정리한 포스팅 입니다.



#### 0. Open Closed Principle

> 확장에 대해선 열려 있고, 변경에 대해선 닫혀있다. 



#### 1. Copy Example

Copy Module을 컴파일도 안하고 Low Level Details를 변경할 수 있다(예. 장치 추가)

- Abstraction and Inversion
  - 추상화 인터페이스(reader, writer)를 copy 와 device 사이에 넣어서 의존성을 역전시킴

<img src="/assets/php/ocp1.png" style="zoom:50%;" />

- 기존 코드

<img src="/assets/php/ocp2.png" style="zoom:50%;" />

- 잘못된 변경 코드 예시

<img src="/assets/php/ocp3.png" style="zoom:50%;" />

**확장이 필요한 행위를 Abstraction시켜야 함**

- Open for extension: 디바이스가 추가되면 해당 디바이스를 담당하는 클래스를 추가
- Closed for modification: 그러나 copy 로직의 수정은 발생하지 않음.

<img src="/assets/php/ocp4.png" style="zoom:50%;" />



#### 2. Is This Possible?

OCP를 준수하면 Modification을 완벽히 제거할 수 있나? 이론적으로는 가능하나, 실용적이지 못함 

- 2가지의 문제점 발생



#### 2-1. Main Partition

main에서 의존성 주입을 해줘야하는 경우가 있음. (그곳에는 if/else 존재) main은 OCP를 준수할 수 없음.



#### 2-2. Crystal ball problem

확장을 위해 미리 인터페이스를 준비해놓는다는 것이 현실적으로 불가능 함. 

- 아무리 잘 찾고 예측하더라도 고객은 **반드시 당신이 준비하지 못한 것에 대한 기능 추가/변경을 요구** 한다 - **Unknown Unknowns**. 미래의 변경으로부터 보호 받도록 Abstraction을 적용하여 설계하는 것은 쉽다. (만일 미래에 어떤 변경이 있을지 알 수 있다면 말이다.) 하지만 우린 그런 미래를 알 수 있는 Crystal ball이 없음.

- 이게 사람들이 **말하기 꺼리는 OCP, OOD에 대한 하나의 더러운 비밀**이다. (OCP, OOD는 당신이 미래를 예측할 수 있을 때만 해당 기능을 보호할 수 있다.)
- 미래를 예측하지 못한다면 어떻게 해야하는가? 완벽한 선견력이 필요하다면 객체지향의 장점은 무엇인가? 지난 30년간 SW 산업은 이 문제와 투쟁을 해왔음. 이러한 노력의 일환으로 Crystal ball의 필요성을 제거하기 위한 2가지 주요한 접근법을 식별했음.



#### 3. Big Design Up Front(BDUF)

조심스럽게 고객과 문제 영역을 고찰하며, 고객의 요구사항을 치밀하고 과도하게 예측하여 도메일 모델을 만든다. 이를 통해 OCP가 가능하도록 도메인 모델에 추상화를 적용시킨다. 그리고 변경될 가능성이 있는 모든 것들에 대한 청사진을 얻을 때 까지 헛된 짓을 계속한다.

- 대부분의 경우 필요치 않는 추상화로 도배된, 매우 크고 무겁고 복잡한 쓰레기 설계를 만드는 문제가 발생한다.
- 추상화는 유용하고 강력한 만큼 비용도 크다.



#### 4. Agile Design

변화에 대한 가장 좋은 예측은 변화를 경험하는 것이다. 발생할 것 같은 변화를 발견한다면 향후 해당 변화와 같은 종류의 변화로부터 코드를 보호할 수 있다.

고객이 요구할 모든 종류의 변경을 완벽하게 예측하고 이에 대한 변경에 대응하기 위해 Abstraction을 적용하는 대신, 고객이 변경을 요구할 때까지 기다리고 Abstraction을 만들어서 향후 추가적으로 재발하는 변화로부터 보호될 수 있도록 하라.

- 점진적으로 추상화를 추가(필요할 때마다)
- 고객이 변경을 요구하면 Agile Designer는 코드를 리팩토링해서 그런 종류의 변경이 쉽게 일어날 수 있도록 추상화를 추가한다(OCP를 준수하도록)



#### 5. Agile Design in Practice

물론 우리는 실제로 BDUF와 Agile 두 극단 사이에 살고 있다.  (BDUF를 피해야 하지만 No DUF도 피해야 한다)

시스템에 대해서 사고하고 Decoupled 모델을 사전 설계하는 것은 가치있는 일이다. 하지만 간단하고 적은 면에 있다. 우리의 목적은 시스템의 기본 모양을 수립하는 것이지 모든 작은 상세까지 수립하는 것이 아니다.

문제에 대해 과하게 생각하면 유지보수비용이 높은 많은 불필요한 추상화를 만들게 된다.

빨리 자주 배포하고 고객의 요구사항 변화에 기반하여 리팩토링하는 것이 매우 가치 있다. 이럴 때 OCP가 진가를 발휘한다. 하지만 간단한 도메인 모델없이 이렇게 진행하면 방향성 없는 혼란한 구조를 유발한다.



#### 6. Reprise

마술이 아니라 공학이다. OCP를 완벽하게 준수하는 것은 불가능하다. 모든 것을 생각해 낼 수는 없다. 

당신의 목적은 변경의 고통을 완전히 제거하는 것이 아니다. 이것은 불가능하다. **당신의 목적은 변경을 최소화하는 것이다**. 이게 OCP를 준수하는 디자인이 당신에게 주는 잇점이다.

OCP는 시스템 아키텍처의 핵심이다. 완벽한 보호(변경으로부터의)를 얻을 수는 없지만, 얻기 위해 투쟁할 필요는 있다.



 

