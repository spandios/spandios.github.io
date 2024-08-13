---
layout: post
title: Factory Method Pattern
date: 2024-06-12 08:00 +0900
img_path: /assets/images/
category: design-pattern
tags: [ design-pattern, 객체 생성 ]
---


# 팩터리 메서드 패턴

팩터리 메서드 패턴은 객체 생성 패턴 중의 하나로, 객체 생성을 특정 서브 클래스에서 결정하도록 하는 패턴이다.

즉, 클라이언트가 직접 객체 생성하는게 아닌 공장 클래스에 위임함으로써 객체 생성의 책임을 분리 할 수 있다.

또한, 객체 생성 전 후로 추가적인 로직이 필요한 경우도 팩터리 메서드 패턴을 사용해 효과적으로 처리할 수 있다.

![]({{site.url}}/assets/images/factory-method-1.png)


위의 그림은 팩터리 메서드 패턴의 구조를 나타낸다. 

- Creator: 최상위 클래스로 팩터리 메서드를 추상화한 클래스(인터페이스)이다. 
  - 팩터리 메서드(createProduct): 객체 생성을 위한 메서드로, 서브 클래스에서 구현해야 한다.
  - 객체 생성 관련 메서드(someOperation): 객체 생성 전 후로 추가적인 로직이 필요한 경우에 사용한다.
- ConcreteCreator: Creator를 구현한 클래스로, 객체 생성을 위한 팩터리 메서드를 실제로 구현한다. 실제 객체 마다 하나씩 구현한다. 
- Product: 팩터리 메서드를 통해 생성되는 객체의 인터페이스다
- ConcreteProduct: Product 인터페이스를 구현한 클래스로, 실제 객체를 생성한다.



# 패턴 적용 전 

초기 선박 회사가 하나의 선박을 생산하는 코드를 작성한다고 가정해보자.

```kotlin
// 선박 클래스로 선박의 이름, 색상, 로고를 가지고 있다.
class Ship {

  lateinit var name: String

  lateinit var color: String

  lateinit var logo: String


  override fun toString(): String {
    return "Ship{" +
      "name='" + name + '\'' +
      ", color='" + color + '\'' +
      ", logo='" + logo + '\'' +
      '}'
  }
}

// 선박을 생성하는 팩토리 클래스로, 선박을 생성하는 메서드를 가지고 있다.
object ShipFactory {
  fun createShip(name: String?, email: String?): Ship {
    // validate
    require(!name.isNullOrBlank()) { "배 이름을 지어주세요" }
    require(!email.isNullOrBlank()) { "연락처를 남겨주세요." }

    prepareFor(name)

    val ship = Ship()
    ship.name = name

    // 이름에 따라 로고를 다르게 설정
    if (name.equals("whiteship", ignoreCase = true)) {
      ship.logo = "\uD83D\uDEE5️"
    } else if (name.equals("blackship", ignoreCase = true)) {
      ship.logo = "⚓"
    }

    // 이름에 따라 색상을 다르게 설정
    if (name.equals("whiteship", ignoreCase = true)) {
      ship.color = "whiteship"
    } else if (name.equals("blackship", ignoreCase = true)) {
      ship.color = "black"
    }

    // 이메일을 보내는 로직
    sendEmailTo(email, ship)

    return ship
  }

  private fun prepareFor(name: String) {
    println("$name 만들 준비 중")
  }

  private fun sendEmailTo(email: String, ship: Ship) {
    println(ship.name + " 다 만들었습니다.")
  }
}

// 클라이언트 코드
fun main(){
  val ship = ShipFactory.createShip("BasicShip", "wndudpower@gmail.com")
  println(ship)
}


```

위의 코드는 다른 종류의 선박이 추가되면 추가될수록 if문을 추가해야하고 이는 코드의 복잡성을 증가시킨다.

또 만약 선박 생성의 로직 자체가 변경되면 클라이언트 코드까지 수정해야한다. 이는 객체 생성의 책임이 팩토리 클래스에 있지 않고 클라이언트에 있기 때문이다. 이것은 OCP(Open-Closed Principle)를 위반한다.

## 패턴 적용

팩터리 메서드 패턴을 사용하면 객체 생성의 책임 분리 및 객체 생성 전 후로 추가적인 로직을 효율적으로 처리할 수 있다.

만들 선박 종류가 어선과 화물선으로 늘어났다고 가정해보자.

먼저 전에 Ship 클래스(Product)는 상속 원형으로 두고 실제 구현 객체(ConcreteProduct) 어선과 화물선 클래스를 정의한다.

```kotlin
class FishShip : Ship() {
  init {
    name = "FishShip"
    logo = "🐟"
    color = "blue"
  }
}

class CargoShip : Ship() {
  init {
    name = "CargoShip"
    logo = "🚚"
    color = "gray"
  }
}

```

이 후 팩토리 클래스(Creator)를 interface로 ShipFacotry로 정의한다. 

`buildShip` 메서드를 통해 객체를 생성하고, 객체 생성 전후로 추가적인 로직을 처리하는 메서드를 정의한다. 

여기서 `buildShip`은 기본 메서드로 구현되어 전체 로직을 담당한다. 

또, 기존에 객체 생성 전후에 추가적인 로직을 처리하는 부분인 `prepareFor`, `validate`, `sendEmailTo`를 메서드로 분리한다. 

실제 구체적인 객체 생성 메서드인 `createShip` 메서드와 이메일을 전송하는 `sendEmailTo`메서드를 서브 팩토리 클래스에서 구현하도록 추상화한다. 

`validate`와 `prepareFor` 메서드 또한 기본 구현으로 공통된 로직을 처리하도록 한다. 이 공통 로직은 서브 클래스에서 재정의할 필요가 없기 때문에 private로 선언한다.   

이 구현의 핵심은 `buildShip` 메서드가 미리 뼈대를 정의해 로직의 흐름을 제어하며, 추가적으로 서브 클래스에서 재정의한 메서드를 호출해 로직을 완성한다.

```kotlin

interface ShipFactory {
  fun buildShip(name: String, email: String?): Ship? {
    validate(name, email)
    prepareFor(name)
    val ship = createShip()
    sendEmailTo(email, ship)
    return ship
  }

  fun sendEmailTo(email: String, ship: Ship)

  fun createShip(): Ship

  private fun validate(name: String?, email: String?) {
    require(!(name == null || name.isBlank())) { "배 이름을 지어주세요." }
    require(!(email == null || email.isBlank())) { "연락처를 남겨주세요." }
  }

  private fun prepareFor(name: String) {
    println("$name 만들 준비 중")
  }
}

```

이후 ConcreteCreator 클래스를 정의한다. ConcreteCreator 클래스는 Creator 인터페이스를 구현하고(ShipFactory), 실제 객체 생성을 담당하는 createShip 메서드를 구현한다.

```kotlin

class FishShipFactory : ShipFactory {
  override fun sendEmailTo(email: String, ship: Ship) {
    println(ship.name + " 다 만들었습니다.")
  }

  override fun createShip(): Ship {
    return FishShip()
  }
}

class CargoShipFactory : ShipFactory {
  override fun sendEmailTo(email: String, ship: Ship) {
    println(ship.name + " 다 만들었습니다.")
  }

  override fun createShip(): Ship {
    return CargoShip()
  }
}
```

sendEmailTo 메서드는 추상 메서드로 정의되어 있으므로 서브 클래스에서 구현해야 한다. 현재 두 Factory 모두 `sendEmailTo`를 똑같은 로직으로 구현하고 있다. 

공통 ShipFactory를 만들어서 sendEmailTo 메서드를 구현하고 FishShipFactory, CargoShipFactory는 이를 상속받아 사용할 수 있다.

```kotlin
abstract class DefaultShipFactory : ShipFactory {
  override fun sendEmailTo(email: String, ship: Ship) {
    println(ship.name + " 다 만들었습니다.")
  }
}

class FishShipFactory : ShipFactory() {
  override fun createShip(): Ship {
    return FishShip()
  }
}

class CargoShipFactory : ShipFactory() {
  override fun createShip(): Ship {
    return CargoShip()
  }
}

```

이후 클라이언트 코드에서는 필요한 객체를 직접 생성하는게 아닌 팩토리 클래스를 통해 객체를 생성한다.

```kotlin
object Client {
  @JvmStatic
  fun main(args: Array<String>) {
    val fishShipFactory = FishShipFactory()
    val fishShip = fishShipFactory.buildShip("주영 어선", "wndudpower@gmail.com")
    println(fishShip)

    val cargoShipFactory = CargoShipFactory()
    val cargoShip = cargoShipFactory.buildShip("주영 화물선", "wndudpower@gmail.com")
    println(cargoShip)
  }
}
```


# 팩터리 메서드 패턴의 장단점

- 장점
  - 객체 생성의 책임을 팩터리 클래스에 위임함으로써 객체 생성의 책임을 분리할 수 있다.
  - 객체 생성 전후로 추가적인 로직이 필요한 경우 팩터리 메서드 패턴을 사용해 효과적으로 처리할 수 있다.
  - 객체 생성의 변화에 대응하기 쉽다. 새로운 객체가 추가되거나 객체 생성 로직이 변경되어도 클라이언트 코드를 수정할 필요가 없다.

- 단점
  - 팩터리 클래스를 사용하면 객체 생성을 위한 별도의 클래스가 추가되므로 코드의 복잡성이 증가할 수 있다.



## Reference

- 코딩으로 학습하는 GoF의 디자인 패턴 - 백기선
- https://inpa.tistory.com/entry/GOF-%F0%9F%92%A0-%ED%8C%A9%ED%86%A0%EB%A6%AC-%EB%A9%94%EC%84%9C%EB%93%9CFactory-Method-%ED%8C%A8%ED%84%B4-%EC%A0%9C%EB%8C%80%EB%A1%9C-%EB%B0%B0%EC%9B%8C%EB%B3%B4%EC%9E%90





