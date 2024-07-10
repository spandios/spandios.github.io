---
layout: post
title: Flyweight Pattern
date: 2024-07-05 08:00 +0900
img_path: /assets/images/
category: design-pattern
tags: [ design-pattern, 구조 관련 디자인 패턴 ]
---

## 프록시(Proxy) 패턴

프록시 패턴은 객체의 대리인 역할을 하는 클래스를 제공해 접근을 제어하거나 기능을 추가할 수 있는 패턴이다. (프록시는 사전적 의미로 대리인)

실제 클래스를 바로 사용하는 대신 프록시 클래스를 통해 간접적으로 접근하며 그 프록시 클래스는 실제 클래스와 같은 인터페이스를 제공하되 중간에 필요한 처리를 수행한다. 

초기화 지연, 접근 제어, 로깅, 캐싱 등 다양한 용도로 사용된다.

## 프록시 패턴 구조

![]({{site.url}}/assets/images/proxy.png)

1. Subject: 클라이언트가 호출할 때 사용하는 인터페이스 
2. Proxy: 원래 요청을 처리하는 RealSubject를 참조해 대리자 역할을 하는 클래스. 클라이언트는 Proxy 객체를 사용하게 된다.
3. RealSubject: 원래 기존 요청을 처리하는 클래스

## 패턴 적용 전 

DB에서 어떤 데이터를 조회하는 기능이 있다고하자.

```kotlin


class DataService {
  override fun findById(id: String): Any {
    return repository.findById(id);
}
}

fun main() {
  val dataService = DataService()
  val data = dataService.findById("test")
}

 

```

이 때 DB에 접근하는 것이 비용이 크다고 생각되어 캐싱을 하려고 한다고 가정해보자. 

다른 캐싱 DB 사용은 어려워 메모리에 캐싱하려 한다. 이 때 기존 DataService를 변경하기 어렵다면 프록시 패턴을 사용해 해결할 수 있다.  


### 패턴 적용 후

먼저 기존 DataService에 대한 인터페이스를 만들고 이를 구현하는 DataServiceImpl를 만든다.

```kotlin

interface DataService {
  fun findById(id: String): Any
}

class DataServiceImpl(private val repository : Repository): DataService{
  override fun findById(id: String): Any {
    return repository.findById(id)
  }
}

```

이제 실제 중간에서 캐싱을 하는 Proxy 클래스를 만든다.

```kotlin

class DataServiceCacheProxy(private val dataService: DataService): DataService{
  private val cache = mutableMapOf<String, Any>()

  override fun findById(id: String): Any {
    // 캐시에서 먼저 조회
    return cache.getOrPut(id) {
      // 캐시에 없는 경우, 실제 서비스를 통해 데이터 조회 및 캐시에 추가
      dataService.findById(id)
    }
  }
}

```

이제 사용은 다음과 같이 한다.

클라이언트는 기존 실제 객체인 DataServiceImpl을 그대로 사용하는 것이 아닌 이 객체를 참조하는 `DataServiceCacheProxy`를 대리자로 사용한다. 

DataServiceCacheProxy는 실제 객체와 같은 인터페이스를 가지고 있으며 참조까지 갖고 있기 때문에 대리자 역할을 완벽히 수행한다.

```kotlin

fun main() {
  val dataService = DataServiceImpl(Repository())
  val dataServiceProxy = DataServiceCacheProxy(dataService)
  val data = dataServiceProxy.findById("test") // DB 조회
  val cachedData = dataServiceProxy.findById("test") // 캐시 조회
}

```



## 장단점

### 장점

- 기존 코드를 변경하지 않고 새로운 기능을 추가 가능하다.
- 기존 코드의 역할을 그대로 유지할 수 있다. 


### 단점 

- 코드가 복잡해질수 있다.

## 실제 사례

- 자바의 다이나믹 프록시
런타임 환경에서 동적으로 프록시 객체를 생성하는 방법이 있다.

```kotlin

fun main(){
  val realSubject = RealSubject() 
  val proxy = Proxy.newProxyInstance(
    realSubject.javaClass.classLoader, // 클래스 로더
    realSubject.javaClass.interfaces, // 인터페이스
    InvocationHandler { proxy, method, args ->
      println("before")
      val result = method.invoke(realSubject, args)
      println("after")
      result
    }
  ) as Subject
  proxy.doSomething() // 기존 실제 객체 메서드 호출 앞뒤로 before, after 출력
}

```


- Spring AOP

Spring AOP는 프록시 패턴을 사용해 구현되어 있다. 

`@Before` 어노테이션을 사용해 메서드 호출 전에 로깅을 하는 기능을 추가할 수 있다. 

```kotlin

@Aspect
@Component
class LoggingAspect {
  @Before("execution(* com.example..*.*(..))")
  fun before() {
    println("before")
  }
}

```

스프링 aop는 다이나믹 프록시(만약 인터페이스를 사용하지 않는 빈이라면 CGLIB를 사용)를 이용해 Target되는 빈 객체를 감싸는 프록시 객체를 생성하고 이를 통해 추가적인 기능을 수행한다. 






## Reference

- 코딩으로 학습하는 GoF의 디자인 패턴 - 백기선




