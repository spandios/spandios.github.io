---
layout: post
title: Command Pattern
date: 2024-07-15 08:00 +0900
img_path: /assets/images/
category: design-pattern
tags: [ design-pattern, 구조 관련 디자인 패턴 ]
---

## 커맨드 패턴 (Command Pattern)

요청을 캡슐화 하여 호출자(invoker)와 수신자(receiver)를 분리하는 패턴이다.

요청 자체 즉, 실행될 기능을 캡슐화하기 때문에 여러 기능이 실행할 수 있는 유연성을 가진다.

어떤 이벤트가 발생했을 때 그 이벤트를 발생시키는 클래스를 변경하지 않고 실행될 기능을 변경할 수 있다.

## 패턴 구조

![]({{site.url}}/assets/images/command.png)

1. Invoker: 요청을 보내는 호출 클래스
2. Command: 요청을 캡슐화하는 인터페이스로 execute 메서드를 정의한다.
3. ConcreateCommand: 실제 요청을 처리하는 클래스로 요청 처리에 필요한 모든 정보를 가지고 있다.
4. Receiver: ConcreateCommand 내부에서 실제로 기능을 실행하는 클래스 

## 패턴 적용 전

예를 들어 버튼이 있는데 이 버튼은 처음에는 불을 켜는 기능을 하지만 나중에는 알람이 울리는 기능을 하도록 변경해야 한다고 가정해보자.

초기 버튼 클래스와 램프 클래스는 다음과 같다.

```kotlin
class Lamp {
  fun turnOn() {
    println("Lamp On")
  }
}

class Button(val lamp: Lamp) {
  fun press() {
    lamp.turnOn()
  }
}

```

이 후 버튼이 알람을 울리는 기능을 하도록 변경해야 한다면 버튼 클래스를 수정해야 한다.

```kotlin

class Alarm {
  fun ring() {
    println("Alarm Ring")
  }
}

class Button(val alarm: Alarm) {
  fun press() {
    alarm.ring()
  }
}

```

만약 다양한 기능을 추가하고 싶다면 다음과 같이 버튼 클래스를 계속 수정해야 한다. 이는 SRP(Single Responsibility Principle)을 위반하는 것이다.

```kotlin

class Button(val alarm: Alarm?, val lamp: Lamp?) {
  fun press(isAlarm: Boolean, isLamp: Boolean) {
    if (alarm && isAlarm)
      alarm.ring()
    if (isLamp && lamp)
      lamp.turnOn()
  }
}

```

### 패턴 적용 후

이러한 문제를 해결하기 위해 요청자가 구체적인 기능을 직접 구현하는 대신 커맨드 객체를 통해 기능을 실행한다.

즉, 버튼 클래스의 pressed 메서드가 구체적인 기능에 대해 관여하는 게 아닌 캡슐화된 기능 자체를 외부에서 주입받아 실행하도록한다.

먼저 Command 인터페이스를 정의해보자. Command 인터페이스는 execute 메서드 하나만을 가진다. 자바에서는 하나의 메서드만 가진 인터페이스를 함수형 인터페이스라고 한다.

```kotlin

interface Command {
  fun execute()
}

```

이제 버튼은 Command 인터페이스를 주입받아 execute 메서드를 실행하도록 변경한다. setNewCommand 메서드를 통해 새로운 기능을 주입받을 수 있다.

```kotlin

class Button(val command: Command) {
  fun press() {
    command.execute()
  }
  
  fun setNewCommand(command: Command) {
    this.command = command
  }
}
```

이제 필요한 건 command 인터페이스를 구현한 클래스를 만드는 것이다. 불을 켜는 기능과 알람을 울리는 기능을 구현한 ConcreteCommand 클래스를 만들어보자.

```kotlin

class LampOnCommand(val lamp: Lamp): Command {
  override fun execute() {
    lamp.turnOn()
  }
}

class AlarmRingCommand(val alarm: Alarm): Command {
  override fun execute() {
    alarm.ring()
  }
}

```

이제 Client에서는 버튼 클래스에 필요한 기능을 주입하기만 하면 된다.

```kotlin

fun main(){
  val lamp = Lamp()
  val alarm = Alarm()
  
  val lampOnCommand = LampOnCommand(lamp)
  val alarmRingCommand = AlarmRingCommand(alarm)
  
  val buttonLamp = Button(lampOnCommand)
  val buttonAlarm = Button(alarmRingCommand)
  
  buttonLamp.press()
  buttonAlarm.press()
}

```

만약 등과 알림이 동시에 실행되어야 한다면 다음과 같이 새로운 커맨드를 만들어 버튼에 주입하기만 하면 된다.

```kotlin

class LampAndAlarmCommand(val lamp: Lamp, val alarm: Alarm): Command {
  override fun execute() {
    lamp.turnOn()
    alarm.ring()
  }
}


fun main(){
  val lamp = Lamp()
  val alarm = Alarm()
  val lampAndAlarmCommand = LampAndAlarmCommand(lamp, alarm)
  val buttonLampAndAlarm = Button(lampAndAlarmCommand)
  buttonLampAndAlarm.press()
}

```

여기서 invoker는 버튼 클래스이고 receiver는 Lamp와 Alarm 클래스이다. 

## 장단점

### 장점

- 요청하는 쪽과 처리하는 쪽의 결합도를 낮춰 유연성을 높일 수 있다.
- 새로운 기능을 기존 코드를 수정하지 않고도 추가할 수 있고 재사용성이 높다.

### 단점

- 커맨드 객체가 많아지면 관리하기 어렵다.

## Reference

- 코딩으로 학습하는 GoF의 디자인 패턴 - 백기선
- https://gmlwjd9405.github.io/2018/07/07/command-pattern.html




