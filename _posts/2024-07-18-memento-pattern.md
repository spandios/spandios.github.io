---
layout: post
title: Memento Pattern
date: 2024-07-17 08:00 +0900
img_path: /assets/images/
category: design-pattern
tags: [ design-pattern, 구조 관련 디자인 패턴 ]
---

## 메멘토 패턴 (Mediator Pattern)
 
캡슐화를 유지하면서 객체 내부 상태를 외부에 저장하는 패턴이다. 객체의 상태를 저장하고 복구하는 데 사용된다.


## 패턴 구조

![]({{site.url}}/assets/images/memento.png)


1. Originator: 현재 상태를 저장하고 다시 상태를 복구하는 가장 중요한 역할을 함. 메멘토를 만드는 함수와 다시 메멘토를 통해 상태를 복구하는 함수를 갖고 있다.
2. Memento: 복원하고자하는 상태이다. 이 내부 상태는 Immuatbe하다.
3. CareTaker : Memento를 관리하는 역할을 함. Memento를 저장하고 복구하는 역할을 함. 필요할 때 Originator에게 Memento를 전달한다.

## 패턴 적용 전

Game 클래스가 있고 이 클래스는 게임의 스코어를 관리한다. 스코어를 저장하고 다시 불러오는 기능을 구현해보자.

```kotlin
  
class Game(val redTeamScore: Int, val blueTeamScore:Int){}

fun main(){
  val game = Game(10, 20)
  
  // 게임이 끝나면 스코어를 저장
  val blueTeamScore = game.blueTeamScore
  val redTeamScore = game.redTeamScore
  
  // 게임이 끝나고 다시 시작
  val newGame = Game(redTeamScore, blueTeamScore)
  
}
```

이렇게 구현하는 코드는 클라이언트가 너무 많은 책임을 가지고 있다. 게임의 스코어를 저장하고 복구하는 책임을 클라이언트가 가지고 있을뿐만아니라 게임 클래스 내부의 정보까지도 알고 있어야 한다.

이를 해결하기 위해서 메멘토 패턴을 적용해보자
 
## 패턴 적용 후

먼저 메멘토 클래스를 만들어보자. 메멘토 클래스는 복구할 상태이다.

```kotlin
class GameSave(val redTeamScore: Int, val blueTeamScore:Int) 

```

이후 Originator 클래스를 만들어보자. Originator 클래스는 현재 상태를 저장하고 복구하는 역할을 한다.

기존 Game 클래스에 메멘토를 저장하고 복구하는 기능을 추가해보자.


```kotlin
class Game(val redTeamScore: Int, val blueTeamScore:Int){
  fun saveGame(): GameSave{
    return GameSave(redTeamScore, blueTeamScore)
  }
  
  fun restoreGame(gameSave: GameSave){
    redTeamScore = gameSave.redTeamScore
    blueTeamScore = gameSave.blueTeamScore
  }
}
  

```
  
이후 굳이 필요없지만 CareTaker 클래스를 만들어보자. CareTaker 클래스는 Memento를 관리하는 역할을 한다.
```kotlin

class CareTaker{
  val gameSaves = mutableListOf<GameSave>()
  
  fun addGameSave(gameSave: GameSave){
    gameSaves.add(gameSave)
  }
  
  fun getGameSave(index: Int): GameSave{
    return gameSaves[index]
  }
}

```

이제 클라이언트 코드를 작성해보자. 메멘토 패턴을 사용했기 때문에 클라이언트는 게임의 스코어를 저장하고 복구하는 책임을 가지지 않고 위임한다.

```kotlin

fun main(){
  val game = Game(10, 20)
  val careTaker = CareTaker()
  
  // 게임이 끝나면 스코어를 저장
  careTaker.addGameSave(game.saveGame())
  
  // 게임이 끝나고 다시 시작했을 때 저장된 스코어를 복구
  val newGame = Game(0, 0)
  newGame.restoreGame(careTaker.getGameSave(0))
  
}
```




## 장단점

### 장점

- 캡슐화를 지키면서 객체 상태 스냅샷을 만들 수 있다. 
- 객체의 상태를 저장하고 복구하는 책임을 클라이언트가 가지지 않아도 된다.


### 단점

- 많은 정보를 저장하는 Mementor를 자주 생성할 경우 메모리 사용량에 영향을 준다.

## Reference

- 코딩으로 학습하는 GoF의 디자인 패턴 - 백기선




