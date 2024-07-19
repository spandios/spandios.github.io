---
layout: post
title: State Pattern
date: 2024-07-18 08:00 +0900
img_path: /assets/images/
category: design-pattern
tags: [ design-pattern, 구조 관련 디자인 패턴 ]
---

## 상태 패턴 (State Pattern)

객체 내부 상태 변경에 따라 객체의 행동이 달라지는 패턴이다. 

상태에 특화된 행동을 분리할 수 있고, 새로운 행동을 추가하더라도 다른 행동에 영향을 주지 않는다.

## 패턴 구조

![]({{site.url}}/assets/images/state.png)

1. State : 컨텍스트가 변경될 수 있는 여러 상태들에 대한 인터페이스를 정의한다. 
2. Context : 상태 객체를 가지고 있는 클래스이다. 상태를 추가하고 변경하는 역할을 한다.
3. ConcreteState : State 인터페이스를 구현한 클래스이다. 상태에 따라 다른 행동을 여기서 정의한다.

## 패턴 적용 전

온라인 강의가 있고 이 강의는 3가지 상태 DRAFT, PUBLISHED, PRIVATE을 가진다고 가정해보자. 

그리고 이 강의에는 리뷰와 수강생의 목록을 가지고 있다.

```kotlin


enum class LectureState {
  DRAFT, PUBLISHED, PRIVATE
}

class Lecture {
  var state: LectureState = LectureState.DRAFT
  val reviews = mutableListOf<String>()
  val students = mutableListOf<String>()

  fun changeState(state: LectureState) {
    this.state = state
  }

  fun addReview(review: String, student: Student) {
    if (this.state == LectureState.DRAFT || this.state == LectureState.PUBLISHED) {
      reviews.add(review)
    } else if (this.state == LectureState.PRIVATE && students.contains(student)) {
      reviews.add(review)
    } else {
      throw UnsupportedOperationException("리뷰를 작성할 수 없습니다.")
    }
  }

  fun addStudent(student: Student) {
    if (this.state == LectureState.PUBLISHED) {
      students.add(student)
    } else if (this.state == LectureState.PRIVATE && availableTo(student)) {
      students.add(student)
    } else {
      throw UnsupportedOperationException("학생을 해당 수업에 추가할 수 없습니다.")
    }

    if (students.size > 1) {
      this.state = LectureState.PRIVATE
    }
  }

  private fun availableTo(student: Student): Boolean {
    return student.isEnabledForPrivateClass(this)
  }
}


```
위의 예제에서는 강의의 상태에 따라 리뷰를 추가하거나 학생을 추가하는 행동이 달라진다. 

위의 예제처럼 상태에 따라 행동을 분기하는 코드를 작성하면 상태가 추가될 때마다 코드가 복잡해진다. 실제로 위의 코드는 읽기가 쉽지 않다. 

## 패턴 적용 후 

먼저 상태(State)를 인터페이스로 정의해보자. 상태에 따라 행동이 달라지는 메소드를 정의한다. if나 switch문을 사용하는 곳이 있다면 이를 인터페이스로 추상화한다. 

```kotlin

interface LectureState {
  fun addReview(review: String, student: Student, lecture: Lecture)
  fun addStudent(student: Student, lecture: Lecture)
}

```

이제 Lecture 클래스에서 상태에 따라 행동을 각 ConcreteState 클래스로 위임하도록 변경해보자.

```kotlin

class Lecture {
  var state: LectureState = DraftState(this)

  val reviews = mutableListOf<String>()
  val students = mutableListOf<String>()

  fun changeState(state: LectureState) {
    this.state = state
  }

  fun addReview(review: String, student: Student) {
    state.addReview(review, student, this)
  }

  fun addStudent(student: Student) {
    state.addStudent(student, this)
  }
}


```

이제 상태 (Draft, Published, Private) 클래스를 만들어보자.

각 상태에 따라 변경 되는 행동을 정의하면 된다. 

```kotlin

class PrivateState(private val lecture: Lecture) : LectureState {
  override fun addReview(review: String, student: Student, lecture: Lecture) {
    if (lecture.students.contains(student)) {
      lecture.reviews.add(review)
    } else {
      throw UnsupportedOperationException("프라이빗 코스를 수강하는 학생만 리뷰를 작성할 수 있습니다.")
    }
  }

  override fun addStudent(student: Student, lecture: Lecture) {
    if (student.isAvailable(lecture)) {
      lecture.students.add(student)
    } else {
      throw UnsupportedOperationException("해당 학생은 프라이빗 코스를 수강할 수 없습니다.")
    }
  }
  

class DraftState(private val lecture: Lecture) : LectureState {
  override fun addReview(review: String, student: Student, lecture: Lecture) {
    throw UnsupportedOperationException("드래프트 상태에서는 리뷰를 남길 수 없습니다.")
  }

  override fun addStudent(student: Student, lecture: Lecture) {
    lecture.students.add(student)
    if(lecture.students.size > 1) {
      lecture.changeState(PrivateState(lecture))
    }
  }
}

class PublishedState(private val lecture: Lecture) : LectureState {
  override fun addReview(review: String, student: Student, lecture: Lecture) {
    lecture.reviews.add(review)
  }

  override fun addStudent(student: Student, lecture: Lecture) {
    lecture.students.add(student)
  }
}


}

```

이제 실제 사용하는 코드에서는 상태에 따라 행동을 위임하는 코드만 작성하면 된다.

```kotlin

val lecture = Lecture()
val student = Student("student1")
val student2 = Student("student2")
student2.addPrivateClass(lecture) // student2은 프라이빗 코스를 수강할 수 있다.
lecture.addStudent(student) // 현재 Draft 상태로 student1이 수강생으로 추가된다.
lecture.changeState(PrivateState(lecture)) // Private 상태로 변경된다.
lecture.addReview("good", student) // student1은 lecture의 수강생이므로 리뷰를 남길 수 있다.
lecture.addStudent(student2) // student2는 프라이빗 코스를 수강할 수 있으므로 수강생으로 추가된다.


```


## 장단점

### 장점

- 상태에 따른 동작을 개별 클래스로 옮겨서 관리 가능하다.
- 기존 특정 상태에 따른 동작을 변경하지 않고도 새로운 상태에 따른 동작을 추가할 수 있다.
- 상태에 따른 동작을 분리함으로써 코드의 가독성이 높아진다.

### 단점

- 복잡도가 증가한다.

## Reference

- 코딩으로 학습하는 GoF의 디자인 패턴 - 백기선




