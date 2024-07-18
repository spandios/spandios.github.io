---
layout: post
title: Mediator Pattern
date: 2024-07-17 08:00 +0900
img_path: /assets/images/
category: design-pattern
tags: [ design-pattern, 구조 관련 디자인 패턴 ]
---

## 중재자 패턴 (Mediator Pattern)

여러 객체들 간의 의사소통하는 방법을 추상화 시켜 의존성간의 결합도를 낮추는 패턴이다. 

객체 지향 프로그래밍에서 점점 비즈니스 로직이 복잡해지면 객체간의 상호작용하는 부분이 많아지고 복잡해진다. 이때 객체간의 상호작용을 중재자 객체를 통해 처리하면 객체간의 결합도를 낮출 수 있다. 

## 패턴 구조

![]({{site.url}}/assets/images/mediator.png)

1. Mediator: Colleague 객체들 간의 상호작용을 중재하는 인터페이스를 정의
2. ConcreteMediator: Mediator 구현체로 Colleague 객체들을 참조하고 중재하는 역할을 함
3. Colleague: Mediator를 통해 다른 Colleague와 상호작용하기 위한 인터페이스를 정의
4. ConcreteColleague: Colleague 인터페이스를 구현하는 클래스

중요한 점은 Colleague는 Colleague끼리 참조하지 않는 선은 보이지 않는다. 대신 Mediator와 Colleague가 서로 참조하고 있어서 Mediator를 통해 상호작용한다.


## 패턴 적용 전

비행기가 이착륙하는 것을 생각해보자. 다만 여기엔 관제탑이 없어서 비행기들간에 소통을 직접 해야한다.   

```kotlin
class Airplane(val name: String) {
  val nearbyAirplanes = mutableListOf<Airplane>() // 주변 비행기
    
  // 착륙 요청
  fun requestTakeoff() {
    for (airplane in nearbyAirplanes) {
      if (!airplane.isSafeForTakeoff()) {
        println("Takeoff request denied for $name due to nearby airplane: ${airplane.name}")
        return
      }
    }
    println("Takeoff request approved for $name")
  }
    
  // 이륙 요청
  fun requestLanding() {
    for (airplane in nearbyAirplanes) {
      if (!airplane.isSafeForLanding()) {
        println("Landing request denied for $name due to nearby airplane: ${airplane.name}")
        return
      }
    }
    println("Landing request approved for $name")
  }

  fun isSafeForTakeoff(): Boolean {
    // Simplified logic for demonstration
    return true
  }

  fun isSafeForLanding(): Boolean {
    // Simplified logic for demonstration
    return true
  }
    
  // 주변 비행기 추가
  fun addNearbyAirplane(airplane: Airplane) {
    nearbyAirplanes.add(airplane)
  }
}

fun main() {
  val airplane1 = Airplane("Airplane 1")
  val airplane2 = Airplane("Airplane 2")

  airplane1.addNearbyAirplane(airplane2)
  airplane2.addNearbyAirplane(airplane1)

  airplane1.requestTakeoff()
  airplane2.requestLanding()
}  

```

위의 상황에서 문제점은 비행기들이 서로 직접 소통을 해야한다는 것이다. 이는 각 객체들이 서로의 상태를 알아야 하기 때문에 이는 결합도가 높아지는 문제를 야기한다.

## 패턴 적용 후

이번에는 중재자 패턴을 적용해보자. 

먼저 Mediator 인터페이스를 정의한다. 
```kotlin

interface AirTrafficControl {
  fun requestTakeoff(airplane: Airplane)
  fun requestLanding(airplane: Airplane)
  fun isSafeForTakeoff(airplane: Airplane): Boolean
  fun isSafeForLanding(airplane: Airplane): Boolean
}
```

다음으로 ConcreteMediator를 정의한다. Mediator는 각 비행기(Colleague)들을 참조하고 있으며 비행기들간의 상호작용에 대한 로직을 수행한다.
```kotlin

class AirTrafficControlImpl : AirTrafficControl {
  private val airplanes = mutableListOf<Airplane>()

  fun addAirplane(airplane: Airplane) {
    airplanes.add(airplane)
  }

  override fun requestTakeoff(airplane: Airplane) {
    for (otherAirplane in airplanes) {
      if (otherAirplane != airplane && !isSafeForTakeoff(otherAirplane)) {
        println("Takeoff request denied for ${airplane.name} due to nearby airplane: ${otherAirplane.name}")
        return
      }
    }
    println("Takeoff request approved for ${airplane.name}")
  }

  override fun requestLanding(airplane: Airplane) {
    for (otherAirplane in airplanes) {
      if (otherAirplane != airplane && !isSafeForLanding(otherAirplane)) {
        println("Landing request denied for ${airplane.name} due to nearby airplane: ${otherAirplane.name}")
        return
      }
    }
    println("Landing request approved for ${airplane.name}")
  }

  override fun isSafeForTakeoff(airplane: Airplane): Boolean {
    // Simplified logic for demonstration
    return true
  }

  override fun isSafeForLanding(airplane: Airplane): Boolean {
    // Simplified logic for demonstration
    return true
  }
}

```

마지막으로 Colleague를 정의한다. 비행기는 Mediator를 참조하고 있으며 다른 비행기와 소통은 Mediator를 통해 수행한다.
```kotlin

class Airplane(val name: String, val airTrafficControl: AirTrafficControl) {
  init {
    airTrafficControl.addAirplane(this)
  }
  fun requestTakeoff() {
    airTrafficControl.requestTakeoff(this)
  }
  
  fun requestLanding() {
    airTrafficControl.requestLanding(this)
  }
}

fun main() {
  val airTrafficControl = AirTrafficControlImpl()
  val airplane1 = Airplane("Airplane 1", airTrafficControl)
  val airplane2 = Airplane("Airplane 2", airTrafficControl)

  airplane1.requestTakeoff()
  airplane2.requestLanding()
}

```


## 장단점

### 장점

- 컴퍼넌트 코드를 변경하지 않고 새로운 중재자를 만들어서 사용 가능하다.
- 객체간의 상호작용을 중재자를 통해 처리하기 때문에 객체간의 의존성을 낮출 수 있다.


### 단점

- 중재자 역할을 하는 클래스의 복잡도와 결합도가 증가한다.

## Reference

- 코딩으로 학습하는 GoF의 디자인 패턴 - 백기선




