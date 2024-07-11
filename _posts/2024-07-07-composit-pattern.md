---
layout: post
title: Composite Pattern
date: 2024-07-07 08:00 +0900
img_path: /assets/images/
category: design-pattern
tags: [ design-pattern, 구조 관련 디자인 패턴 ]
---

## 컴포짓(Composite) 패턴

컴포짓 패턴은 객체들을 트리 구조로 구성하여 부분-전체 계층을 표현하는 패턴이다. 즉, 그룹 전체와 개별 객체를 동일하게 처리 할 수 있는 패턴이다.

클라이언트 입장에서 사용하는 객체가 트리의 마지막 오브젝트인지 전체인지 구분하지 않고 사용할 수 있도록 한다. 

## 컴포짓 패턴 구조

![]({{site.url}}/assets/images/composit.png)

1. Component : 전체와 부분을 나타내는 인터페이스를 정의한다. Leaf와 Composite 클래스는 Component 인터페이스를 구현한다.
2. Leaf : 부분 객체에 해당하는 클래스로, 자식을 가질 수 없다.
3. Composite : 전체 객체에 해당하는 클래스로, 자식을 가질 수 있다.
4. Client : Component 인터페이스를 사용하는 클라이언트

Leaf와 Composite 클래스는 Component 인터페이스를 구현하기 때문에 클라이언트는 Leaf와 Composite를 구분하지 않고 즉, 전체와 부분을 동일하게 취급할 수 있다.

## 패턴 적용 X 
아이템과 그 아이템을 담는 가방이 있다고 하자. 아이템은 부분 객체이고, 가방은 전체 객체라고 할 수 있다. 

이 때 클라이언트 쪽에서 아이템 가격을 print 해보는 코드를 패턴 적용 없이 작성해보자.
 

```kotlin
class Item(val name:String, val price:Int)

class Bag{
  val items = mutableListOf<Item>()
  
  fun addItem(item: Item){
    items.add(item)
  }
}


fun main(){
  val book = Item("책", 10000)
  val earphone = Item("이어폰", 5000000)
  
  val bag = Bag()
  bag.addItem(book)
  bag.addItem(earphone)

  fun printPrice(item:Item){
    println(item.price)
  }

  fun printPrice(bag: Bag){
    println(bag.items.sumOf { it.price })
  }
  
  printPrice(book)
  printPrice(bag)
  
}

```

함수 `printPrice`를 보게 되면 클라이언트는 구체적으로 `Item`과 `Bag`를 구분해 사용하고 있다. 

클라이언트 측에서 어떤 객체를 상세히 알아야하고 그것을 구분해 사용하는 것은 유연성이 떨어진다.

## 패턴 적용 O 

컴포짓 패턴을 적용하여 클라이언트 측에서 전체와 부분을 구분하지 않고 사용할 수 있도록 해보자.

먼저 `Component` 인터페이스를 정의한다. 클라이언트는 `Component` 인터페이스를 사용하여 전체와 부분을 구분하지 않고 사용할 수 있다.

```kotlin

interface Component{
  fun printPrice()
}

```

이 후 Item과 Bad 클래스를 `Component` 인터페이스를 구현하도록 변경한다.

```kotlin
class Item(val name:String, val price:Int): Component{
  override fun printPrice() {
    println(price)
  }
}

class Bag: Component{
  val items = mutableListOf<Component>()
  
  fun addItem(item: Component){
    items.add(item)
  }
  
  override fun printPrice() {
    println(items.sumOf { it.printPrice() })
  }
}

```

이제 클라이언트는 `Component` 인터페이스를 사용하여 전체와 부분을 구분하지 않고 사용할 수 있다.

```kotlin

fun main() {

  fun printPrice(comp: Component) {
    println(comp.printPrice())
  }

  val book = Item("책", 10000)
  printPrice(book)
  val earphone = Item("이어폰", 5000000)

  val bag = Bag()
  bag.addItem(book)
  bag.addItem(earphone)
  printPrice(bag)
}

```


## 장단점

### 장점

- 전체와 부분을 동일하게 취급할 수 있다.
- 복잡한 트리 구조를 간단하게 표현할 수 있다.
- 클라이언트 코드를 변경하지 않고도 새로운 객체를 추가할 수 있다.


### 단점 

- 공통된 인터페이스를 반드시 사용하기 때문에 지나치게 일반화하거나 억지로 일반화 하게 될 수 있다.


## 실제 사례

### Swing의 Frame과 Component 클래스

Frame은 Component 클래스를 여러개 가질 수 있다. JTextFiled와 JButton 모두 Component 클래스를 상속받아 사용한다.

이 때 여러 Component를 담고 있는 frame에 `setVisible` 함수를 호출하면 frame에 속한 모든 Component들의 `setVisible` 함수를 호출 할 수 있다. 



## Reference

- 코딩으로 학습하는 GoF의 디자인 패턴 - 백기선




