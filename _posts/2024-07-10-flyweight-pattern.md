---
layout: post
title: Flyweight Pattern
date: 2024-07-05 08:00 +0900
img_path: /assets/images/
category: design-pattern
tags: [ design-pattern, 구조 관련 디자인 패턴 ]
---

## 플라이웨이트(Flyweight) 패턴

객체를 가볍게 만들어 메모리 사용을 줄이는 패턴으로, 주로 굉장히 많은 인스턴스를 생성해야할 때 사용한다.

자주 변하는 속성과 변하지 않는 속성을 분리하고 재사용해 메모리 사용을 줄인다. 

## 플라이웨이트 패턴 구조

![]({{site.url}}/assets/images/flyweight.png)

1. FlyweightFactory : 플라이웨이트 객체를 생성하고 관리하는 클래스
2. Flyweight : 플라이웨이트 객체의 인터페이스

## 패턴 적용 전 

Character 객체를 생성하는 예제를 살펴보자. 

지금은 5개지만 만약 Character가 더 필요해서 많은 객체 생성을 하면 메모리 사용량이 많아질 것이다. 

```kotlin

fun main(){
 val c1 =  Character(value = 'A', size = 12, color = "red", font = "Arial")
  val c2 = Character(value = 'B', size = 12, color = "red", font = "Arial")
  val c3 = Character(value = 'A', size = 12, color = "red", font = "Arial")
  val c4 = Character(value = 'A', size = 12, color = "red", font = "Arial")
  val c5 = Character(value = 'B', size = 12, color = "red", font = "Arial")
}


```

메모리 사용량을 줄이기 위해 플라이웨이트 패턴을 적용해보자. 

### 패턴 적용 후

먼저 크게 변경되지 않는 Font와 관련된 것을 따로 나눈다. 또한 이 flyweight에 해당하는 부분은 immutable해야한다. 다른 곳에서 공통적으로 사용하기 때문에 혹시 변경되면 다른 곳에 영향을 줄 수 있기 때문이다.  

```kotlin

data class Font(val name: String, val size: Int)

```

이제 변경이 자주 되는 부분인 Character Flywieght을 만들어보자. Character 클래스는 value, color 그리고 덜 변경되는 font객체를 가지고 있다. 

```kotlin

data class Character(val value: Char, val color:String, val font: Font)


```

이제 `FlyweightFactory`를 만들어 Flyweight 객체를 관리하도록 한다. FlywieghtFactory는 Font 객체를 관리하고 캐싱하는 역할을 한다.  

```kotlin

class FontFactory{
  private val fonts = mutableMapOf<String, Font>()
  
  fun getFont(name: String, size: Int): Font{
    val key = "$name-$size"
    return fonts.getOrPut(key) { Font(name, size) }
  }
}



```

이제 사용은 다음과 같이 한다.

```kotlin

fun main() {
  val fontFactory = FontFactory()
  val fontXS = fontFactory.getFont("Arial", 12)
  val fontSM = fontFactory.getFont("Arial", 14)
  val fontMD = fontFactory.getFont("Arial", 16)
  val fontLG = fontFactory.getFont("Arial", 18)
  val c1 = Character(value = 'A', color = "red", font = fontXS)
  val c2 = Character(value = 'B', color = "red", font = fontXS)
  val c3 = Character(value = 'A', color = "red", font = fontSM)
  val c4 = Character(value = 'A', color = "red", font = fontMD)
  val c5 = Character(value = 'B', color = "red", font = fontLG)
}

```




## 장단점

### 장점

- 메모리를 절약할 수 있다.


### 단점 

- 코드가 복잡해질수 있다.



## Reference

- 코딩으로 학습하는 GoF의 디자인 패턴 - 백기선




