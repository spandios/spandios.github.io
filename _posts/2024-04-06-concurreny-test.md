---
layout: post
title: 동시성 테스트 도구들
date: 2024-04-06 10:43 +0900
img_path: /assets/images/
category: [concurrency]
tags: [ Concurrency, Test ]
---

# Executor Service
- ExecutorService는 동시성 테스트를 위해 작업을 비동기로 수행해야할 때 주로 사용되는 인터페이스이다. 
- 이미 풀링된 여러 스레드 중 하나를 사용해 작업을 수행한다.

## 쓰레드 풀
쓰레드 풀은 미리 생성된 스레드의 집합으로 사용자가 쉽게 여러 쓰레드를 관리할 수 있다.
DB ConnectionPool과 마찬가지로 많은 수의 비동기 작업을 실행할 때 호출 오버헤드가 감소하여 성능이 향상되고, 실행되는 쓰레드 리소스를 관리할 수 있다.  

쓰레드 풀 자체를 설정할 수 있는 ThreadPoolExecutor 클래스를 사용할 수도 있지만, 더 편한 Executors Factory Method가 제공하는 다음 3가지 유형을 사용하는 것이 권장된다.
- FixedThreadPool
  - 고정된 수의 쓰레드를 가지고 있는 풀이다.
  - 모든 쓰레드가 활성 상태에서 추가 작업이 제출되면 쓰레드를 사용할 수 있을 때까지 대기한다.
- CachedThreadPool
  - FixedThreadPool과 다르게 필요에 따라 새 스레드를 생성한다.
  - 쓰레드가 사용되지 않으면 자동으로 제거되기 때문에 작업이 짧으면서 많은 수의 스레드를 사용해야할 때 유용하다.
- ScheduledThreadPool
  - 작업을 딜레이를 주거나 주기적으로 실행할 수 있도록 스케줄링 할 수 있다.

## ThreadPoolExecutor
ThreadPoolExecutor는 ExecutorService 인터페이스를 실제 구현한 클래스로 쓰레드 풀을 생성하고 관리하는 클래스이다. 

다음은 Executors 클래스를 사용하여 ThreadPoolExecutor를 생성하는 예제이다.

```java

public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue) {
        this(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue,
             Executors.defaultThreadFactory(), defaultHandler)
```

```kotlin
// 고정 쓰레드풀은 아래 생성자 호출을 통해 알 수 있듯이 풀에 돌아가는 기본 쓰레드 수와 최대 쓰레드 수가 같은 것을 알 수 있다.     
fun newFixedThreadPool(nThreads: Int): ExecutorService {
  return ThreadPoolExecutor(nThreads, nThreads,
    0L, TimeUnit.MILLISECONDS,
    LinkedBlockingQueue())
}

// CachedThreadPool은 고정과 다르게 고정된 최대 쓰레드 수도 없고 60초 동안의 유휴 시간이 지나면 쓰레드를 제거한다.
fun newCachedThreadPool(): ExecutorService {
  return ThreadPoolExecutor(0, Integer.MAX_VALUE,
    60L, TimeUnit.SECONDS,
    SynchronousQueue())
}

// ScheduledThreadPool도 고정된 쓰레드 수를 가지고 있지만, 쓰레드를 사용하지 않을 때 쓰레드를 제거하지 않는다.
fun newScheduledThreadPool(corePoolSize: Int): ScheduledExecutorService {
  return ThreadPoolExecutor(corePoolSize, Integer.MAX_VALUE,
    0, TimeUnit.NANOSECONDS,
    DelayedWorkQueue())
}


```

## 기본적인 사용법
- newFixedThreadPool() 메서드를 사용하여 고정된 쓰레드 수를 가지는 쓰레드 풀을 생성하고, submit() 메서드를 사용하여 쓰레드에 작업을 제출한다.

```kotlin
val threadCount = 100
val executorService = Executors.newFixedThreadPool(threadCount) // 고정된 쓰레드 수를 지정한다.

for (i in 1..threadCount) {
  executorService.submit { // 작업을 제출한다.
    stockService.decreaseStock(1, 1L)
  }
}


```

ScheduledThreadPool을 사용하여 3초 후에 작업을 실행하는 예제
```kotlin
 val executor = Executors.newScheduledThreadPool(1)

// 3초 후에 작업을 실행
executor.schedule({
  println("Hello from thread " + Thread.currentThread().name)
}, 3, TimeUnit.SECONDS)

// 스케줄러를 종료
executor.shutdown()

```



## BlockingQueue
위의 생성자에서 봤듯이 ThreadPoolExecutor는 BlockingQueue를 인자로 받는다. 위의 3가지 유형의 쓰레드 풀은 각기 다른 BlockingQueue를 사용하는 것을 볼 수 있다.


#### BlockingQueue란
BlockingQueue는 Java의 java.util.concurrent 패키지에 있는 인터페이스로, 스레드 안전한 큐를 제공한다.
BlockingQueue가 기존 큐와 다른 특징은 다음과 같다.
  - 원소 추가 : 만약 큐가 가득찬 경우라면 빈 공간이 생길 때까지 쓰레드를 블록한다. 
  - 원소 제거 : 만약 큐가 비어있는 경우라면 요소가 추가될 때까지 쓰레드를 블록한다.

이렇게 BlockingQueue는 큐가 가득차거나 비어있을 때 쓰레드를 블록시켜 멀티쓰레드 환경에서 안전하게 요소를 추가하거나 제거할 수 있다.


### BlockingQueue 구현체
- LinkedBlockingQueue : BlockingQueue의 구현체 중 하나로, 연결 리스트로 구현되어 있다. 큐의 크기를 지정하지 않으면 Integer.MAX_VALUE로 지정된다.
- SynchronousQueue: 큐의 크기가 0인 큐로, 요소가 추가되면 그 즉시 요소를 소비할 스레드가 나타날 때까지 스레드가 블록된다. 
  - CachedThreadPool에서는 작업이 할당되면 그 즉시 새로운 쓰레드를 생성하던가 기존 쓰레드를 사용해야하기 때문에 SynchronousQueue를 사용한다.
  - 만약 SynchronousQueue를 사용하지 않는다면, 새 작업 할당 시 큐에 빈 공간이 생길 때까지 쓰레드가 블록되어 쓰레드가 생성되지 않을 수 있다.
- DelayedWorkQueue : Delayed 인터페이스를 추가 구현한 큐이다.



### BLockingQueue 테스트
```kotlin
    // BlockingQueue는 공간이 가득찬 상태에서 put을 호출하면 스레드가 블록되고, 공간이 비어있는 상태에서 take를 호출하면 스레드가 블록된다.
    val queueSize = 5
    val blockingQueue: BlockingQueue<Int> = LinkedBlockingQueue(queueSize)

    val producer = Thread {
        for (i in 1..10) {
            try {
                blockingQueue.put(i)
                println("Produced: $i")
            } catch (e: InterruptedException) {
                e.printStackTrace()
            }
        }
    }

    val consumer = Thread {
        try {
            while (true) {
                Thread.sleep(2000)
                val value = blockingQueue.take()
                println("Consumed: $value")
            }
        } catch (e: InterruptedException) {
            e.printStackTrace()
        }
    }

    producer.start()
    consumer.start()

    // Wait for both threads to finish
    producer.join()
    consumer.join()


결과 :
Produced: 1
Produced: 2
Produced: 3
Produced: 4
Produced: 5
Produced: 6
Consumed: 1
Consumed: 2
Produced: 7
Consumed: 3
Produced: 8
Consumed: 4
Produced: 9
Consumed: 5
Produced: 10
Consumed: 6
Consumed: 7
Consumed: 8
Consumed: 9
Consumed: 10

5까지는 바로바로 큐에 삽입 되지만 큐가 꽉 찼을 때는 블록 된다. 
이후 take()메서드가 호출 되어 큐가 빈 공간이 생기면 바로 대기중인 6을 추가하는 쓰레드가 실행되어 추가 되는 것을 볼 수 있다. 
```
