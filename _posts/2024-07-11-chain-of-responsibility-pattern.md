---
layout: post
title: Chain Of Responsibilities Pattern
date: 2024-07-11 08:00 +0900
img_path: /assets/images/
category: design-pattern
tags: [ design-pattern, 구조 관련 디자인 패턴 ]
---

## 책임 연쇄 패턴 (Chain Of Responsibilities Pattern)

좋은 객체는 한 가지 책임만을 가지고 있다. 이 패턴은 객체의 책임 연결되어 구현한다. 

요청을 보내는 쪽과 처리하는 쪽을 분리하는 패턴이며 요청을 처리하는 핸들러가 어떤 구체적인 타입인지 알 필요 없이 즉, 디커플링 된 구조를 가지게 하는 패턴이다.

## 패턴 구조

![]({{site.url}}/assets/images/chain.png)

1. Handler: 요청을 처리하는 인터페이스를 정의한다.
2. ConcreteHandler: 실제 요청을 처리하는 클래스들이다. 이 handler들은 서로 연결(체인)되어 있기 때문에 어떤 요구사항에 따라 요청을 처리하거나 다음 핸들러에게 넘길 수 있다.

## 패턴 적용 전

예를 들어 단순히 요청 본문을 출력하는 기능이 있다. 

```kotlin

class Request(val content: String)

class RequestHandler {
  fun handleRequest(request: Request) {
    println("Request: ${request.content}")
  }
}

val request = Request("Hello")
val handler = RequestHandler() 
handler.handleRequest(request)  

```

여기에 요청이 인증된 사용자에게만 허용되도록 하는 기능을 추가하고 싶다고 가정해보자. 만약 기능 추가를 위해 RequestHandler 클래스를 수정한다면 이는 SRP(Single Responsibility Principle)을 위반하는 것이다. 

가장 쉬운 방법은 새로운 클래스를 만들고 기존 클래스를 상속 받는 것이다. 

```kotlin

class AuthRequestHandler: RequestHandler() {
  fun handleRequest(request: Request) {
    // 인증 로직 ~~  
    super.handleRequest(request)
  }
  
}
```

이제 사용은 다음과 같이 한다.

```kotlin

val request = Request("Hello")
val handler = AuthRequestHandler() // 기존 RequestHandler 대신 AuthRequestHandler 사용
handler.handleRequest(request)  

```

하지만 이 방법은 새로운 기능을 추가할 때마다 새로운 클래스를 만들어야 하기 때문에 좋지 못한다.

예를들어 로깅 기능이 추가되어야한다면 새로운 클래스를 만들어야 한다. 만약 로깅과 인증을 동시에 사용해야하는 경우도 또 다른 클래스를 만들어야 한다.

또한 클라이언트 측이 어떤 핸들러를 사용해야 하는지 구체적으로 알기 때문에 결합도가 높아진다.



### 패턴 적용 후

책임 연쇄 패턴을 적용하면 이러한 문제를 해결할 수 있다. 클라이언트는 단지 요청을 보내기만 하면 되며 어떤 핸들러들이 요청에 대해 기능을 수행할지에 대해 알지 못한다.

먼저 Handler 추상 클래스로 다음 핸들러를 연결하기 위한 next 변수와 다음 핸들러를 설정하는 메서드를 정의한다. 

`setNext`는 다음 핸들러를 설정하고 다음 핸들러를 반환한다. 이를 통해 체인 형태로 핸들러를 연결할 수 있다.

```kotlin

abstract class Handler {
  var next: Handler? = null
  
  fun setNext(handler: Handler): Handler {
    this.next = handler
    return handler
  }
  
  abstract fun handleRequest(request: Request)
}

```

그리고 이 Handler를 상속받아 구체적인 핸들러를 만든다. 

```kotlin

class RequestHandler: Handler() {
  override fun handleRequest(request: Request) {
    next?.handleRequest(request)
  }
}

class PrintRequestHandler: Handler() {
  override fun handleRequest(request: Request) {
    println("Print: ${request.content}")
    next?.handleRequest(request)
  }
}


class AuthRequestHandler: Handler() {
  override fun handleRequest(request: Request) {
    // 인증 로직 ~~  
    next?.handleRequest(request)
  }
}

class LoggingRequestHandler: Handler() {
  override fun handleRequest(request: Request) {
    println("Logging: ${request.content}")
    next?.handleRequest(request)
  }
}

```

실제 사용은 다음과 같다.

```kotlin

val request = Request("Hello")
val handler = RequestHandler()

handler.setNext(PrintRequestHandler())
  .setNext(AuthRequestHandler())
  .setNext(LoggingRequestHandler()) // 체인 형태로 핸들러 연결 

handler.handleRequest(request)


```
지금 예제는 모든 핸들러가 동작하는 것이지만 각 핸들러에서 요청을 처리하거나 처리하지 않고 다음 핸들러에게 넘길지 결정할 수 있다. 

그리고 중요한 점은 클라이언트는 어떤 특정 핸들러가 요청을 처리하는지 알 필요가 없다.


## 장단점

### 장점

- 기존 클리언트 코드를 변경하지 않고 요청에 대한 처리를 추가하거나 변경할 수 있다.
- 기존 코드의 역할을 그대로 유지할 수 있다. 


### 단점 

- 요청을 처리하는 핸들러가 많아질수록 디버깅이 어려워질 수 있다.

## 실제 사례

- 자바 서블릿 필터 

필터 여러개를 거치면서 요청을 처리하는 방식이 책임 연쇄 패턴과 유사하다.

```kotlin

val filter = Filter { request, response, chain ->
  println("before")
  chain.doFilter(request, response)
  println("after")
}
```


## Reference

- 코딩으로 학습하는 GoF의 디자인 패턴 - 백기선




