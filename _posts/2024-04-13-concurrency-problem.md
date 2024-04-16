---
layout: post
title: 동시성에서 발생할 수 있는 문제들
date: 2024-04-11 10:43 +0900
img_path: /assets/images/
category: [ concurrency ]
tags: [ Concurrency ]
---

동시성에서 발생할 수 있는 문제는 원자성과 가시성의 문제로 설명할 수 있다.

# 원자성 
원자성이란 멀티 쓰레드 환경에서 하나의 작업이 완전히 실행되거나 전혀 실행되지 않는 성질이다. 

멀티 쓰레드 환경에서는 여러 스레드가 동시에 실행되기 때문에 원자성을 지키지 못한다면 데이터의 무결성이 깨질 수 있다.

int형 변수에 1을 더하는 작업을 생각해보자.(i++) 이 작업은 1. 변수를 읽어들인다. 2. 변수에 1을 더한다. 3. 변수에 저장한다. 이렇게 3단계로 나눌 수 있다.

만약 단일 스레드로 하나씩 실행된다면 1,2,3 순서대로 실행되어 1이 더해진 값이 저장될 것이다.

하지만 멀티 쓰레드에서 1개 이상의 스레드가 동시에 이 연산이 실행된다면 어떻게 될까? 

만약 두 스레드가 동시에 변수를 읽어들이고 1을 더하고 저장한다면 두 스레드가 동시에 1을 더한 값이 저장될 것이다.

그렇다면 하나의 스레드의 연산은 무시하게 된 것이고 이는 원자성이 깨졌다고 할 수 있다.

## 경쟁 조건 (Race Condition)

원자성과 관계 깊은 개념으로 경쟁 조건이 있다.

경쟁 조건이란 두 개 이상의 스레드가 공유 자원에 접근해 동시에 데이터를 변경하려고 할 때 발생하는 문제를 뜻한다.

먼저 도착한 스레드가 값을 변경하더라도 뒤늦게 데이터를 변경한 스레드에 의해 변경된 데이터를 덮어쓰게 되어 결과가 달라질 수 있다.

예를들어, 어떤 쇼핑몰에서 비전 프로 100개를 선착순으로 싸게 살 수 있는 이벤트를 열었고 이벤트가 시작되자마자 100건의 주문이 들어 왔다고 가정해보자.

```kotlin

@Test
fun 동시에_100개의_요청하기() {
  val threadCount = 100
  val executorService = Executors.newFixedThreadPool(threadCount)
  val countDownLatch = CountDownLatch(threadCount)

  for (i in 1..threadCount) {
      executorService.submit {
        try{
          stockService.decreaseQuantity(`비전프로`)  
        }finally {
          countDownLatch.countDown()        
        }
      }
  }

  countDownLatch.await()

  val visionPro = productRepisotory.findById(`비전프로`).orElseThrow()
  assertEquals(0L, stock.quantity) // 재고가 0개가 되어야 한다.

  /**
   * 테스트 에러
   * Expected :0
   * Actual   :99
   */

}
```

주문이 100개 들어왔으니 재고가 0을 예상하지만 실제 남은 재고수는 훨씬 많은 수가 남아있다.

이 문제는 동시에 접속된 주문 스레드들 끼리 재고를 줄이는 작업과 동기화가 되어 있지 않기 때문에 발생한다.

모든 주문 스레드가 읽어온 재고 값 100을 기준으로 단지 -1을 하기 때문에 모든 스레드가 99개의 재고를 저장하게 되는 것이다.

혹은 다음과 같은 문제도 발생할 수 있다. 

마지막 남은 비전 프로를 구매하려는 100명의 고객이 거의 동시에 주문을 한다고 가정해보자.

실제로는 한명만 구매가 가능해야하는데 각각의 주문 스레드가 재고가 1개인 상태라고 알고 있기 때문에 실제 재고와는 상관없이 재고가 있다고 판단 되어 주문을 처리 할 수 있게 된다. 

결과적으로, 실제 재고보다 더 많은 주문이 완료되는 문제가 발생한다. 

```kotlin

 @Test
    fun 마지막_남은_비전프로_동시에_100개_주문하기(){
        val threadCount = 100
        val executorService = Executors.newFixedThreadPool(threadCount)
        val countDownLatch = CountDownLatch(threadCount)

        for (i in 1..threadCount) {
            executorService.submit {
                try {
                    orderService.createOrder(2, 1L)
                } finally {
                    countDownLatch.countDown()
                }
            }
        }

        countDownLatch.await()

        assertEquals(1,orderRepository.count()) // 실패 
    }
 /**
  * 테스트 에러 
  * Expected :1
  * Actual   :57 
  */

```


# 가시성 

멀티 코어에서 쓰레드 하나는 코어 하나를 사용하게 되면서 CPU Cache를 사용한다.

CPU Cache란 CPU 내부에 있는 메모리로, CPU가 메모리에 접근해 데이터를 읽어 올 때 일부분을 CPU Cache 메모리에 저장해 둔다. 

이러면 같은 데이터를 읽어 올 때 메모리에 직접 접근하는 것보다 빠르게 데이터를 읽어 올 수 있다.

![]({{site.url}}/assets/images/concurrency-visibility.png)

가시성이란 멀티 쓰레드 환경에서 하나의 스레드가 작업을 수행한 결과를 다른 스레드가 즉시 볼 수 있는지에 대한 것이다.

다시 말해, 한 쓰레드가 CPU Cache에서 작업을 수행한 결과를 다른 쓰레드에서도 볼 수 있는지이다.

여기서 발생할 수 있는 문제는 어떤 한 코어가 작업을 Cpu Cache에 해 아직 메모리에 반영되지 않은 상태에서, 다른 코어가 메모리에 접근해 데이터를 읽어 올 때 문제가 발생할 수 있다.

따라서 다른 스레드가 변경한 데이터를 즉시 볼 수 있도록 동기화를 해주어야 한다.

실제 예시 코드는 다음과 같다.
```kotlin

fun main() {
    var flag = true
    val threadCount = 2

    val executorService = Executors.newFixedThreadPool(threadCount)
    val countDownLatch = CountDownLatch(threadCount)

    // thread1
    executorService.submit {
        var count = 0
        while (flag) {
            count++
        }
        println("Count: $count")
        countDownLatch.countDown()
        
    }
    
   // thread2
    executorService.submit {
        Thread.sleep(100)
        
        flag = false
        println("Thread 2 Finished")
        countDownLatch.countDown()
    }

    countDownLatch.await()

}
 ```

맨 처음 Thread는 flag가 true인 동안 count를 1씩 증가시키는 작업을 수행한다. 

Thread 2는 100ms 후에 flag를 false로 변경하고 메세지를 출력한다.

예상하기로는 100ms 뒤에 flag가 false로 변경되어 Thread 1이 종료되어야 한다. 

하지만 실제로는 Thread 1이 종료되지 않고 계속 반복되는 것을 볼 수 있다.

이는 thread2가 flag 값을 Cache 메모리에만 반영하고 메모리에 반영되지 않아 Thread1이 flag 변경 값을 인지하지 못하기 때문이다.


## 마치며

동시성에서 발생할 수 있는 문제는 원자성과 가시성의 문제로 설명할 수 있다.

원자성은 멀티 쓰레드 환경에서 하나의 작업이 완전히 실행되거나 전혀 실행되지 않는 성질이다.

경쟁 조건은 두 개 이상의 스레드가 공유 자원에 접근해 동시에 데이터를 변경하려고 할 때 발생하는 문제를 뜻한다.

가시성은 멀티 쓰레드 환경에서 하나의 스레드가 작업을 수행한 결과를 다른 스레드가 즉시 볼 수 있는지에 대한 것이다.

다음은 글은 경쟁 조건을 해결하는 방법에 대해 알아본다.






