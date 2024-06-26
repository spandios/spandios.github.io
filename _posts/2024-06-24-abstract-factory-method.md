---
layout: post
title: Abstract Factory Pattern
date: 2024-06-24 08:00 +0900
img_path: /assets/images/
category: design-pattern
tags: [ design-pattern, 객체 생성 ]
---


## 추상 팩터리 패턴

추상 팩토리 패턴은 팩토리 메서드 패턴과 모양과 효과는 비슷하다.

둘 다 구체적인 객체 생성 과정을 추상화한 인터페이스를 제공해 객체 생성의 책임을 팩토리 클래스에 위임한다.

하지만 목적이 다르다. 팩토리 메서드 패턴은 구체적인 객체 생성 과정을 다른 서브(구체)클래스로 위임하는 것이 목적이다.

반면 `추상 팩토리 패턴`은 연관성이 있는 객체 군이 여러개 있을 경우 이들을 묶어 추상화하며 팩토리 객체에서 집합으로 묶은 객체 군을 구현화 하는 생성 패턴이다

팩토리 메서드 패턴에선 하나의 팩토리가 하나의 객체를 생성하고 생성 전후로 후속 처리를 하는 반면 추상 팩토리 패턴은 하나의 팩토리가 여러 객체를 생성한다.


![]({{site.url}}/assets/images/abstract-factory-pattern.png)


위의 그림은 추상 팩토리 패턴의 구조를 나타낸다. 

AbstractFactory: 추상 팩토리 인터페이스로, 관련된 객체 군을 생성하는 메서드를 선언한다.


ConcreteFactory: AbstractFactory를 구현한 클래스로, 객체 군을 생성하는 메서드를 실제로 구현한다.

AbstractProduct: 객체 군의 인터페이스로, 객체 군의 공통 인터페이스를 선언한다.


ConcreteProduct: AbstractProduct를 구현한 클래스로, 객체 군의 실제 객체를 생성한다.


Client: AbstractFactory와 AbstractProduct 인터페이스만을 의존해 실제 객체를 생성하고 사용하는 곳이다. 

이제 추상 팩토리 패턴 예시를 살펴보려한다.

## 브랜드 컴퓨터 부품 예시

완제품 컴퓨터를 생산하는 회사가 있다고 가정해보자. 이 회사는 브랜드별로 컴퓨터 부품으로 다르게 생산한다. 
삼성 브랜드는 CPU로 Intel, Memory로 Samsung을 사용하고, LG 브랜드는 CPU로 AMD, Memory로 Hynix를 사용한다고 가정해보자.

```kotlin

interface PCComponent {}

class Computer {
  lateinit var cpu: CPU
  lateinit var memory: Memory
}

abstract class CPU : PCComponent {}

abstract class Memory : PCComponent {}

class IntelCPU(private val model: String) : CPU() {}

class AmdCPU(private val model: String) : CPU() {}


class SamsungMemory(private val model: String) : Memory() {
}

class HynixMemory(private val model: String) : Memory() {
}

```

이제 브랜드별로 컴퓨터 부품을 생성하는 추상 팩토리 클래스를 정의한다.

```kotlin

interface PCFactory {
  fun makeComputer(): Computer
  fun createCPU(): CPU
  fun createMemory(): Memory
}

class SamsungComputerFactory : PCFactory {

  override fun makeComputer(): Computer {
    val computer = Computer()
    computer.cpu = createCPU()
    computer.memory = createMemory()
    return computer
  }

  override fun createCPU(): CPU {
    return IntelCPU("i7")
  }

  override fun createMemory(): Memory {
    return SamsungMemory("DDR4")
  }
}

class LgComputerFactory : PCFactory {
  
  override fun makeComputer(): Computer {
    val computer = Computer()
    computer.cpu = createCPU()
    computer.memory = createMemory()
    return computer
  }
  
  
  override fun createCPU(): CPU {
    return AmdCPU("Ryzen 7 5800X")
  }

  override fun createMemory(): Memory {
    return HynixMemory("DDR4")
  }
}

```

이제 브랜드 별로 컴퓨터를 생성해본다.

```kotlin

fun main() {
  val samsungFactory = SamsungComputerFactory()
  val lgFactory = LgComputerFactory()
  
  val samsungComputer = samsungFactory.makeComputer()
  samsungComputer.cpu // IntelCPU(model=i7)
  samsungComputer.memory // SamsungMemory(model=DDR4)
  
  val lgComputer = lgFactory.makeComputer()
  lgComputer.cpu // AmdCPU(model=Ryzen 7 5800X)
  lgComputer.memory // HynixMemory(model=DDR4)
  
}

```

만약 새로운 브랜드가 추가되어도 PCFactory를 구현하는 클래스만 추가하면 된다. 

```kotlin

class DellComputerFactory : PCFactory {
  
  override fun makeComputer(): Computer {
    val computer = Computer()
    computer.cpu = createCPU()
    computer.memory = createMemory()
    return computer
  }
  
  override fun createCPU(): CPU {
    return IntelCPU("i9")
  }

  override fun createMemory(): Memory {
    return HynixMemory("DDR4")
  }
}

```

```kotlin

fun main() {
  val dellFactory = DellComputerFactory()
  val dellComputer = dellFactory.makeComputer()
  dellComputer.cpu // IntelCPU(model=i9)
  dellComputer.memory // HynixMemory(model=DDR4)
}

```

하지만 추상 팩토리의 인터페이스가 변경되면 모든 구현체가 변경되어야 한다는 단점이 있다. 만약 기존 부품에서 메인보드가 추가된다면 모든 구현체에서 메인보드를 생성하는 메서드를 추가해야한다.


## 선박 예시

저번 팩토리 메서드 패턴에서 선박 클래스를 만들었다.

이제 선박에는 Anchor와 Wheel이라는 부품이 추가되었다고 가정해보자.

```kotlin

interface Anchor {

}

interface Wheel {

}

class BasicAnchor : Anchor {

}


class BasicWheel : Wheel {

}

```

```kotlin
class Ship {
  
  ... // 생략
  
  lateinit var anchor: Anchor // 추가된 부품
  
  lateinit var wheel: Wheel // 추가된 부품
}

```


이제 지난 어선 클래스를 생성하는 팩토리 메서드 패턴의 factory 클래스에서 anchor와 wheel을 추가한 코드는 다음과 같다.

```kotlin
class FishShipFactory : ShipFactory {
  
  override fun createShip(): Ship {
    val fishShip =  FishShip()
    fishShip.anchor = BasicAnchor()
    fishShip.wheel = BasicWheel()
    return fishShip
  }
}

```

위의 코드는 만약 어선의 부품인 anchor와 wheel이 바뀔 때마다 어선 클래스를 생성하는 factory 클래스에서도 변경해야한다.

이 때 부품에 해당하는 부분을 위해 추상 팩토리를 사용하면 이러한 문제를 해결할 수 있다. 


## 추상 팩토리 패턴 적용하기  

추상 팩토리 패턴을 적용하기 위해 Anchor와 Wheel을 생성하는 추상 팩토리 클래스를 정의한다.

```kotlin

interface ShipComponentFactory {
  fun createAnchor(): Anchor
  fun createWheel(): Wheel
}

```

이후 구체적인 부품을 생성하는 클래스를 정의한다.

```kotlin

class FishBasicComponentFactory : ShipComponentFactory {
  override fun createAnchor(): Anchor {
    return BasicAnchor()
  }

  override fun createWheel(): Wheel {
    return BasicWheel()
  }
}

```

이제 어선 클래스를 생성하는 factory 클래스에서 부품을 생성하는 부분은 추상 팩토리 클래스로 위임한다. 

이를 위해 Factory 생성자에 ShipComponentFactory를 주입한다.


```kotlin

class FishShipFactory(private val componentFactory: ShipComponentFactory) : ShipFactory {

  override fun createShip(): Ship {
    val fishShip = FishShip()
    fishShip.anchor = componentFactory.createAnchor()
    fishShip.wheel = componentFactory.createWheel()
    return fishShip
  }
}

```

이제 어선의 부품이 바뀌어도 어선 클래스를 생성하는 factory 클래스에서는 변경할 필요가 없다.

어선은 기본적인 BasicAnchor와 BasicWheel로 선박된다고 가정해보자.


```kotlin

fun main() {
  val basicComponentFactory = FishBasicComponentFactory()
  val fishShipFactory = FishShipFactory(basicComponentFactory)
  val fishShip = fishShipFactory.createShip()
  println(fishShip) // 기본 어선 부품을 가진 어선 생성 완료
}


```

하지만 화물선은 기본적인 부품에서 더 나아가 고급 부품을 사용하게 된다면 고급 부품을 생성하는 팩토리 클래스를 만들고 교체만 해주면 된다. 

```kotlin

class AdvancedAnchor : Anchor {

}

class AdvancedWheel : Wheel {

}

class CargoAdvancedComponentFactory : ShipComponentFactory {
  override fun createAnchor(): Anchor {
    return AdvancedAnchor()
  }

  override fun createWheel(): Wheel {
    return AdvancedWheel()
  }
}

```


```kotlin
fun main() {
  val advancedComponentFactory = CargoAdvancedComponentFactory()
  val cargoShipFactory = CargoShipFactory(advancedComponentFactory)
  val cargoShip = cargoShipFactory.createShip()
  println(fishShip) // 고급 어선 부품을 가진 어선 생성 완료!
}
```



## 추상 팩토리 패턴의 장단점

- 장점
  - 객체 생성을 위한 별도의 클래스를 사용하기 때문에 객체 생성을 위한 코드가 클라이언트 코드로부터 분리된다.
- 단점
  - 객체 생성을 위한 별도의 클래스가 추가되므로 코드의 복잡성이 증가할 수 있다.
  - 추상 팩토리의 인터페이스가 변경되면 모든 구현체가 변경되어야 한다.

## 요약

추상 팩토리 패턴은 연관된 객체 군을 생성하는 패턴이다.

팩토리 메서드 패턴과 비슷하지만 팩토리 메서드 패턴은 하나의 객체를 생성하는 반면 추상 팩토리 패턴은 여러 객체를 생성한다.

추상 팩토리 패턴을 사용하면 객체 생성을 위한 코드가 클라이언트 코드로부터 분리되어 객체 생성을 위한 별도의 클래스를 사용할 수 있다.




## Reference

- 코딩으로 학습하는 GoF의 디자인 패턴 - 백기선
- https://refactoring.guru/ko/design-patterns/abstract-factory




