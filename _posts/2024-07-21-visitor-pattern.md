---
layout: post
title: Visitor Pattern
date: 2024-07-20 08:00 +0900
img_path: /assets/images/
category: design-pattern
tags: [ design-pattern, 구조 관련 디자인 패턴 ]
---

## 방문자 패턴 (Visitor Pattern)

객체 구조를 변경하지 않고도 새로운 동작을 추가할 수 있게 해주는 패턴이다. 

방문자 패턴은 객체의 구조와 동작을 분리하여 객체 구조를 변경하지 않고 새로운 동작을 추가할 수 있다.

주로 객체 구조는 안정적이지만 동작 부분에서 변경점이 많은 상황에 사용하면 유용하다.

## 패턴 구조

![]({{site.url}}/assets/images/visitor.png)

1. Visitor : 각 ConcreteElement에 대한 visit 메서드를 선언하는 인터페이스이다. visit메서드는 element를 인자로 받아 element의 타입에 따라 다른 동작을 수행한다.
2. Element: accept 메서드를 선언하는 인터페이스이다. accept는 Visitor를 인자로 받아들이는 메서드이다. 즉, Visitor를 받아들이는 메서드를 가지고 있다.

## 패턴 적용 전

강아지와 고양이를 표현하기 위해 Animal 클래스를 만들어보자. Animal 클래스는 추상 클래스로 `sound`와 `eat`메서드를 추상 메서드로 정의한다. 

```kotlin

// 동물 클래스들
abstract class Animal {
    abstract fun sound()
    abstract fun eat()
}

class Dog : Animal() {
    override fun sound() {
        println("개가 짖습니다.")
    }
    override fun eat() {
        println("개가 사료를 먹습니다.")
    }
}

class Cat : Animal() {
    override fun sound() {
        println("고양이가 야옹합니다.")
    }
    override fun eat() {
        println("고양이가 생선을 먹습니다.")
    }
}

// 사용
fun main() {
    val animals = listOf(Dog(), Cat())
    for (animal in animals) {
        animal.sound()
        animal.eat()
    }
}


```
위의 코드에서 새로운 동작인 `move`을 추가한다고 가정해보자. 그러면 모든 Animal 서브클래스를 수정해야만 한다. 이는 확장성이 부족하다고 볼 수 있다.

또 유사한 동작임에도 모든 서브 클래스에서 동일한 코드를 작성해야 하는 문제가 있다.

```kotlin

abstract class Animal {
  abstract fun sound()
  abstract fun eat()
  abstract fun move()
}

class Dog : Animal() {
  override fun sound() {
    println("개가 짖습니다.")
  }
  override fun eat() {
    println("개가 사료를 먹습니다.")
  }

  override fun move() {
    println("개가 걷습니다.")
  }
}

class Cat : Animal() {
  override fun sound() {
    println("고양이가 야옹합니다.")
  }
  override fun eat() {
    println("고양이가 생선을 먹습니다.")
  }

  override fun move() {
    println("고양이가 걷습니다.")
  }
}

```

## 패턴 적용 후

방문자 패턴을 적용하여 동작을 추가해보자.

먼저 Visitor 인터페이스를 정의한다. 이 인터페이스는 각 ConcreteElement에 대한 visit 메서드를 선언한다.


```kotlin

// Visitor 인터페이스
interface AnimalVisitor {
    fun visit(dog: Dog)
    fun visit(cat: Cat)
}
```

이제 Element 인터페이스를 정의한다. Element 인터페이스는 기존 객체에 새로운 동작을 추가할 수 있도록 accept 메서드를 선언한다.

다음은 Element 인터페이스를 구현한 Dog와 Cat 클래스이다. 각 `accept`메서드는 Visitor를 인자로 받아들이고 Visitor의 visit 메서드를 호출하는 것을 볼 수 있다. 이 때 Visitor의 visit 메서드에 자신을 인자로 넘긴다.

```kotlin
// Element 인터페이스
interface Animal {
    fun accept(visitor: AnimalVisitor)
}

// ConcreteElement 클래스들
class Dog : Animal {
    override fun accept(visitor: AnimalVisitor) {
        visitor.visit(this)
    }
}

class Cat : Animal {
    override fun accept(visitor: AnimalVisitor) {
        visitor.visit(this)
    }
}
```

다음은 실제 행동을 구현하는 ConcreteVisitor 클래스들이다. 각 ConcreteVisitor 클래스는 Visitor 인터페이스를 구현하고 각 ConcreteElement에 대한 visit 메서드를 구현한다.

```kotlin

// ConcreteVisitor 클래스들
class SoundVisitor : AnimalVisitor {
    override fun visit(dog: Dog) {
        println("개가 짖습니다.")
    }
    override fun visit(cat: Cat) {
        println("고양이가 야옹합니다.")
    }
}

class EatVisitor : AnimalVisitor {
    override fun visit(dog: Dog) {
        println("개가 사료를 먹습니다.")
    }
    override fun visit(cat: Cat) {
        println("고양이가 생선을 먹습니다.")
    }
}


class MoveVisitor : AnimalVisitor {
  override fun visit(dog: Dog) {
    println("개가 걷습니다.")
  }
  override fun visit(cat: Cat) {
    println("고양이가 걷습니다.")
  }
}

```

실제 사용은 다음과 같다. 기존 객체에 행동 코드를 추가하지 않고도 새로운 동작을 추가할 수 있다.

```kotlin
// 사용
fun main() {
    val animals = listOf(Dog(), Cat())
    val soundVisitor = SoundVisitor()
    val eatVisitor = EatVisitor()
    val moveVisitor = MoveVisitor() 

    for (animal in animals) {
        animal.accept(soundVisitor)
        animal.accept(eatVisitor)
        animal.accept(moveVisitor)
    }
}
```



## 장단점

### 장점

- 새로운 동작을 쉽게 추가할 수 있다.
- 관련 동작을 한 곳에서 관리할 수 있다.
- 객체 구조와 동작을 분리해 단일 책임 원칙을 지킬 수 있다.

### 단점

- 복잡성 증가
- 새로운 ConcreteElement를 추가할 때 Visitor 인터페이스와 모든 ConcreteVisitor 클래스를 수정해야 한다.




