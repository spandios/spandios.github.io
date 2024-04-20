---
layout: post
title: 동시성 제어  - 프로그래밍 언어
date: 2024-04-18 10:43 +0900
img_path: /assets/images/
category: [ concurrency ]
tags: [ Concurrency ]
---

이제 전에 글에서 알아보았던 동시성에서 발생할 수 있는 원자성과 가시성의 문제를 언어수준에서 해결하는 방법에 대해 알아보려 한다.

그 전에 관련된 개념을 간단히 살펴보자.

- 임계영역(Critical Section): 한 번에 한 스레드만 접근할 수 있는 영역을 뜻한다.
- 상호 배제(Mutual Exclusion): 임계 영역에 접근하는 스레드가 다른 스레드에게 접근하지 못하도록 하는 것을 뜻한다. 이를 줄여서 Mutex라고도 한다.
- 세마 포어(Semaphore): 동기화 방법 중 하나로, 임계 영역에 접근할 수 있는 스레드의 수를 제한하는 방법이다.
- 모니터 (Monitor): 실제 상호 배제가 가능하게 하는 고수준의 동기화 방법이다. 자바에서 모든 객체는 모니터 잠금을 가지고 있으며, 잠금을 획득한 스레드만이 임계 영역에 접근할 수 있다. 자바에서 동기화
  기법은 모니터를 사용한다.

# synchronized

synchronized는 자바에서 제공하는 키워드로, 한 스레드가 잠금을 가지고 있다면 다른 스레드는 임계 영역에 접근하지 못하도록 하는 기술이다. 다시말해 모니터를 사용해 상호 배제를 구현하는 방법이다.

synchornized는 원자성을 보장한다. 하나의 스레드가 이미 임계 영역에 접근해 있다면 온전히 실행을 마칠 때까지 다른 스레드는 접근하지 못하기 때문이다.

또 가시성도 보장해준다. 한 스레드가 synchronized 블록 내에서 변수를 수정하고 스레드를 종료할 때 수정된 값이 메인 메모리에 반영됩니다. 그 후 다른 스레드가 synchronized 블록을 진입하면, 그
스레드는 메인 메모리에서 최신의 값을 읽게 된다.

synchronized는 메서드 자체에 사용할 수도 있고, 특정 객체에만 임계영역을 설정해 사용할 수도 있다.

코틀린에서는 `@Synchronized` 어노테이션을 사용하면 된다.

아래 코드는 synchronized를 사용한 예시이다.

```kotlin

class Stock(val name: String) {
  var price = 0

  @Synchronized
  fun upPrice(price: Int) {
    println("UP PRICE")
    val oldPrice = this.price
    this.price += price

    try {
      Thread.sleep(Random.nextInt(100).toLong())
    } catch (e: InterruptedException) {
      e.printStackTrace()
    }

    // 다른 스레드가 price를 변경했는지 확인
    if (this.price != oldPrice + price) {
      println("Thread unsafe")
    }
  }


  @Synchronized
  fun downPrice(price: Int) {
    println("DOWN PRICE")
    val oldPrice = this.price
    this.price -= price

    try {
      Thread.sleep(Random.nextInt(100).toLong())
    } catch (e: InterruptedException) {
      e.printStackTrace()
    }

    // 다른 스레드가 price를 변경했는지 확인
    if (this.price != oldPrice - price) {
      println("Thread unsafe")
    }
  }
}



```

Stcok 클래스는 price를 가지고 있고, upPrice와 downPrice 메서드를 가지고 있다. 각 메서드는 `@Synchronized` 키워드를 사용해 한번에 한 스레드만 접근할 수 있도록 한다.

만약 @Synchronized 키워드를 사용하지 않는다면, 다른 스레드가 price를 변경한 것을 확인하고 Thread unsafe를 출력한다.

@Synchronized 키워드를 사용하면 한 스레드가 메서드에 있다면 다른 스레드가 도중에 접근하지 못하기 때문에 price 또한 변경하지 못한다.

```kotlin

fun main() {

  val threadCount = 50

  val executorService = Executors.newFixedThreadPool(threadCount)
  val countDownLatch = CountDownLatch(threadCount * 2)

  val stock = Stock("APPLE")

  for (i in 0 until threadCount) {
    executorService.submit {
      try {
        stock.upPrice(i)

      } finally {
        countDownLatch.countDown()
      }
    }

    executorService.submit {
      try {
        stock.downPrice(i)
      } finally {
        countDownLatch.countDown()
      }
    }


  }

  countDownLatch.await()
  executorService.shutdown()

  println("STOCK PRICE : ${stock.price}")


}

```

이번엔 메서드 전체가 아니라 특정 객체에만 임계 영역을 설정해보자.
lock 객체를 생성하고, synchronized(lock)을 사용해 임계 영역을 설정해서 사용했다.

```kotlin
class Stock(val name: String) {
  var price = 0

  val lock = Any()

  fun upPrice(price: Int) {
    synchronized(lock) {
      println("UP PRICE")
      val oldPrice = this.price
      this.price += price

      try {
        Thread.sleep(Random.nextInt(100).toLong())
      } catch (e: InterruptedException) {
        e.printStackTrace()
      }

      // 다른 스레드가 price를 변경했는지 확인
      if (this.price != oldPrice + price) {
        println("Thread unsafe")
      }
    }
  }

  fun downPrice(price: Int) {
    synchronized(lock) {
      println("DOWN PRICE")
      val oldPrice = this.price
      this.price -= price

      try {
        Thread.sleep(Random.nextInt(100).toLong())
      } catch (e: InterruptedException) {
        e.printStackTrace()
      }

      // 다른 스레드가 price를 변경했는지 확인
      if (this.price != oldPrice - price) {
        println("Thread unsafe")
      }
    }
  }
}
```

# Atomic Type

Atomic Type은 원자적 연산을 지원하는 데이터 타입이다. atomic type을 사용하는 이유는 무엇일까? synchronized는 임계 영역에 접근하는 스레드가 다른 스레드에게 접근하지 못하도록 잠금을 사용한다. 
이 때 대기하고 있는 스레드는 잠금을 획득할 때까지 아무것도 할 수 없어 성능이 떨어질 수 밖에 없다.

반면 atomic type은 잠금이 아니라 CAS라는 비차단 알고리즘을 사용한다.
CAS는 값을 변경하기 전 알고 있던 기존 값과 현재 메인 메모리에 저장된 값을 비교하여 일치하는 경우 새로운 값으로 교체하고, 일치 하지 않는 다면 실패 처리를 한다.
만약 알고 있던 기존 값과 메인 메모리에 저장된 값이 일치하지 않는다면, 다른 스레드에 의해 이미 값이 변경된 것이므로 원자성을 보장하지 못하기 때문에 다시 시도해야 하는 것이다.

atomic type은 synchronized와 다르게 잠금을 강제하는게 아니라 실패 처리를 어떻게 할지 개발자가 정할 수 있기 때문에 스레드 대기와는 다르게 다른 작업을 할 수 있는 유연성을 가지며, 락 처리에 대한 오버헤드가 적다. 

하지만 복잡한 조건이나 여러 연산을 조작해야하는 경우에는 synchronized를 사용하는 것이 좋다.

위의 예시가 사실 atomic type을 사용하는 것이 더 적합하다. price를 변경할 때, 다른 스레드가 price를 변경했는지 확인하는 코드가 있었는데, 이를 atomic type을 사용하면 더 간단하게 구현할 수 있다.

```kotlin

class Stock2(val name: String) {
  val price = AtomicInteger(0)

  fun upPrice(increment: Int) {
    println("UP PRICE")
    var success = false
    do{
      val current = price.get()
      success = price.compareAndSet(current, current + increment)
    } while (!success)

    try {
      Thread.sleep(Random.nextInt(100).toLong())
    } catch (e: InterruptedException) {
      e.printStackTrace()
    }
  }

  fun downPrice(decrement: Int) {
    println("DOWN PRICE")
    var success = false
    do{
      val current = price.get()
      success = price.compareAndSet(current, current - decrement)
    } while (!success)

    try {
      Thread.sleep(Random.nextInt(100).toLong())
    } catch (e: InterruptedException) {
      e.printStackTrace()
    }
  }
}

```

price를 AtomicInteger로 변경하고 compareAndSet 메서드를 사용해 값을 변경한다. compareAndSet 메서드는 현재 메모리에 저장된 값과 기존 값을 비교하여 일치하면 새로운 값을 저장하고, 일치하지 않으면 실패 처리를 한다.

만약 compareAndSet 결과 값이 false이면 실패이기 때문에 계속 다시 시도해 원자성을 보장한다.


# volatile

volatile 키워드가 붙은 변수는 한 스레드에서 값이 변경되면 변경된 값이 다른 스레드에 즉시 보이게 된다. 즉, 한 스레드가 volatile 변수를 수정하면, 그 변경 사항이 모든 다른 스레드에게 반영되어 보이게 된다.

바로 코드로 알아보자.

```kotlin
fun main() {
    val threadCount = 2
    val executorService = Executors.newFixedThreadPool(threadCount)
    val countDownLatch = CountDownLatch(threadCount)  // upPrice와 downPrice 각각에 대한 카운트

    var running = true


    executorService.submit {
        var count = 0
        while (running) {
            count++
        }
        countDownLatch.countDown()
    }
    executorService.submit {
        try {
            Thread.sleep(1000)
            running = false
        } catch (e: InterruptedException) {
            e.printStackTrace()
        } finally {
            countDownLatch.countDown()
        }
    }


    countDownLatch.await()  // 모든 작업이 완료될 때까지 기다립니다.
    executorService.shutdown()


    println("All threads finished")
}

```

위 코드는 running 변수가 true일 때 계속해서 count를 증가시키는 스레드와 1초 뒤 running 변수를 false로 변경하는 스레드를 실행한다.

스레드 2가 running 변수를 false로 변경(CPU Cache에만)해도 스레드 1은 계속해서 무한 반복되어 끝나지 않는다. 

스레드1이 알고 있는 running 변수는 스레드2가 실제 변경한 값이 아니라, 스레드1이 처음에 알고 있던 값이기 때문이다. 

이 문제를 해결하려면 volatile 키워드를 사용해야 한다.

```kotlin
class Main {
    @Volatile
    private var running = true

    fun start() {
        val threadCount = 2
        val executorService = Executors.newFixedThreadPool(threadCount)
        val countDownLatch = CountDownLatch(threadCount)  // upPrice와 downPrice 각각에 대한 카운트

        executorService.submit {
            var count = 0
            while (running) {
                count++
            }
            countDownLatch.countDown()
        }
        executorService.submit {
            try {
                Thread.sleep(1000)
                running = false
            } catch (e: InterruptedException) {
                e.printStackTrace()
            } finally {
                countDownLatch.countDown()
            }
        }

        countDownLatch.await()  // 모든 작업이 완료될 때까지 기다립니다.
        executorService.shutdown()

        println("All threads finished")
    }
}

fun main() {
    val main = Main()
    main.start()
}
```

running 변수에 @Volatile 키워드를 붙여주면, 스레드 2가 running 변수를 변경하면 그 즉시 메인메모리에도 적용되기 때문에 스레드 1도 즉시 변경된 값을 알 수 있게 된다.

따라서 while문이 무한 반복 되지 않고 All threads finished가 출력되는 것을 볼 수 있다.

# ReentrantLock

ReentrantLock은 synchronized와 같은 기능을 제공하는 클래스이다. synchronized와 다르게 ReentrantLock은 락을 획득한 스레드가 다시 락을 획득할 수 있는 재진입이 가능하다.

주로 재귀 호출이나 다른 메서드를 호출할 때 사용할 때 주로 사용되는데 잠금을 획득한 상태에서 다른 메서드를 호출해야 하는 경우에 사용한다.

개발자가 직접 락을 걸고 다시 락을 해제하는 작업을 할 수 있기 때문에 synchronized보다 더 세밀한 제어가 가능한 장점이 있지만 그 만큼 신경을 더 써야한다. 

잠금을 획득하면 임계 영역에 다른 스레드는 접근하지 못하기 때문에 원자성을 보장하며, 잠금을 해제하고 다른 스레드가 잠금을 획득할 때 메인 메모리에서 최신 값을 얻기 때문에 가시성도 보장한다.

ReentrantLock은 lock()과 unlock() 메서드를 사용해 잠금을 획득하고 해제한다.

```kotlin
class ReentrantProcess {
  private val lock = ReentrantLock()

  fun initialProcess() {
    lock.lock()
    try {
      println("Initial process started. lock count : ${lock.holdCount}")
      nestedProcess()
    } finally {
      lock.unlock()
      println("Initial process unlock. lock count : ${lock.holdCount}")
    }
  }

  private fun nestedProcess() {
    lock.lock()
    try {
      println("Nested process executed. lock count : ${lock.holdCount}")
    } finally {
      lock.unlock()
      println("Nested process unlock. lock count : ${lock.holdCount}")
    }
  }


}

fun main() {
  val reentrantProcess = ReentrantProcess()

  val threadCount = 10
  val executorService = Executors.newFixedThreadPool(threadCount)
  val countDownLatch = CountDownLatch(threadCount)  // upPrice와 downPrice 각각에 대한 카운트


  executorService.submit {
    reentrantProcess.initialProcess()
    countDownLatch.countDown()
  }


  countDownLatch.await()  // 모든 작업이 완료될 때까지 기다립니다.
  executorService.shutdown()
  
  // Initial process started. lock count : 1
  // Nested process executed. lock count : 2
  // Nested process unlock. lock count : 1
  // Initial process unlock. lock count : 0
}

```

initialProcess 메서드에서 lock을 획득하고 nestedProcess 메서드를 호출한다. nestedProcess 메서드에서도 lock을 획득하고 해제하고 있다.

## 마치며

syncronized, atomic type, volatile, ReentrantLock은 동시성 문제를 해결하는 방법들이다.

synchronized는 임계영역에 다른 스레드에게 접근하지 못하도록 잠금을 사용하는 강력한 방법이지만 성능에 영향을 미칠 수 있다. 

atomic type은 원자적 연산을 지원하는 데이터 타입이다. synchronized와 다르게 잠금을 강제하는게 아니라 실패 처리를 어떻게 할지 개발자가 정할 수 있기 때문에 스레드 대기와는 다르게 다른 작업을 할 수 있는 유연성을 가지며, 락 처리에 대한 오버헤드가 적다.

volatile는 한 스레드에서 값이 변경되면 변경된 값이 다른 스레드에 즉시 보이게 된다. 즉, 한 스레드가 volatile 변수를 수정하면, 그 변경 사항이 모든 다른 스레드에게 반영되어 보이게 된다.

ReentrantLock은 synchronized와 같은 기능을 제공하는 클래스이다. synchronized와 다르게 ReentrantLock은 락을 획득한 스레드가 다시 락을 획득할 수 있는 재진입이 가능하다.

각 방법에 적합한 상황에 맞게 사용하도록 하자.


#### Reference

1. https://velog.io/@syleemk/Java-Concurrent-Programming-%EA%B0%80%EC%8B%9C%EC%84%B1%EA%B3%BC-%EC%9B%90%EC%9E%90%EC%84%B1#volatile%EB%A1%9C%EB%8F%84-%EC%B6%A9%EB%B6%84%ED%95%9C-%EA%B2%BD%EC%9A%B0
2. https://chat.openai.com/c/69ca47f7-394d-424f-a791-fd10cb8437a3
