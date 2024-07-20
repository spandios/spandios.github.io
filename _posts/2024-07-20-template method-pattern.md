---
layout: post
title: Template Method Pattern
date: 2024-07-20 08:00 +0900
img_path: /assets/images/
category: design-pattern
tags: [ design-pattern, 구조 관련 디자인 패턴 ]
---

## 템플릿 메소드 패턴 (Template Method Pattern)

이름에서 알 수 있듯이 상위 클래스에서 템플릿 메서드로 알고리즘의 구조를 정의하고 그 중 일부 단계의 알고리즘은 서브 클래스에서 구현을 위임하는 패턴이다.

공통되거나 변경이 없는 로직은 슈퍼 클래스에서 구현하고 변경이 필요한 로직은 서브 클래스에서 구현한다.

추가적으로 Hook 메서드란 것도 있는데 이는 서브 클래스에서 선택적으로 오버라이딩할 수 있는 기본 구현을 가진 메서드이다.

## 패턴 구조

![]({{site.url}}/assets/images/template method.png)

1. AbstractClass : 상위 클래스로 알고리즘 구조를 정의하되 일부 단계는 하위 클래스가 구현하도록 한다.
2. ConcreteClass: AbstractClass를 상속받아 구현한 클래스로 템플릿 메소드를 구현한다.

## 패턴 적용 전

예를 들어 커피와 차를 만드는 코드를 작성해보자.

```kotlin
class Coffee {
  fun prepare(addCondiments: Boolean) {
    boilWater()
    brewCoffee()
    pourInCup()
    if (addCondiments)
      addSugarAndMilk()
  }

  private fun boilWater() {
    println("물을 끓입니다.")
  }

  private fun brewCoffee() {
    println("커피를 우려냅니다.")
  }

  private fun pourInCup() {
    println("컵에 따릅니다.")
  }

  private fun addSugarAndMilk() {
    println("설탕과 우유를 추가합니다.")
  }
}

class Tea {
  fun prepare(addCondiments: Boolean) {
    boilWater()
    brewTea()
    pourInCup()
    if (addCondiments)
      addLemon()
  }

  private fun boilWater() {
    println("물을 끓입니다.")
  }

  private fun brewTea() {
    println("차를 우려냅니다.")
  }

  private fun pourInCup() {
    println("컵에 따릅니다.")
  }

  private fun addLemon() {
    println("레몬을 추가합니다.")
  }
}

fun main(){
  val coffee = Coffee()
  val tea = Tea()

  coffee.prepare(true)
  tea.prepare(false)
}
``` 

이 코드의 문제점은 코드가 중복되는 점이 많아 공통 로직을 변경할 때마다 여러 클래스를 반복적으로 수정해야한다는 것이다.

또 새로운 음료가 추가될 때마다 기존 코드를 복사하고 수정해야한다. 

이러한 문제는 템플릿 메서드 패턴을 적용해 해결할 수 있다.

## 패턴 적용 후

먼저 음료를 만드는 공통 클래스를 만든다. 

`prepare`메서드는 템플릿 메서드로 기본적인 알고리즘 뼈대가 구현되어 있고 `brew`와 `addCondiments`는 추상 메서드로 선언된다. 

brew와 addCondiments는 서브 클래스에서 구현해야하는 메서드인 것이다. 이외 boilWater와 pourInCup은 공통 메서드로 추상클래스에서 기본 구현된다.

`customerWantsCondiments`는 훅 메서드로 서브 클래스에서 선택적으로 오버라이딩할 수 있는 메서드이다. 

```kotlin
  abstract class Beverage {
    // 템플릿 메서드
    fun prepare() {
      boilWater()
      brew()
      pourInCup()
      if (addCondiments())
        addCondiments()
    }

    abstract fun brew()
    abstract fun addCondiments()
  
    private fun boilWater() {
      println("물을 끓입니다.")
    }

    private fun pourInCup() {
      println("컵에 따릅니다.")
    }
        
    // Hook 메서드
    open fun customerWantsCondiments(): Boolean = true
  }
```

이 후 Concrete Class인 Coffee와 Tea를 구현해보자.

```kotlin

  class Coffee : Beverage() {
    override fun brew() { // 추상 메서드 구현
      println("커피를 우려냅니다.")
    }

    override fun addCondiments() { // 추상 메서드 구현
      println("설탕과 우유를 추가합니다.")
    }
  }

  class Tea : Beverage() {
    override fun brew() { // 추상 메서드 구현
      println("차를 우려냅니다.")
    }

    override fun addCondiments() { // 추상 메서드 구현
      println("레몬을 추가합니다.")
    }

    override fun customerWantsCondiments(): Boolean = false // Hook 메서드 오버라이딩
  }

  fun main() {
    val coffee = Coffee()
    val tea = Tea()

    coffee.prepare()
    tea.prepare() 
  }

```

## 장단점

### 장점

- 공통 로직을 상위 클래스에 정의하기 때문에 중복을 줄이고 재사용성을 높인다.
- 기존 코드를 수정하지 않고도 새로운 구현을 추가할 수 있다.
- 제어의 역전의 원리를 사용하여 확장에 용이하다.

### 단점

- 상속을 사용하기 때문에 상위 클래스의 변경이 하위 클래스에 영향을 미칠 수 있다.


## 템플릿 콜백 패턴

템플릿 콜백 패턴은 템플릿 메서드 패턴의 변형으로 상속 대신 콜백 메서드를 사용하는 패턴이다.

다음은 간단한 예시이다.
```kotlin

// 콜백 인터페이스
interface Callback {
    fun doSomething()
}

// 템플릿 클래스
class Template {
    fun execute(callback: Callback) {
        println("시작")
        callback.doSomething()
        println("종료")
    }
}

// 사용 예
fun main() {
    val template = Template()
    
    // 익명 객체를 사용한 콜백
    template.execute(object : Callback {
        override fun doSomething() {
            println("콜백 실행")
        }
    })
    
    // 람다를 사용한 더 간결한 방식
    template.execute {
        println("람다로 콜백 실행")
    }
}

```

템플릿 콜백 패턴은 템플릿 메서드 패턴과 달리 상속을 사용하지 않기 때문에 상속의 단점을 피할 수 있다.







