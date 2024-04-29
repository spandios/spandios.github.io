---
layout: post
title: 동시성 제어 - 레디스
date: 2024-04-28 10:43 +0900
img_path: /assets/images/
category: [ concurrency ]
tags: [ Concurrency, Redis ]
---

이번에는 레디스를 통해 동시성 문제를 해결하는 방법을 알아보자.

레디스는 메모리 기반의 키-값 데이터베이스로 기존 RDB보다 훨씬 빠른 속도를 가지고 있다.

동시성 문제 해결 시 레디스는 특히 분산락을 구현하는데 매우 유용하다.

## 분산 락

단일 서버에서 돌아가는 환경이라면 synchronized 키워드나 ReentrantLock을 사용하여 동시성 문제를 충분히 해결할 수 있다.

하지만 분산 환경이라면 여러 서버에서 스레드가 동시에 실행되어 아무리 synchronized 키워드나 ReentrantLock을 사용해도 동시성 문제가 여전히 발생할 수 있다.

이 때 분산락이 필요한데, 락을 여러 서버가 관리하는게 아니라 레디스와 같은 외부 서버에서 락을 관리하기 때문에 분산 환경에서도 동시성 문제를 해결할 수 있다.

레디스는 메모리 기반의 데이터베이스이기 때문에 빠른 속도를 가지고 있어 분산락을 구현하는데 적합하다.

이제 레디스를 통해 분산락을 구현하는 방법을 알아보자.

# 스핀락

락이 있다면 잠금을 획득할 때까지 계속해서 잠금을 요청하는 방식이다.

가장 쉽게 구현할 수 있는 장점이 있다.

하지만 잠금을 획득할 때까지 계속해서 요청하기 때문에 만약 잠금을 가져오는데 오래 걸리면 걸릴수록 레디스 서버에 부하가 있을 수 있고 해당 스레드는 계속 대기 상태이기 때문에 자원 비효율적이다.

때문에 스핀락은 잠금을 획득하는데 시간이 오래 걸리지 않을 때 사용하는 것이 좋을 것 같다.

저번에 살펴본 자바에서 동시성과 관련된 예시를 레디스를 통해 해결해보자.

먼저 strter redis 의존성을 추가한다.
```groovy
implementation 'org.springframework.boot:spring-boot-starter-data-redis'
```

그리고 RedisRepository를 생성한다.
```kotlin

@Repository
class RedisRepository(val redisTemplate: RedisTemplate<String, String>) {

  fun lock(key: Long, duration: Duration = Duration.ofMillis(3_000)): Boolean? {
    return redisTemplate
      .opsForValue()
      .setIfAbsent(key.toString(), "l", duration)
  }

  fun unlock(key: Long): Boolean {
    return redisTemplate.delete(key.toString())
  }
}

```

잠금은 Lock의 키를 파라미터로 받고 레디스에 저장하는 것으로 구현한다. 저장은 setIfAbsent 메서드를 사용하여 해당 키가 없을 때만 저장한다.

해당 키가 이미 존재한다면 이미 다른 곳에서 잠금을 획득한 것이기 때문에 false를 반환한다. 

unlock 메서드는 해당 키를 삭제함으로써 잠금 해제를 구현한다.

지난 예시에서 사용한 decreaseStock 메서드를 레디스를 통해 구현해보자.



```kotlin
@Transactional()
fun decreaseStock(
  productId: Long,
  quantity: Long,
) {
  try {
    val stock = stockRepository.findByProductIdWithPessimisticLock(productId)
    if (stock == null) {
      throw IllegalArgumentException("Stock not found")
    }

    while (redisRepository.lock(productId) != true) {
      Thread.sleep(300)
    }

    stock.decrease(quantity)
    stockRepository.save(stock)
  } finally {
    redisRepository.unlock(productId)
  }
}

```


```kotlin
while (redisRepository.lock(productId) != true) {
      Thread.sleep(300)
    }
```

해당 코드는 레디스를 통해 잠금을 획득할 때까지 계속해서 요청하는 코드이다. 만약 잠금을 획득하지 못하면 300ms 대기 후 다시 요청한다.

try finally 구문을 사용하여 잠금을 획득하고 작업을 수행한 후 반드시 잠금을 해제하도록 구현한다.

결과는 동시에 100개의 요청을 보내도 재고가 예상대로 0이 된다.

# 메시지 브로커 (Pub/Sub) 

위에서 살펴봤듯이 스핀락의 단점은 잠금을 언제 획득할 수 있을지 모르기 때문에 계속 요청하기 때문에 비효율적으로 자원을 사용한다는 점이다.

이를 해결하기 위해 Redis의 기능 중 하나인 메시지 브로커를 사용할 수 있다. 

메시지 브로커는 발행/구독(Pub/Sub) 모델을 사용하여 메시지를 발행하고 구독하는 방식으로 동작한다. 

만약 잠금이 풀렸다면 메시지를 발행하고 해당 메시지를 구독하는 쪽(잠금 획득 대기하는 쪽)에서 바로 잠금을 획득하도록 구현할 수 있다.

이러한 일련의 과정을 이미 구현한 Redisson 라이브러리를 사용하면 더욱 쉽게 구현할 수 있다.

```groovy
implementation("org.redisson:redisson-spring-boot-starter:3.21.1")
```


```kotlin

@Service
class StockService(private val stockRepository: StockRepository, private val redissonClient: RedissonClient) {


  fun findById(id: Long) = stockRepository.findById(id)

  @Transactional()
  fun decreaseStock(
    productId: Long,
    quantity: Long,
  ) {
    val lock: RLock = redissonClient.getLock(productId.toString())
    try {
      val stock = stockRepository.findByProductIdWithPessimisticLock(productId)
      if (stock == null) {
        throw IllegalArgumentException("Stock not found")
      }

      lock.tryLock(10, 2, TimeUnit.SECONDS)
      stock.decrease(quantity)
      stockRepository.save(stock)
    } finally {
      lock.unlock()
    }
  }
}


```

RedissonClient를 DI 받고 getLock 메서드를 통해 RLock을 생성한다.

tryLock 메서드를 통해 잠금을 획득하고 작업을 수행한 후 반드시 unlock 메서드를 호출하여 잠금을 해제한다.

tryLock의 파라미터인자로 잠금을 획득할 때까지 대기하는 시간과 잠금을 획득한 후 유지할 시간을 설정할 수 있다.

결과로는 마찬가지로 동시에 100개의 요청을 보내도 재고가 예상대로 0이 된다.












