---
layout: post
title: Spring Container과 Bean
date: 2024-05-16 08:00 +0900
img_path: /assets/images/
category: spring
tags: [ spring ]
---

# Spring Container란
스프링은 IoC/DI의 개념의 기능을 제공을 위해 컨테이너를 제공한다.

스프링의 컨테이너는 미리 오브젝트를 생성해서 가지고 있다가 필요한 곳에 적절히 주입해준다.

핵심은 컨테이너가 주도적으로 생성, 관계 설정, 제거 등 오브젝트에 대해 제어권을 코드 대신에 가지고 있는 것이다. 이는 IoC와 같기 때문에 IoC Container라고도 부르기도 한다.

스프링 컨테이너가 관리하는 오브젝트를 Bean이라고 부른다. Container를 더 자세히 알아보기 전에 Bean에 대해 알아보자.

# Spring Bean 
Spring Bean은 Spring Container에 의해 실제 관리 되는 객체를 의미한다. Bean은 애플리케이션 전반적으로 공유되며 컨테이너에 의해 의존성 주입되어 진다. 

### Bean의 생명주기
Bean은 생성부터 소멸까지의 아래의 생명주기를 가진다.

1. 인스턴스화
2. 의존성 주입
3. 초기화
4. 사용
5. 소멸

이 때 초기화와 소멸 단계는 Bean 객체에 @PostContruct, @PreDestory 애노테이션을 메서드에 명시하면 직접 제어할 수 있다.

```java
@Component
public class MyBean {
    @PostConstruct
    public void init() {
        // 초기화
    }

    @PreDestroy
    public void destroy() {
        // 소멸
    }
}
```

### Bean의 스코프

스프링 Bean 스코프는 말 그대로 Bean이 효력을 가지는 범위를 의미한다.

1. Singleton : 기본 스코프로, 컨테이너 내에서 하나의 인스턴스만 생성된다.
Service나 DB Pool처럼 공유되는 상태를 가지지 않으며 하나의 인스턴스만 생성되어야 하는 경우에 사용되며 기본값이다.

2. Prototype : 요청할 때마다 새로운 인스턴스를 생성한다.
어떤 복잡한 연산이나 트랜잭션처럼 별도의 상태를 가져야하는 경우 사용된다.

이 스코프는 생성, 주입, 초기화까지만 처리 하고 소멸은 컨테이너가 관리하지 않는다. (PostDestroy 메서드 호출 X)

```kotlin
@Component
@Scope("prototype")
class MyBean {
    // ...
}
```

3. Request : HTTP 요청마다 빈이 생성된다. 
HTTP 요청마다 독립적인 상태를 유지할 때 사용된다.

```kotlin
@Component
@Scope("request")
class MyBean {
    // ...
}
```

## Bean 등록 방법
- @Configuration, @Bean
@Bean은 메서드 레벨의 애노테이션으로 해당 메서드가 반환하는 객체를 Bean으로 등록한다.

이 후 해당 클래스에 @Configuration 애노테이션을 붙여 스프링 컨테이너에게 해당 클래스가 Bean을 등록하는 설정 클래스임을 알려준다.

```kotlin
@Configuration
class AppConfig {
    @Bean
    fun myBean(): MyBean { // Bean 이름은 메서드 이름으로 등록된다.
        return MyBean() 
    }
}
```

- Bean의 이름을 지정하고 싶다면 `@Bean(name = "myBean")`으로 지정할 수 있다.
- Bean의 스코프를 지정하고 싶다면 `@Scope("prototype")`으로 지정할 수 있다.
- 빈으로 설정할 객체가 또 다른 빈을 참조하고 싶다면 메서드의 파라미터로 다른 빈을 주입받을 수 있다.

```kotlin
@Configuration
class AppConfig {
    @Bean
    fun myBean(anotherBean: AnotherBean): MyBean {
        return MyBean(anotherBean)
    }
  
    @Bean
    fun anotherBean(): AnotherBean {
      return AnotherBean()
    }
}
```


- @Component, @ComponentScan
이 애너테이션은 해당 클래스를 Bean으로 인식하게 해준다. 

```kotlin
@Component
class MyBean {
    // ...
}
``` 

웹 서비스에서 많이 사용하는 @Service, @Repository, @Controller, @RestController, @Configuration 애노테이션도 이미 @Component를 상속받기 때문에 자동으로 Bean으로 등록된다.

![]({{site.url}}/assets/images/spring-container-1.png)


@ComponentScan은 @Component 애노테이션이 붙은 클래스를 찾아 Bean으로 등록한다.

Spring boot에서는 @SpringBootApplication 애노테이션에 이미 @ComponentScan이 포함되어 있기 때문에 별도로 설정할 필요가 없다.

![]({{site.url}}/assets/images/spring-container-2.png)

그럼 이 두 가지 기술은 각각 어떨때 사용하는게 좋을까?

- @Component (자동)
  많이 사용되고 정형화된 패턴에 대해 가독성을 높여주고 반복적인 작업을 편하게 도와주기 때문에 일반적으로 많이 사용된다.

- @Bean (수동)
이 방식은 보통 비지니스 로직를 도와주는 역할인 기술적인 문제나 aop 같은 곳에서 주로 사용된다.
앱에서 공통적으로 사용되는 만큼 설정 정보를 수등 등록해서 명확히 보여줄수 있는 장점이 있다. 




이제 다시 Spring Container에 대해 알아보자.

# Container를 구성하는 인터페이스 : BeanFactory, ApplicationContext

### BeanFactory
스프링 컨테이너의 최상위 인터페이스로, 스프링 빈을 생성, 관리, 주입하는 핵심 역할을 담당한다.

### 의존성 주입 방식
BeanFactory는 생성자 주입, setter 주입, 필드 주입 방식을 지원하는데 프레임워크는 이 중 생성자 주입을 적극 권장한다. 

생성자 주입은 생성자를 통해 의존성을 주입하는 방식으로 생성자의 특성에 따라 장점을 아래와 같은 장점이 있다.

  - 불변
    의존관계는 한번 맺어진 이후 변경 될 일은 거의 없다. 때문에 객체가 생성될 때 딱 한번만 호출 되는 생성자와 잘 어울린다. 이 후에 의존 관계를 변경할 수 있는 방법 자체가 없기 때문이다.
  - 안정성
    생성자를 이용한다면 객체 생성 시점에 반드시 필요한 정보를 모두 받을 수 있다. 이는 객체의 상태를 항상 유효한 상태라고 볼 수 있기 때문에 안전한다.
  - POJO
  순수 자바 코드이기 때문에 스프링이나 DI 프레임워크에 종속되지 않고 유닛 테스트가 가능하다.

```kotlin
@Component
class MyBean @Autowired constructor(anotherBean: AnotherBean) {
  private val anotherBean: AnotherBean = anotherBean

}
```

이 때 만약 생성자가 하나만 존재한다면 @Autowired 애노테이션을 생략해도 된다!

```kotlin
@Component
class MyBean(anotherBean: AnotherBean) {
  private val anotherBean: AnotherBean = anotherBean

}
```

### ApplicationContext
사실 BeanFactory는 스프링 컨테이너의 최상위 인터페이스이지만 실제로 사용되는 컨테이너는 ApplicationContext이다. 

ApplicationContext는 BeanFactory의 모든 기능을 포함하며, 더 많은 기능을 제공한다. 

- 국제화 기능
- 환경변수: 로컬,운영 등의 환경변수를 구분해서 처리 가능
- 애플리케이션 이벤트 : 이벤트를 발행하고 구독하는 모델을 지원
- 편리한 리소스 조회 : 파일,클래스 패스 등 리소스를 조회 가능 


![]({{site.url}}/assets/images/spring-container-3.png)





