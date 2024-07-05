---
layout: post
title: Bridge Pattern
date: 2024-07-05 08:00 +0900
img_path: /assets/images/
category: design-pattern
tags: [ design-pattern, 구조 관련 디자인 패턴 ]
---

## 브릿지(Bridge) 패턴

브릿지 패턴은 추상적인 것과 구체적인 것을 분리하여 서로 독립적으로 수정할 수 있도록 하여 서로 영향을 미치지 않도록 하는 것이다.

## 브릿지 패턴 구조

![]({{site.url}}/assets/images/bridge.png)

1. Abstraction : 가장 상위의 추상적인 개념을 포함하며 일반적으로 인터페이스나 추상클래스로 정의한다.
2. RefinedAbstraction : 추상화를 확장한 것으로, 구체적인 동작을 정희한다.
3. Implementor : 추상화된 것과 같이 행동해야하지만 구체적으로 구현되어야 하는 것을 정의한다.
4. ConcreteImplementor : 실제 Implementor의 구현을 담당한다.


## 패턴 적용 X 

LOL 챔피언은 움직이고 스킬을 사용할 수 있다. 그리고 챔피언은 스킨을 입을 수 있다. 

```kotlin
   interface Champion {
  fun move()

  fun skillQ()

  fun skillW()

  fun skillE()

  fun skillR()
}

```

챔피언 중에 스킨을 입은 것을 나타내보자.

```kotlin

class 하이눈루시안 : Champion {
  override fun move() {
    println("하이눈루시안이 이동합니다.")
  }

  override fun skillQ() {
    println("하이눈루시안의 Q 스킬을 사용합니다.")
  }

  override fun skillW() {
    println("하이눈루시안의 W 스킬을 사용합니다.")
  }

  override fun skillE() {
    println("하이눈루시안의 E 스킬을 사용합니다.")
  }

  override fun skillR() {
    println("하이눈루시안의 R 스킬을 사용합니다.")
  }

  override fun skin() {
    println("하이눈루시안의 스킨을 입습니다.")
  }
}

class 공각기동대카이사: Champion {
  override fun move() {
    println("공각기동대카이사가 이동합니다.")
  }

  override fun skillQ() {
    println("공각기동대카이사의 Q 스킬을 사용합니다.")
  }

  override fun skillW() {
    println("공각기동대카이사의 W 스킬을 사용합니다.")
  }

  override fun skillE() {
    println("공각기동대카이사의 E 스킬을 사용합니다.")
  }

  override fun skillR() {
    println("공각기동대카이사의 R 스킬을 사용합니다.")
  }

  override fun skin() {
    println("공각기동대카이사의 스킨을 입습니다.")
  }
}

```
챔피언과 스킨이 강결합 되어 있어 챔피언이 변경되면 스킨도 변경되어야 한다. 또한 새로운 스킨이나 챔피언이 생기면 새로운 클래스를 계속 만들어야 한다.

## 브릿지 패턴 적용하기 
브릿지 패턴은 추상적인 것과 구체적인 것을 분리하는 것이다. 

챔피언은 추상적인 것이고 스킨은 구체적인 것이다. 따라서 챔피언과 스킨을 분리하여 각각의 클래스로 만들어보자.

DefaultChampion 클래스는 Champion 인터페이스를 구현하고 있으며, 스킨을 가지고 있다. 스킨을 이렇게 분리하면 새로운 챔피언이나 스킨이 생겨도 새로운 클래스를 만들 필요가 없다. 

이렇게 어떤 객체를 포함하는 설계 기법을 합성(Composition)이라고 한다.

```kotlin
interface Skin{
  val name: String
}

// 스킨 클래스
class 하이눈: Skin {
  override val name: String
    get() = "하이눈"
}

```

```kotlin
// 기본 챔피언 클래스, 보유한 스킨 객체를 이용해 Champion 인터페이스의 기능을 동작한다.   
class DefaultChampion(skin: Skin, private val name: String) : Champion {
  private val skin: Skin = skin

  fun move() {
    println("${skin.name} ${name}이 이동합니다.")
  }

  fun skillQ() {
    println("${skin.name} ${name} Q")
  }

  fun skillW() {
    println("${skin.name} ${name} W")
  }

  fun skillE() {
    println("${skin.name} ${name} E")
  }

  fun skillR() {
    println("${skin.name} ${name} R")
  }
  
}


```

이제 루시안이라는 챔피언을 만들고 스킨을 입혀보자.

```kotlin

class 루시안(skin: Skin?) : DefaultChampion(skin, "루시안")

fun main(){
  val 하이눈루시안 = 루시안(하이눈())
  하이눈루시안.move()  // 하이눈 루시안이 이동합니다.
  하이눈루시안.skillQ() // 하이눈 루시안 Q 
  하이눈루시안.skillW() // 하이눈 루시안 W
  하이눈루시안.skillE() // 하이눈 루시안 E
  하이눈루시안.skillR() // 하이눈 루시안 R
  
}
```

## 장단점

- 장점
  - 추상적인것과 구체적인것을 분리하여 각각 독립적으로 확장할 수 있다.
  - 구현부를 변경하지 않고 추상화를 변경할 수 있다.
  - 추상화와 구현부를 독립적으로 확장할 수 있다.
- 단점
  - 추상화와 구현부를 분리하면서 클래스의 수가 늘어나기 때문에 복잡해질 수 있다.   

## 실사례 

- JDBC 

추상적인 것은 JDBC API이고 구체적인 것은 JDBC 드라이버이다. 

드라이버가 달라진다고해서 connection, statement, resultset을 사용하는 방법이 달라지지 않는다. 

```kotlin
fun main(){
  Class.forName("com.mysql.jdbc.Driver") // MYSQL 드라이버를 로드한다.
  
  try{
   val conn = DriverManager.getConnection("jdbc:mysql://localhost:3306) // 데이터베이스에 연결한다." +
     "user=root&password=root")
    val stmt = conn.createStatement()
    val rs = stmt.executeQuery("SELECT * FROM user")
  }catch (e: SQLException){
    e.printStackTrace()
  }
}
```

 
- Logging API

추상적인 것은 로깅 API이고 구체적인 것은 로깅 구현체이다. 로깅 구현체는 Log4j, Logback, JUL 등이 있다. 

```kotlin

fun main(){
  val logger = LoggerFactory.getLogger("com.example.Main")
  logger.info("Hello, World!")
}


```




## Reference

- 코딩으로 학습하는 GoF의 디자인 패턴 - 백기선




