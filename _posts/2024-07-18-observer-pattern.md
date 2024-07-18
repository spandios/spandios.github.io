---
layout: post
title: Observer Pattern
date: 2024-07-18 08:00 +0900
img_path: /assets/images/
category: design-pattern
tags: [ design-pattern, 구조 관련 디자인 패턴 ]
---

## 옵저버 패턴 (Observer Pattern)

옵저버 패턴은 특정 객체 상태 변화를 관찰하고 알림 받는 패턴이다. 객체의 상태가 변화하면 이를 관찰하는 객체에게 알려주어 상태 변화에 대한 처리를 할 수 있도록 한다.

## 패턴 구조

![]({{site.url}}/assets/images/observer.png)

1. Subject : 관찰 대상이 되는 객체이다. 상태가 변화하면 옵저버에게 알린다. 여러 옵저버들을 가지고 있을 수 있고, 옵저버들을 추가(구독)하거나 삭제하는 기능을 가진다.
2. Observer : Subject의 상태 변화를 관찰하는 객체이다. Subject의 상태가 변화하면 Observer에게 알린다.

## 패턴 적용 전

간단한 채팅 서버가 있다고 가정해보자. 유저는 채팅 서버에 어떤 주제에 대해 메세지를 보낼수 있고 조회할 수 있다.

그런데 이 채팅서버는 폴링 방식으로 구현되어 있어서 유저가 최신 메시지를 보려면 조회 함수를 계속 호출해야 한다.

먼저 챗서버를 구현해보자. 주제에 관한 메세지를 저장하고 조회하는 기능을 가지고 있다.

```kotlin


class ChatServer {
  private val messages: MutableMap<String, MutableList<String>> = HashMap()


  fun add(subject: String, message: String) {
    messages.putIfAbsent(subject, mutableListOf(message))
  }

  fun getMessage(subject: String): List<String> {
    return messages[subject] ?: emptyList()
  }
}



```

이제 유저를 구현해보자. 유저는 실제로 채팅 서버에 메세지를 보내고 조회하는 기능을 가지고 있다.

```kotlin

class User(val chatServer: ChatServer) {
  fun sendMessage(subject: String, message: String) {
    chatServer.add(subject, message)
  }

  fun getMessage(subject: String): List<String> {
    return chatServer.getMessage(subject)
  }
}


```

이제 클라이언트 코드를 작성해보자. 유저가 메세지를 보내는 것에 바로 반응하지 못하고 직접 조회 메서드를 호출해야지만 볼 수 있다.

```kotlin

fun main() {
  val chatServer = ChatServer()
  val user = User(chatServer)
  val user2 = User(chatServer)

  user.sendMessage("topic", "hello")
  user.sendMessage("topic2", "world")

  println(user2.getMessage("topic")) // 직접 조회 메서드를 호출하기전까지는 상태 변경을 모른다.


}
```

## 패턴 적용 후

옵저버 패턴을 적용하면 메세지가 보내지면 바로 유저가 알 수 있다. 먼저 Subscriber(Observer) 인터페이스를 만들어보자.

Subscriber는 Subject의 상태 변화를 관찰하는 객체이다. 위의 예제에서 Subscriber 즉 Observer는 유저이다. 유저는 채팅 서버의 상태 변화를 관찰하고 있다.

```kotlin
interface Subscriber {
  fun handleMessage(message: String)
}

class User(val userName: String) : Subscriber {

  override fun handleMessage(message: String) {
    println(message)
  }
}
```

이제 ChatServer 클래스를 수정해보자. ChatServer는 Subject이다. 

Subject는 옵저버들을 가지고 있고 만약 상태가 변화하면 이 상태에 관심있는 옵저버들에게 알린다.

```kotlin

class ChatServer {
  private val subscribers: MutableMap<String, MutableList<Subscriber>> = HashMap<String, MutableList<Subscriber>>()

  fun subscribe(subject: String, subscriber: Subscriber) {
    this.subscribers.putIfAbsent(subject, mutableListOf(subscriber))
  }

  fun unsubscribe(subject: String, subscriber: Subscriber) {
    if (this.subscribers.containsKey(subject)) {
      this.subscribers.remove(subject)
    }
  }

  fun sendMessage(user: User, subject: String, message: String) {
    if (subscribers.containsKey(subject)) {
      val userMessage: String = user.name + ": " + message
      subscribers[subject].forEach { s ->
        s.handleMessage(userMessage)
      }
    }
  }
}



```

이제 클라이언트 코드를 작성해보자. 유저가 채팅 서버에 구독하고 메세지를 보내면 바로 메세지를 받을 수 있다.

```kotlin

fun main() {
  val chatServer = ChatServer()
  val user = User("user1")
  val user2 = User("user2")

  chatServer.subscribe("topic", user)
  chatServer.subscribe("topic", user2)

  chatServer.sendMessage(user, "topic", "hello") // user1과 user2 모두 hello를 받는다.
  chatServer.sendMessage(user2, "topic", "world") // user1과 user2 모두 world를 받는다.
  
  chatServer.unsubscribe("topic", user2) // user2 구독 해지
  
  chatServer.sendMessage(user, "topic", "hello") // user1: hello , user2는 구독 해지되어 메세지를 받지 않는다.

}
```

## 장단점

### 장점

- 상태를 변경하는 객체 (Publisher)와 변경을 감지하는 객체(Subscriber)의 관계를 느슨하게 유지할 수 있다.
- Subject의 상태 변경을 주기적으로 조회하지 않고도 자동으로 감지 가능하다.

### 단점

- 복잡도가 증가한다.
- 다수의 옵저버를 등록 후 제대로 해지 하지 않으면 메모리 누수가 발생할 수 있다.

## Reference

- 코딩으로 학습하는 GoF의 디자인 패턴 - 백기선




