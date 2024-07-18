---
layout: post
title: Mediator Pattern
date: 2024-07-17 08:00 +0900
img_path: /assets/images/
category: design-pattern
tags: [ design-pattern, 구조 관련 디자인 패턴 ]
---

## 메멘토 패턴 (Mediator Pattern)
 
캡슐화를 유지하면서 객체 내부 상태를 외부에 저장하는 패턴이다. 객체의 상태를 저장하고 복구하는 데 사용된다. 


## 패턴 구조

![]({{site.url}}/assets/images/memento.png)

1. CareTake: Originator의 상태를 저장하고 복구하는 역할을 함 
2. Originator: 현재 상태를 저장하고 복구하는 역할을 함
3. Memento: Originator의 상태를 저장하는 역할을 함

## 패턴 적용 전



## 장단점

### 장점

- 컴퍼넌트 코드를 변경하지 않고 새로운 중재자를 만들어서 사용 가능하다.
- 객체간의 상호작용을 중재자를 통해 처리하기 때문에 객체간의 의존성을 낮출 수 있다.


### 단점

- 중재자 역할을 하는 클래스의 복잡도와 결합도가 증가한다.

## Reference

- 코딩으로 학습하는 GoF의 디자인 패턴 - 백기선




