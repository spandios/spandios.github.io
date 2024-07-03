---
layout: post
title: Adapter Pattern
date: 2024-07-03 08:00 +0900
img_path: /assets/images/
category: design-pattern
tags: [ design-pattern, 구조 관련 디자인 패턴 ]
---

## 어댑터(Adapter) 패턴

어댑터 패턴은 쉽게 말해 서로 다른 인터페이스를 가진 두 클래스를 연결해주는 패턴이다.

만약 220v용 전자기기를 가지고 110v를 사용하는 나라에 여행을 가면 전압 어댑터를 사용해야한다. 

이 때 전압 어댑터는 220v를 110v로 변환해주는 역할을 한다. 즉, 서로 다른 인터페이스이지만 어댑터가 중간에서 같은 인터페이스로 변환해주는 역할을 한다.

target이나 adaptee 쪽을 변경할 수 없는 경우에 주로 사용한다.

## 어댑터 패턴 구조

![]({{site.url}}/assets/images/adapter-pattern.png)

1. Client : 어댑터 패턴을 사용하는 클라이언트
2. Target: 기존 클라이언트가 사용하고 있는 인터페이스
3. Adaptee: Target과 다른 인터페이스를 가진 클래스
4. Adapter: Target 인터페이스를 구현. 실제 Adaptee의 인터페이스를 Target 인터페이스에 맞게 변환하는 역할을 하는 클래스. (보통 Adapter 클래스는 Adaptee 클래스를 필드로 가지고 있다.)


## 어댑터 패턴 코드 예시

```kotlin
// Target interface
interface Voltage110V {
  fun provide110V()
}

// Adaptee class
class Voltage220V {
  fun provide220V() {
    println("Providing 220V")
  }
}

// Adapter class
class VoltageAdapter(private val voltage220V: Voltage220V) : Voltage110V {
  override fun provide110V() {
    println("Adapting 220V to 110V")
    // 추가적인 변압 로직이 여기에 포함될 수 있음
    voltage220V.provide220V()
  }
}

// Client code
fun main() {
  val voltage220V = Voltage220V()
  val adapter: Voltage110V = VoltageAdapter(voltage220V)

  // 이제 220V 전자기기를 110V 소켓에서 사용할 수 있음
  adapter.provide110V()
}

```

## 실제 사례 

- `java.util.Arrays.asList()` : Arrays 클래스의 asList() 메서드는 배열을 List로 변환해주는 역할을 한다. 

```java
String[] array = new String[] {"a", "b", "c"};
List<String> list = Arrays.asList(array);
```

어댑티: array, 타겟: List, 어댑터: Arrays.asList

- Spring DispatcherServlet의 HandlerAdapter

여기서 핸들러란 클라이언트의 요청을 처리하는 컨트롤러를 의미한다.
  
```java
  public interface HandlerAdapter {
    boolean supports(Object handler);
    ModelAndView handle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception;
    long getLastModified(HttpServletRequest request, Object handler);
  }
  
```

```java
HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());
ModelAndView mv = ha.handle(request, response, mappedHandler.getHandler());
```


먼저 getHandlerAdapter() 메서드는 handler에 맞는 HandlerAdapter를 반환한다.

왜 HandlerAdapter가 필요한가? 바로 다양한 핸들러를 지원하기 위해서이다.  

실제로 스프링에서 핸들러 중에는 @Controller를 사용하는 핸들러도 있고 HttpRequestHandler 인터페이스를 구현한 핸들러도 있다.

다양한 핸들러는 어댑티이고 프레임워크는 클라이언트이다. 프레임워크가 기대하는 Target은 ModelAndView이고 다양한 핸들러로 부터 ModelAndView를 얻기 위해 어댑터인 HandlerAdapter가 필요한것이다.


## 장단점

- 장점 
  - 기존 코드를 수정하지 않고 원하는 인터페이스를 사용할 수 있다.
- 단점 
  - 객새 클래스가 생겨 복잡도가 증가할 수 있다. 만약 어댑티에 해당하는 클래스를 수정 가능하다면 어댑터 패턴을 사용하지 않는 방법도 고려해볼 수 있다. 




## Reference

- 코딩으로 학습하는 GoF의 디자인 패턴 - 백기선




