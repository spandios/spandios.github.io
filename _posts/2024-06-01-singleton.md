---
layout: post
title: Singleton Pattern
date: 2024-05-16 08:00 +0900
img_path: /assets/images/
category: design-pattern
tags: [ design-pattern, 객체 생성 ]
---


# 싱글턴 패턴

싱글턴 패턴은 객체 생성 패턴 중의 하나로, 클래스가 단 하나의 인스턴스만을 갖도록 보장하는 패턴이다.

가장 대표적인 예로는 Spring Framework의 Bean, DB Pool, Logger 등이 있다. 모두 한번 인스턴스를 생성하고 재사용하는 것이 효율적인 사례인 것을 알 수 있다.  


## 구현 방법들

### Eager Initialization
클래스 로딩 시점에 바로 인스턴스를 생성하는 방법으로 가장 간단한 방법이지만, 당장 사용하지 않더라도 인스턴스를 생성하기 때문에 만약 리소스가 큰 인스턴스라면 메모리 낭비가 발생할 수 있다.

이 때 instance는 static final로 선언하도록 한다. 

`static`은 클래스가 로딩시점에 초기화 되는데 java의 클래스 로딩은 스레드에 안전하게 설계 되어 있기 때문에 멀티 쓰레드 환경에서도 안전하다.

`final`키워드는 한번만 초기화되고 변경되지 않기 때문에 멀티스레드 환경에서 안정성을 높이는데 도움이 된다.


```java

public class Singleton {
    private static final Singleton instance = new Singleton(); // Eager Initialization

    private Singleton() {}

    public static Singleton getInstance() {
        return instance;
    }
}

```

### Lazy Initialization

인스턴스가 필요할 때 생성하는 방법으로 앞서 살펴본 Eager와 다르게 실제 사용 시점에 인스턴스를 생성하기 때문에 메모리 낭비를 줄일 수 있다.

하지만 멀티 쓰레드 환경에서는 동기화 문제가 발생해 디자인 패턴 목적과 다르게 여러 인스턴스가 생성될 수 있다.

만약 `Singleton.getInstance()`를 스레드 A와 B가 동시에 호출한다면, A와 B모두 인스턴스가 아직 생성되기 전이라서 두 스레드 모두 인스턴스를 생성하게 되는 것이다.

```java

public class Singleton {
    private static Singleton instance;

    private Singleton() {}

    public static Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();
        }
        return instance;
    }
}

```


## Lazy Holder

이 방식은 Lazy Loading도 가능하며 멀티 쓰레드 환경에서도 안전하다. 

클래스 안에 내부 클래스(Holder)를 두어 클래스 로딩 시점에는 미리 인스턴스를 생성하지 않기 때문에 Lazy Loading이 가능하다.

또한 getInstance() 메서드가 호출 될 때 내부 클래스가 로딩되어 인스턴스를 생성하는데 이 때 JVM의 클래스 로딩은 스레드에 안전하게 설계되어 있기 때문에 멀티 쓰레드 환경에서도 안전하다.

다만 Reflection API를 통해 private 생성자를 호출하거나 직렬화, 역직렬화를 통해 객체를 생성하는 경우에는 대응할 수 없다.


```java
class Singleton {

    private Singleton() {}

    // Lazy Holder, 내부 클래스라서 클래스 로딩 시점에는 인스턴스 생성이 되지 않는다.
    private static class SingleInstanceHolder {
        private static final Singleton INSTANCE = new Singleton();
    }
    
    // 이 메서드가 호출되는 시점에 내부 클래스가 로딩이 되어 Singleton 인스턴스가 생성된다. JVM 클래스 로딩은 스레드에 안전하게 설계되어 있기 때문에 멀티 쓰레드 환경에서도 안전하다.
    public static Singleton getInstance() {
        return SingleInstanceHolder.INSTANCE;
    }
}
```


## Enum
Enum은 JVM에 의해 싱글톤이 보장되는 방법으로 언어단에서 기본적으로 한번만 초기화하기 때문에 thread safe하다. 

위에 LazyHolder와 달리 Reflection, 직렬화, 역직렬화에도 안전하다. 

다만 Eager Initialization로 아직 사용하지 않아도 메모리를 차지하게 된다.

  ```java
public enum SingletonEnum {
    INSTANCE;
    
    private final DBClient dbClient;
    
    private SingletonEnum() {
        dbClient = new DBClient();
    }
    
    public DBClient getDBClient() {
        return dbClient;
    }
    
}


public static void main(String[] args) {
  DBClient dbClient = SingletonEnum.INSTANCE.getDBClient();
}

```
 


## 싱글턴 패턴의 장단점
### 장점
- 하나의 인스턴스만을 사용하기 때문에, 불필요한 인스턴스 생성 비용을 아낄 수 있다.

### 단점
- 테스트가 어렵다.
  - 전역 상태 : 테스트 케이스간에 전역 상태를 공유하게 되어 테스트 케이스간의 의존성이 생길 수 있다.
  - Mocking : 테스트 프레임워크가 Mocking을 위해 상속이나 생성자 방법을 사용한다면 싱글턴 객체는 상속이나 생성자를 사용할 수 없기 때문에 Mocking이 어렵다.
- 사실 전역변수와 같기 때문에 많은 곳에서 사용되면 결합도가 높아질 수 있어 주의해야 한다.


