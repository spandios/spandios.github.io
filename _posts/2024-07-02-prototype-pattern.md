---
layout: post
title: Prototype Pattern
date: 2024-07-02 08:00 +0900
img_path: /assets/images/
category: design-pattern
tags: [ design-pattern, 객체 생성 ]
---

## 프로토타입 패턴

프로토타입은 기존 인스턴스를 복제해 새로운 인스턴스는 방법이다. 

만약 기존의 인스턴스를 만들 때 리소스가 많다고 가정해보자. 이 때 매번 새로운 인스턴스를 생성하는 것은 비효율적이다.

이 때 프로토타입 패턴을 사용하면 기존 인스턴스를 복제해 새로운 인스턴스를 생성할 수 있다.

## 프로토타입 패턴 구조

![]({{site.url}}/assets/images/prototype-1.png)

1. Prototype Interface: 복제를 위한 메서드를 선언한다.
2. ConcretePrototype: Prototype 인터페이스를 구현하며, 복제를 위한 메서드를 구현한다.
3. Client: Prototype 인터페이스를 사용한다.


## 프로토타입 패턴 코드 예시

Github Issue를 생성하는 예시를 들어보자. Github Issue를 생성하기위해서는 Github Repoisitory의 정보를 API로 선행해 가져와야한다고 가정해보자. 

이 때, Github Repository의 정보를 가져오는 API 호출은 리소스가 많이 들기 때문에 매번 새로운 Issue를 생성할 때마다 API를 호출하는 것은 비효율적이다.

이 때 프로토타입 패턴을 사용하면 기존 Issue를 복제해 새로운 Issue를 생성할 수 있다.

자바에서 Object class에 이미 clone() 메서드가 존재한다. 하지만 clone() 메서드는 Protected로 선언되어 있기 때문에 Public으로 접근제한을 바꾸려면 Cloneable 인터페이스를 구현해야한다.

만약 Cloneable 인터페이스를 구현하지 않으면 Object 클래스의 clone 메서드를 호출할 때 CloneNotSupportedException이 발생한다. 이는 복제가 허용되지 않았음을 의미한다. 따라서 Cloneable 인터페이스를 구현함으로써 해당 클래스의 객체가 복제 가능하다는 의도를 명시적으로 나타내고, 안전하게 복제 작업을 수행할 수 있습니다.

```kotlin


class GithubIssue(val repository: GithubRepository) : Cloneable {
  var id: Int = 0

  var title: String? = null

  @Throws(CloneNotSupportedException::class)
  override fun clone(): Any {
    val repository: GithubRepository = GithubRepository(this.repository.user, this.repository.name)

    val githubIssue = GithubIssue(repository)
    githubIssue.id = id
    githubIssue.title = title
    
    
    return githubIssue
  }
  
  
  
}


class GithubRepository(val user: string, val name: string)

fun main() {
  // Github Repository 정보를 API에서 가져온다고 가정
  // CALL API 
  val (user, name) = API.getUserAndName("URL")
  val repository: GithubRepository = GithubRepository(user, name)

  val issue = GithubIssue(repository)
  issue.id = 1
  issue.title = "Issue 1"

  // Issue 1을 복제해 새로운 Issue 2를 생성한다. API 호출 없이 새로운 Issue를 생성할 수 있다.
  val cloneIssue = issue.clone() as GithubIssue
  cloneIssue.id = 2
  cloneIssue.title = "Issue 2"

  

}

```

## Eqauls & Hashcode 구현하기 
기존 인스턴스와 클론된 인스턴스는 당연히 다른 인스턴스이다. clone method에서 새로운 인스턴스를 생성하기 때문이다. 

하지만 equlas는 인스턴스의 내부 값이 같은지를 비교하기 때문에 따로 구현해주어야하며 hashcode도 같이 구현해주어야한다. hashcode는 equals를 오버라이딩할 때 반드시 같이 오버라이딩해주어야한다. hash map같은 자료구조에서 key로 이 메서드를 사용해 인스턴스를 비교하기 때문이다.

아래는 인텔리제이가 자동으로 변수를 선택해 equals와 hashcode를 해준 코드이다.

```kotlin

override fun equals(other: Any?): Boolean {
  if (this === other) return true
  if (javaClass != other?.javaClass) return false

  other as GithubIssue

  if (repository != other.repository) return false
  if (id != other.id) return false
  if (title != other.title) return false

  return true
}

override fun hashCode(): Int {
  var result = repository.hashCode()
  result = 31 * result + id
  result = 31 * result + (title?.hashCode() ?: 0)
  return result
}

```

## Copy는 얕은 복사이다.

clone() 메서드는 얕은 복사를 수행한다. 즉, 객체의 참조만 복사한다. 따라서 객체의 참조가 가리키는 객체는 같은 객체를 가리킨다. 

이를 깊은 복사로 바꾸려면 clone() 메서드를 오버라이딩해 깊은 복사를 수행하면 된다. 

우리는 이미 위에서 참조하는 repository `객체를 새로 생성해` 깊은 복사를 수행했다. 


## 장단점

- 장점 
  - 생성 시 리소스가 많이 드는 객체를 효율적으로 생성할 수 있다.
- 단점 
  - 객체를 만드는 과정 자체가 복잡할 수 있다. 특히 깊은 복사를 수행해야할 때 더욱 복잡해진다. 



## Reference

- 코딩으로 학습하는 GoF의 디자인 패턴 - 백기선




