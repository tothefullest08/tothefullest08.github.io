---
layout: post
title: Clean code - Architecture
category: PHP
tags: [clean coders, clean architecture, architecture, OOP]
comments: true
---

> 백명석님의 클린코드 [강의](https://www.youtube.com/playlist?list=PLuLb6MC4SOvXCRePHrb4e-EYadjZ9KHyH) 를 듣고 정리한 포스팅 입니다.

#### 1. What is Architecture

> 전체적인 시스템 개발에 기반을 제공하는 변경 불가능한 초기 결정 사항의 집합(Set of irrevocable early decisions that raised the foundation for the entire system development)

- Java, Eclipse, Spring, MySQL, MVC 등은 아키텍처가 아니다. 
  - 건물을 지을 때 해머, 못, 기와장 벽돌 등이 아키텍처가 아니듯이, 이 또한 도구이며 뭔가를 만들기 위한 재료에 불과하다. 
- 아키텍처는 사용법(Usage)여야 한다. (이것이 무엇인지 사용법을 드러내는 것)



#### 2. What is Use Case

> A list of steps, typically defining interactions between a role(actor) and a system, to achieve a goal

- 시스템의 사용법(Usage)를 보여주는 것. 시스템이 무엇을 하는지를 나타낼 수 있는 가장 좋은 방법.
- 객체 지향 분석 설계 단계
  1. Actor를 찾음 (Actor: 시스템을 사용하는 목적이 같은 사용자들의 집합) Ex) 글쓰기 사용자, 읽기 사용자, 운영자
  2. Actor들이 사용하는 시스템의 기능을 찾음 (=Use Case)
- 게시글 작성하기
  - 아래와 같은 일련의 단계가 Use Case가 됨.
  - 게시글을 작성하기 위해 게시글 작성자는 시스템과 아래와 같이 상호작용을 한다.

| 사용자                     | 시스템                       |
| -------------------------- | ---------------------------- |
| 1. 제목을 입력한다.        | 2. 제목의 유효성을 조사한다. |
| 3. 본문을 입력한다.        | 4. 본문의 유효성을 조사한다. |
| 5. 글 작성하기를 요청한다. | 6. 글을 저장한다.            |



#### 3. Architecture Exposes Usage

- 아키텍처는 사용법에 대해서 설명
- 일반적으로 아키텍처라고 일컫는 "MVC 구조만 있는 웹 시스템"은 아키텍처가 아님.
  - Use case를 숨기고 Delivering 매커니즘(MVC)만 노출
  - 중요한 것은 Delivering 매커니즘이 아니라 Use case. 시스템이 무엇을 하는 건지를 나타 내어야 함.
  - Use Case는 Delivering 매커니즘에 decoupled되어야 한다. 
  - UI, DB, Framework, Tools 등에 대한 결정들이 Use case와는 완전히 decouple되어야 한다.
- Use case should stand alone



#### 4. Deferring Decisions - 결정 미루기

- 좋은 아키텍처는 FW, WAS, UI 등과 같은 stuff들에 대한 결정을 연기하는 것을 허용.
  - stuff에 대한 결정은 연기 될 수 있어야 하고, 연기 되어야만 한다. 
  - 좋은 아키텍처의 주요한 목적 중 하나
- 시간이 지날 수록 결정을 위한 정보가 풍부해짐



#### 5-1. Deferring Decisions examples - FitNesse

> 개발을 시작할 때 일반적으로 DB부터 생각한다. MySQL을 선정하고, 스키마를 짜고, 테이블을 만들고.. 이러한 방식은 지양되어야 한다. 개발자 입장에서 data부터 생각하고 시작하면 꼬인다. 반드시 절차지향적으로 가게 된다. 아키텍처를 Usecase에 집중해야 한다. (시스템의 의도)

FitNesse 는 WIKI 이며, acceptance test tool 로 개발자가 아닌 비지니스 피플들이 위키 스타일로 테스트를 작성할 수 있게 해줌.

- 위키 저장을 위해 MySQL을 생각
- 무언가를 하기 전에 DB를 먼저 기동하고 스키마를 개발해야한다고 생각
- 그러나 사실 이게 바로 필요하진 않았다.
- 위키 텍스트를 html로 변환하는 것에 초점
  - 파싱, 번역하는 코드는 DB 없이도 개발할 수 있다.
- WikiPage 라는 중요한 추상화에 집중
  - wiki text를 가짐
  - toHtml: wiki text를 html로 parse
  - Save: wiki text를 DB에 저장
- mocking
  - override save (DB 사용없이 use case에 집중)
  - do nothing
- 하나 이상의 페이지를 사용해야 하는 경우 발생
  - DB가 실제로 필요한 상황
- InMemory Page - Test Double
- 결국에는 persistence 없이는 구현할 것이 없어졌다.
  - 생각해 낼 수 있는 새로운 테스트는 항상 DB가 필요.
  - DB를 기동하고 스키마를 만들고 DB에 위키 페이지를 저장하고 읽는 기능을 구현.
- FileSystemPage Test Double



#### 5-2. Summary

- Flat File system이 충분히 좋았기 때문에 MySQL은 사용하지 않았음.
- 주요한 아키텍처 결정을 연기했다 (Deffering Decision)
  
  - 실제 DB는 사용 하지 않았음 (더 간단하고 충분히 좋은 해결책이 있었기에)
- 최종적으로 고객사에서 모든 데이터는 실제 DB에 넣어야 한다는 정책이 있어서 그렇게 했다.
  - MySqlPage를 추가.

  

#### 6. Central Abstraction - 핵심 추상화

- 많은 아키텍트는 DB를 core abstraction 이라고 생각함.
- DB가 동작하고 스키마가 준비되기 전에는 어떠한 생각도, 작업도 시작하지 않음.
- 핵심적인 추상화는 Fitness 프로젝트에서 **WikiPage**이다.
  - DB는 연기할 수 있는 Detail로 간주.
  - 좋은 아키텍처는 Tool, FW로 구성되는 것이 아니다.
  - 좋은 아키텍처는 UI, WAS, DB, FW 등과 같은 디테일에 대한 결정을 연기 할 수 있도록 해준다.
- 어떻게 하면 이렇게 연기할 수 있나?
  
  - 연기하고 싶은 것(도메인 로직)에서 구조를 decouple, irrelevant하게 설계.
- 어떻게 하면 Tool, FWs, DB에서 decouple하나?
  - 아키텍처 측면에서 SW 환경이 아닌 Use Case에 집중하라.

  

#### 7. Conclusion

- 아키텍처를 use case 에 집중
- UI, WAS, DB, FW 등과 같은 다른 시스템 컴포넌트에 대한 결정을 미룰 수 있음.
- 이러한 연기는 우리의 선택을 최대한 오래 열어 둘 수 있음.
- 이말은 필요에 따라 결정을 변경할 수 있다는 것이다.
- 프로젝트 진행 중에 충분한 정보가 생김에 따라, undo에 대한 비용 없이 여러 번 변경 할 수 도 있다.


