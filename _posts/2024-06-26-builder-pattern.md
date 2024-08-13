---
layout: post
title: Builder Pattern
date: 2024-06-26 08:00 +0900
img_path: /assets/images/
category: design-pattern
tags: [ design-pattern, 객체 생성 ]
---

## 빌더 패턴란

빌더 패턴은 복잡한 객체를 단계별로 생성할 수 있도록 돕는 디자인 패턴이다.

생성자로 받을 매개변수를 함수로 차곡 차곡 쌓아놓고 마지막에 통합해서 객체를 생성하는 방식으로, 특히 생성자가 복잡하거나 객체를 설정할 때 많은 매개변수가 필요한 경우 유용하다.


## 빌더 패턴 구조

![]({{site.url}}/assets/images/builder-pattern.png)

1. Builder: 객체 생성에 필요한 메서드들을 선언한다.
2. ConcreteBuilder: Builder 인터페이스를 구현하며, 제품의 각 부분을 생성하는 데 필요한 메서드를 구현한다.
3. Product: 빌더에 의해 생성되는 복잡한 객체이다.
4. Director: Builder 인터페이스를 사용하여 객체를 생성하는 과정을 관리한다. 필수는 아님.

## 빌더 패턴이 필요한 이유

빌더 패턴전에 객체를 생성할 때 생성자로 매개변수를 받아 객체를 생성하는 방식을 사용했다.

아래 예제는 컴퓨터 객체 생성자를 오버로딩해 필요한 모든 경우의 수를 다루는 예제이다. 이러한 패턴을 점층적 생성자 패턴이라 한다.

하지만 많은 생성자 수 때문에 사용하는 측에서 파악이 어렵고, 매개변수가 길어지면 길어질수록 순서를 헷갈리기 쉽다.

```kotlin


class Computer(
  val cpu: String,
  val memory: String,
  val storage: String,
  val power: String,
  var gpu: String? = null, // 선택적으로 사용할 수 있는 매개변수
  var cooler: String? = null // 선택적으로 사용할 수 있는 매개변수
) {

  // 보조 생성자: gpu만 초기화할 때 사용
  constructor(cpu: String, memory: String, storage: String, power: String, gpu: String)
    : this(cpu, memory, storage, power, gpu, null)

  // 보조 생성자: cooler만 초기화할 때 사용
  constructor(cpu: String, memory: String, storage: String, power: String, cooler: String, dummy: Unit = Unit)
    : this(cpu, memory, storage, power, null, cooler)
}
  
  val computer = Computer("i7", "DDR4", "SSD", "500W", "RTX 3080", "Cooler")
  
  val computer = Computer("i7", "DDR4", "SSD", "500W", "RTX 3080",)
  
  val computer = Computer("i7", "DDR4", "SSD", "500W", "Cooler", Unit)




```

이러한 문제를 해결하기 위해 자바 빈 패턴도 사용할 수 있다. 자바 빈 패턴은 객체를 일단 생성한 후에 setter 메서드를 사용해 객체의 필드를 설정하는 방식이다.

```kotlin

class Computer {
  lateinit var cpu: String // 필수
  lateinit var memory: String // 필수
  lateinit var storage: String // 필수
  lateinit var power: String // 필수

  lateinit var gpu: String // 선택
  lateinit var cooler: String // 선택


  fun setCpu(cpu: String) {
    this.cpu = cpu
  }

  fun setMemory(memory: String) {
    this.memory = memory
  }

  fun setStorage(storage: String) {
    this.storage = storage
  }

  fun setPower(power: String) {
    this.power = power
  }

  fun setGpu(gpu: String) {
    this.gpu = gpu
  }

  fun setCooler(cooler: String) {
    this.cooler = cooler
  }

}

val computer = Computer()
computer.setCpu("i7")
computer.setMemory("DDR4")
computer.setStorage("SSD")
computer.setPower("500W")
computer.setGpu("RTX 3080")


```

하지만 자바 빈 패턴은 객체의 일관성을 보장하지 못하고, 객체가 완성되기 전에 객체의 일부가 설정되어 있을 수 있다. 가독성을 위해 생성자의 안정성을 포기한 것이다.

이 때 빌더 패턴을 사용하면 생성자의 안정성과 자바 빈 패턴의 가독성을 모두 확보할 수 있다.

## 빌더 패턴 구현
빌더 패턴은 두가지 방식으로 구현할 수 있다. 

하나는 `심플 빌더 패턴`이라 불리며 이펙티브 자바에서 소개되었다.

이 패턴은 빌더 클래스가 제품 클래스의 정적 내부 클래스(Static Inner Class)로 정의된다.

`내부 클래스`로 정의하는 이유는 다음과 같다.

- 빌더 클래스는 단지 목표하는 객체 생성만을 위해 존재하기 때문에, 빌더 클래스를 그 클래스를 물리적으로 가깝게 하는게 파악하기 쉽다.
- 제품 클래스의 생성을 빌더 클래스로만 가능하게 하기 위해서는 `제품 생성자를 외부에 노출시키면` 안된다. 
  - 내부 클래스를 이용하면 제품 생성자를 private으로 선언해 외부에 노출시키지 않으면서도 빌더 클래스에서 제품의 생성자를 호출해 생성할 수 있다.
  - 내부 클래스는 외부 클래스의 private 생성자라도 호출할 수 있기 때문이다.

추가적으로 내부 클래스를 `정적`으로 정의하는 이유는 다음과 같다.

- 내부 클래스는 정적으로 정의해야 메모리 누수에 대한 문제를 쉽게 방지할 수 있다.
  - 내부 클래스는 외부 클래스의 인스턴스를 참조하기 때문에 외부 클래스의 인스턴스가 GC되지 않는 문제가 발생할 수 있다.
  - 정적 내부 클래스는 외부 클래스의 인스턴스를 참조하지 않기 때문에 이러한 문제가 발생하지 않는다.
  - 내부 클래스가 외부 클래스의 멤버를 참조할 필요가 없다면 정적 내부 클래스로 정의하는 것이 좋다.
  - 빌더 클래스는 제품 클래스의 멤버를 참조하지 않고 단지 제품 객체를 생성하기만 하기 때문에 정적 내부 클래스로 정의하는 것이 좋다.

- 빌더 클래스를 제품 클래스를 생성하기 전에 사용할 수 있어야한다. 
  - 만약 빌더 클래스를 그냥 내부 클래스로 정의하면 객체를 만들고 나서야 빌더 클래스를 사용할 수 있다.

```kotlin

class Computer private constructor( // private 생성자로 외부에서 생성을 막는다.
  val cpu: String,
  val memory: String,
  val storage: String,
  val power: String,
  val gpu: String?,
  val cooler: String?
) {
    
  // 정적 내부 클래스로 빌더 클래스를 정의한다.
  // 빌더의 생성자에 필수 매개변수를 받아 객체를 생성한다.
  class Builder(
    val cpu: String,
    val memory: String,
    val storage: String,
    val power: String
  ) {
    private var gpu: String? = null // 선택적 매개변수
    private var cooler: String? = null // 선택적 매개변수

    // apply 함수를 사용해 빌더 객체를 반환한다. 이를 통해 지속적으로 빌더 객체를 사용할 수 있다.
    fun gpu(gpu: String) = apply { this.gpu = gpu } 
    fun cooler(cooler: String) = apply { this.cooler = cooler }
    
    // 빌더 객체를 사용해 최종적으로 Computer 객체를 생성한다.
    fun build() = Computer(cpu, memory, storage, power, gpu, cooler)
  }
}

val computer = Computer.Builder("i7", "DDR4", "SSD", "500W")
  .gpu("RTX 3080")
  .cooler("Cooler")
  .build()


```

다음은 `디렉터 빌더 패턴`으로 불리며 GoF의 디자인 패턴에서 소개된 방식이다. 

위에 소개한 심플 빌더 패턴은 객체 생성하는 것에 목적을 두는 반면, 이 패턴은 객체 생성과 조립 방법을 분리하는 것에 목적을 둔다.

이 때 빌더를 받아 실제 객체를 생성하는 클래스를 디렉터라고 부른다. 클라이언트가 직접 빌더를 사용해 객체를 생성하는 것이 아니라 디렉터를 통해 객체를 생성하는 방식인 것이다.

```kotlin

class Computer(
  val cpu: String,
  val memory: String,
  val storage: String,
  val power: String,
  val gpu: String?,
  val cooler: String?
)

interface ComputerBuilder {
  fun cpu(cpu: String)
  fun memory(memory: String)
  fun storage(storage: String)
  fun power(power: String)
  fun gpu(gpu: String)
  fun cooler(cooler: String)
  fun build(): Computer
}

class DefaultComputerBuilder : ComputerBuilder {
  private var cpu: String = ""
  private var memory: String = ""
  private var storage: String = ""
  private var power: String = ""
  private var gpu: String? = null
  private var cooler: String? = null

  override fun setCpu(cpu: String) {
    apply { this.cpu = cpu }
  }

  override fun setMemory(memory: String) {
    apply { this.memory = memory }
  }

  override fun setStorage(storage: String) {
    apply { this.storage = storage }
  }

  override fun setPower(power: String) {
    apply { this.power = power }
  }

  override fun setGpu(gpu: String) {
    apply { this.gpu = gpu }
  }

  override fun setCooler(cooler: String) {
    apply { this.cooler = cooler }
  }

  override fun build(): Computer {
    return Computer(cpu, memory, storage, power, gpu, cooler)
  }
}

// 디렉터 클래스로 유저가 빌더를 직접 사용하는 것이 아닌 디렉터를 통해 빌더를 사용한다.
class Director(private val builder: ComputerBuilder) {

  fun documentPC() {
    builder.cpu("i3").memory("DDR3").storage("HDD").power("300W")
  }

  fun gamingPC() {
    builder.cpu("i7").memory("DDR4").storage("SSD").power("500W").gpu("RTX 3080").cooler("Cooler")
  }
}

val builder = DefaultComputerBuilder()
val director = Director(builder)

val docuPC = director.documentPC()
val gamingPC = director.gamingPC()

println(docuPC)
println(computer)

```

## Reference

- 코딩으로 학습하는 GoF의 디자인 패턴 - 백기선
- https://refactoring.guru/ko/design-patterns/builder
- https://inpa.tistory.com/entry/GOF-%F0%9F%92%A0-%EB%B9%8C%EB%8D%94Builder-%ED%8C%A8%ED%84%B4-%EB%81%9D%ED%8C%90%EC%99%95-%EC%A0%95%EB%A6%AC




