---
layout: post
title: Iterator Pattern
date: 2024-07-17 08:00 +0900
img_path: /assets/images/
category: design-pattern
tags: [ design-pattern, 구조 관련 디자인 패턴 ]
---

## 이터프리터 패턴 (Iterator Pattern)

집합 객체 내부 구조를 노출시키지 않고 순회 하는 방법을 제공 하는 패턴이다.

집합 객체를 순회하는 클라이언트 코드를 변경하지 않고도 다양한 방식으로 순회할 수 있다.

## 패턴 구조

![]({{site.url}}/assets/images/iterator.png)

1. Iterator: 순회하는 인터페이스를 정의하는 클래스  
2. ConcreteIterator: Iterator 인터페이스를 구현하는 클래스
3. Aggregate: 집합 객체를 정의하는 클래스, Iterator를 반환함
4. ConcreteAggregate: Aggregate 인터페이스를 구현하는 클래스

## 패턴 적용 전

칠판에 post를 붙이는 프로그램을 만들어보자.

```kotlin
import java.time.LocalDateTime

class Post(val content: String, val createdAt: Date = LocalDateTime.now()) {
  override fun toString(): String {
    return "Post(content='$content', createdAt=$createdAt)"
  }
}

class Board {
  val posts = mutableListOf<Post>()

  fun addPost(post: Post) {
    posts.add(post)
  }
}


fun main() {
  val board = Board()
  board.addPost(Post("Hello"))
  board.addPost(Post("World"))
  board.addPost(Post("Kotlin"))
    
    // 게시글 출력
  for (post in board.posts) {
    println(post.content)
  }
  
  // 최신 글 먼저 출력
  board.posts.sortByDescending { it.createdAt }.forEach {
    println(it.content)
  }
}

```
문제는 이렇게 작성된 코드는 클라이언트가 집합 객체를 직접 순회하기 때문에 집합 객체의 내부 구조가 변경되면 클라이언트 코드도 변경해야 한다. 

따라서 클라이언트는 내부 구조를 모르게 하고 집합 객체를 순회할 수 있는 방법을 제공해야 한다.

## 패턴 적용 후

자바에는 Iterator 인터페이스가 있어서 이를 사용하면 된다.

최근 글을 순회하는 Iterator를 구현해보자. 이 후 Board에서 이 Iterator를 반환하는 메서드를 추가한다.

```kotlin

class RecentPostIterator(board: Board) : Iterator<Post> {
  
  private val internalIterator: Iterator<Post> = board.posts.sortedByDescending { it.createdAt }.iterator()
  
  override fun hasNext(): Boolean {
    return this.internalIterator.hasNext()
  }

  override fun next(): Post {
    return this.internalIterator.next()
  }
}

class Board{
  val posts = mutableListOf<Post>()
  
  fun addPost(post: Post) {
    posts.add(post)
  }
  
  fun recentPosts(): Iterator<Post> {
    return RecentPostIterator(this)
  }
    
  fun getRecentPostIterator() = RecentPostIterator(this)
}


```

이제 클라이언트 코드는 집합 객체의 내부 구조를 알 필요 없이 집합 객체를 순회할 수 있다.

```kotlin
  
fun main(){
  
  val board = Board()
  board.addPost(Post("Hello"))
  board.addPost(Post("World"))
  board.addPost(Post("Kotlin"))
  
  val recentPostIterator = board.getRecentPostIterator()
  while(recentPostIterator.hasNext()){
    println(recentPostIterator.next())
  }
  
}
```

## 장단점

### 장점

- 집합 객체가 가지고 있는 객체들에 손쉽게 접근 가능하다.
- 일관된 인터페이스를 사용해 여러 형태의 집합 구조를 순회 가능하다.
### 단점

- 클래스가 늘어나고 복자도가 증가한다.

## Reference

- 코딩으로 학습하는 GoF의 디자인 패턴 - 백기선




