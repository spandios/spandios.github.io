---
layout: post
title: Facade Pattern
date: 2024-07-09 08:00 +0900
img_path: /assets/images/
category: design-pattern
tags: [ design-pattern, 구조 관련 디자인 패턴 ]
---

## 퍼싸드(Facade) 패턴

퍼싸드란 프랑스어로 '건물의 정면'을 뜻한다. 

우리가 건물의 정면을 보고 있을 때 그 안의 수도나 전기 등의 설비는 볼 수 없지만, 그것들을 사용할 수 있는 것처럼 퍼싸드 패턴은 복잡한 시스템의 내부를 감추고 단순한 인터페이스로 제공하는 패턴이다.

다시말해 클라이언트가 복잡한 시스템인 프레임워크나 라이브러리를 직접 사용하는게 아니라 퍼싸드가 그 중간에서 디테일한 부분은 감추고 간소화된 인터페이스로 클라이언트에게 제공하는 패턴이다.

## 퍼싸드 패턴 구조

![]({{site.url}}/assets/images/facade.png)

1. Facade: 복잡한 시스템의 기능을 간소화하여 제공하는 클래스
2. SubSystem: 실제 복잡한 시스템의 기능들

## 패턴 적용 전 

JAVA를 이용해 메일을 보내는 예제를 살펴보자.

```kotlin

fun main(){
  val to = "wndudpower@gmail.com"
  val from = "spandios.github.io"
  val host = "localhost"
  
  val properties = System.getProperties()
  properties.setProperty("mail.smtp.host", host)
  
  val session = Session.getDefaultInstance(properties)
  
  try {
    val message = MimeMessage(session)
    message.setFrom(InternetAddress(from))
    message.addRecipient(Message.RecipientType.TO, InternetAddress(to))
    message.setSubject("This is the Subject Line!")
    message.setText("This is actual message")
    
    Transport.send(message)
  } catch (e: MessagingException) {
    e.printStackTrace()
  }
}


```

현재 클라이언트가 너무 복잡하고 구체적인 부분을 다 알고있고 사용하고 있다. 

이는 한 시스템에 의존성이 너무 높은 코드이고 유지보수에 영향을 주게 될 것이다. 이제 퍼싸드 패턴을 적용해보자. 

### 패턴 적용 후

퍼싸드 클래스는 EmailSender 인터페이스를 구현하도록 정의하도록 했다. EmailSender는 EmailSetting과 EmailMessage를 사용한다.

EmailSetting은 이메일 호스트를 설정하는 클래스이고, EmailMessage는 이메일의 제목, 내용, 참조, 숨은 참조를 설정하는 클래스이다.

이제 클라이언트는 크게는 EmailSender의 `sendEmail`함수만 사용하면 되고 내부적인 동작은 EmailSender가 알아서 처리하게 된다.

```kotlin

class EmailMessage(val subject: String, val text: String, val cc: List<String>?, val bcc: List<String>?)

interface EmailSender {
  fun sendEmail(to:String, from:String, message: EmailMessage)
}

class JavaEmailSender(val emailSetting: EmailSetting) : EmailSender{


  fun sendEmail(to: String, from: String, emailMessage: EmailMessage) {
    val properties = System.getProperties()
    properties.setProperty("mail.smtp.host", emailSetting.host)

    val session = Session.getDefaultInstance(properties)

    try {
      val message = MimeMessage(session)
      message.setFrom(InternetAddress(from))
      message.addRecipient(Message.RecipientType.TO, InternetAddress(to))
      message.setSubject(emailMessage.subject)
      message.setText(emailMessage.text)

      Transport.send(message)
    } catch (e: MessagingException) {
      e.printStackTrace()
    }
  }
}

class EmailSetting(val host: String)

fun main() {
  val emailSetting = EmailSetting("localhost")
  val emailSender = JavaEmailSender(emailSetting)

  val emailMessage = EmailMessage("This is the Subject Line!", "This is actual message", null, null)
  emailSender.sendEmail("wndudpower@gmail.com", "spandios.github.io", emailMessage)

}

```



## 장단점

### 장점

- 서브 시스템에 대한 의존성을 한 곳으로 집중시킬 수 있다.


### 단점 

- 퍼싸드 클래스가 너무 많은 기능을 가지게 되면 퍼싸드 클래스가 복잡해질 수 있다.


## 실제 사례

- Collections.checkedList

기존 List에 타입을 체크하는 기능을 추가하는 `checkedList`는 데코레이터 패턴을 사용한다. 

```kotlin
val list = mutableListOf()

list.add(Book("Java"))
val checkedList = Collections.checkedList(list, Book::class) // Book 타입만 추가할 수 있는 리스트
list.add("Kotlin") // ClassCastException 발생


```

- Collections.unmodifiableList

기존 List에 수정을 막는 기능을 추가하는 `unmodifiableList`는 데코레이터 패턴을 사용한다.

```kotlin

val list = mutableListOf("Java", "Kotlin")

val unmodifiableList = Collections.unmodifiableList(list)

unmodifiableList.add("Python") // UnsupportedOperationException 발생

```

- HttpServletRequestWrapper : HttpServletRequest를 상속받아 기능을 확장하는 클래스 

HttpRequest 해당 요청의 파라미터를 대문자로 변환하는 예제

```java

public class MyHttpServletRequestWrapper extends HttpServletRequestWrapper{
  public MyHttpServletRequestWrapper(HttpServletRequest request){
    super(request);
  }
  
  @Override
  public String getParameter(String name){
    String value = super.getParameter(name);
    return value == null ? null : value.toUpperCase();
  }
}

```


## Reference

- 코딩으로 학습하는 GoF의 디자인 패턴 - 백기선




