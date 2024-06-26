---
layout: post
title: Builder Pattern
date: 2024-06-26 08:00 +0900
img_path: /assets/images/
category: design-pattern
tags: [ design-pattern, 객체 생성 ]
---


# 빌더 패턴

빌더 패턴은 복잡한 객체를 단계별로 생성할 수 있도록 돕는 디자인 패턴이다.

빌더 패턴은 특히 생성자가 복잡하거나 객체를 설정할 때 많은 매개변수가 필요한 경우 유용하다.

생성자로 받을 매개변수를 함수로 차곡 차곡 쌓아놓고 마지막에 통합해서 객체를 생성하는 방식이다.


![]({{site.url}}/assets/images/builder-pattern.png)

1. Builder 인터페이스: 객체 생성에 필요한 메서드들을 선언한다.
2.	ConcreteBuilder (구체적 빌더): Builder 인터페이스를 구현하며, 제품의 각 부분을 생성하는 데 필요한 메서드를 구현한다.
3.	Product (제품): 빌더에 의해 생성되는 복잡한 객체이다.
4.	Director: Builder 인터페이스를 사용하여 객체를 생성하는 과정을 관리한다.



## Reference

- 코딩으로 학습하는 GoF의 디자인 패턴 - 백기선
- https://refactoring.guru/ko/design-patterns/builder
- https://inpa.tistory.com/entry/GOF-%F0%9F%92%A0-%EB%B9%8C%EB%8D%94Builder-%ED%8C%A8%ED%84%B4-%EB%81%9D%ED%8C%90%EC%99%95-%EC%A0%95%EB%A6%AC




