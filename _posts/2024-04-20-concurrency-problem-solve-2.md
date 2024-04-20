---
layout: post
title: 동시성 제어  - 데이터베이스
date: 2024-04-18 10:43 +0900
img_path: /assets/images/
category: [ concurrency ]
tags: [ Concurrency, Database ]
---

이전 글 [동시성 제어 - 프로그래밍 언어](https://spandios.github.io/posts/concurrency-problem-solve-1/)에서는 자바에서 동시성 문제를 해결하는 방법을 알아보았다.

이번에는 데이터베이스 수준에서 동시성 문제를 해결하는 방법을 알아보자.

데이터베이스 수준에서는 잠금과 트랜잭션 격리 수준을 이용하여 동시성 문제를 해결한다.

먼저 잠금에 대해 알아보자. 데이터베이스에서도 잠금을 이용해 동시성 문제를 해결한다. 데이터베이스에서 데이터를 읽거나 쓸 때 다른 트랜잭션이 접근하지 못하도록 한다.

# 비관적 잠금 (Pessimistic Locking)

비관적이라는 용어 그대로 동시성 문제(경쟁 조건)가 자주 발생할 것이라고 가정해 미리 잠금을 걸어두는 방식이다.

실제 데이터를 처리하기전에 레코드를 조회 후 그 레코드에 잠금을 걸어 다른 트랜잭션이 이 레코드를 변경하지 못하도록 한다.

비관적 잠금은 DB 락을 사용하기 때문에 개발 편의성이 있고 실패 시 따로 복구 작업이 필요하지 않는다.

하지만 행 자체에 락을 걸기 때문에 성능이 떨어지고 잘못된 설계로 인해 데드락이 발생할 수 있다.

이제 비관적 잠금을 사용하는 방법을 알아보자.

## SELECT ... FOR UPDATE

사용법은 간단하다 `SELECT ... FOR UPDATE` 구문을 사용하면 조회한 레코드에 잠금을 걸 수 있다.


spring data jpa를 사용한다면 `@Lock(LockModeType.PESSIMISTIC_WRITE)`를 사용하면 된다. 

저번에 살펴본 자바에서 동시성과 관련된 예시를 데이터베이스 수준에서 해결해보자.

Repository에 @Lock 어노테이션을 사용하여 비관적 잠금을 사용하고 메서드에 @Transactional 어노테이션을 추가한다.

```kotlin
   // Repository 
interface StockRepository : JpaRepository<Stock, Long> {
  @Lock(LockModeType.PESSIMISTIC_WRITE)
  @Query("select s from Stock s where s.productId=:id")
  fun findByProductIdWithPessimisticLock(id: Long?): Stock?
}

// Service 
@Transactional
fun decreaseStock(
  productId: Long,
  quantity: Long,
) {
  val stock = stockRepository.findByProductIdWithPessimisticLock(productId)
    ?: throw IllegalArgumentException("Stock not found")
  stock.decrease(quantity)
  stockRepository.save(stock)
}
```

테스트 코드를 작성해보자. 조회 후 잠금을 걸어 다른 트랜잭션이 변경하지 못하하기 때문에 동시성 문제가 발생하지 않는다.
따라서 동시에 100개의 요청을 보내도 재고가 예상대로 0이 된다.


```kotlin
 @Test
fun 동시에_100개의_요청하기() {
  val threadCount = 100
  val executorService = Executors.newFixedThreadPool(threadCount)
  val countDownLatch = CountDownLatch(threadCount)

  for (i in 1..threadCount) {
    executorService.submit {
      try {
        stockService.decreaseStock(1, 1L)
      } finally {
        countDownLatch.countDown()
      }

    }
  }

  countDownLatch.await()

  val stock = stockRepository.findById(1).orElseThrow()
  assertEquals(0L, stock.quantity)
}


```

## 데드락 발생 예시

이쯤에서 데드락에 대해 간단히 알아보자. 데드락이란 두 개 이상의 트랜잭션이 각자 잠금을 보유하면서 잠금된 상대방의 자원을 계속해서 기다리는 상황을 말한다.

데드락을 방지하기 위해선 잠금을 걸 순서를 한 흐름으로 통일 시켜 잠금에 대한 요청이 꼬이지 않게 하는 법 그리고 락을 걸 때 타임아웃까지 설정해 일정 시간이 지나면 락을 해제하는 방법이 있다.

데드락이 발생하는 예시를 살펴보자. 두 개의 계좌가 있고 두 계좌 끼리 동시에 이체를 시도하는 상황이다.

### Entity 
```kotlin
@Entity
class Account(
        @Id
        @GeneratedValue(strategy = GenerationType.IDENTITY)
        val id: Long = 0,
        val owner: String? = null,
        val balance: Double = 0.0,
)
```

### Service

```kotlin
@Service
class AccountService(private val accountRepository: AccountRepository) {

  @Transactional
  fun transferMoney(fromAccountId: Long, toAccountId: Long, amount: Double) {
    val fromAccount: Account = accountRepository.findById(fromAccountId).orElseThrow()
    val toAccount: Account = accountRepository.findById(toAccountId).orElseThrow()

    fromAccount.balance -= amount
    accountRepository.save(fromAccount)

    toAccount.balance += amount
    accountRepository.save(toAccount)
  }
}
```
두 계좌를 조회 후 잠금을 걸고 fromAccount의 잔액을 감소시킨 후 toAccount의 잔액을 증가시킨다. 

두 개의 스레드가 동시에 이체를 시도하면 데드락이 발생한다. 정말 데드락이 발생하는지 테스트 코드를 작성해보자.

```kotlin
 @Test
fun 데드락_발생() {
  var isDeadLock = false
  val t1 = Thread {
    try {

      accountService.transferMoney(1L, 2L, 100.0)
    } catch (e: CannotAcquireLockException) {
      isDeadLock = true
    }
  }
  val t2 = Thread {
    try {
      accountService.transferMoney(2L, 1L, 100.0)
    } catch (e: CannotAcquireLockException) {
      isDeadLock = true
    }
  }

  t1.start()
  t2.start()

  t1.join()
  t2.join()

  assert(isDeadLock)

}

```

왜 데드락이 발생한걸까? 잠금 순서가 고정적으로 보내는 계좌 부터 받는 계좌에 잠금을 걸고 있기 때문에 만약 동시에 서로 이체를 하게 된다면 잠금을 기다리는 상황이 발생한다.

![]({{site.url}}/assets/images/deadlock.png)

이제 데드락을 해결하는 방법을 알아보자.

현재 account id는 Long 타입으로 생성될때마다 증가한다. 따라서 id가 작은 것이나 큰 것 둘 중 하나를 선택해 잠금을 걸도록 수정하면 데드락이 발생하지 않는다.


```kotlin
 @Transactional
    fun transferMoneyWithOrder(fromAccountId: Long, toAccountId: Long, amount: Double) {

        val minId = min(fromAccountId, toAccountId)
        val maxId = max(fromAccountId, toAccountId)
  
        // 계좌 ID 값을 비교하고 작은 값부터 큰 값 순서로 락을 건다.
        val firstAccount = accountRepository.findById(minId).orElseThrow()
        val secondAccount = accountRepository.findById(maxId).orElseThrow()

        // minId가 fromAccountId인 경우
        if (fromAccountId == minId) {
            firstAccount.balance -= amount
            secondAccount.balance += amount
        } else {
        // minId가 toAccountId인 경우
            secondAccount.balance -= amount
            firstAccount.balance += amount
        }

        // 변경 사항 저장
        accountRepository.save(firstAccount)
        accountRepository.save(secondAccount)
    }
```

테스트 코드를 작성해보자. 이제 데드락이 발생하지 않는다.
```kotlin
 @Test
    fun 데드락_미발생() {
        var isDeadLock = false
        val t1 = Thread {
            try {
                accountService.transferMoneyWithOrder(1L, 2L, 100.0)
            } catch (e: CannotAcquireLockException) {
                isDeadLock = true
            }
        }
        val t2 = Thread {
            try {
                accountService.transferMoneyWithOrder(2L, 1L, 200.0)
            } catch (e: CannotAcquireLockException) {
                isDeadLock = true
            }
        }

        t1.start()
        t2.start()

        t1.join()
        t2.join()

        assertThat(isDeadLock).isFalse()
    }

```

# 낙관적 잠금

낙관적 잠금은 비관적 잠금과 다르게 동시성 문제가 자주 발생하지 않을 것이라고 가정하기 때문에 미리 잠금을 걸지 않는다. 

대신 데이터를 변경하기 직전에 최신 데이터를 조회 후 기존에 알고있던 값이라면 즉, 변경이 되지 않았다면 데이터를 변경한다. 이는 전에 알아보았던 CAS(Compare And Swap)와 비슷하다.

낙관적 잠금은 비관적 잠금에 비해 락을 걸지 않고 애플리케이션단에서 처리하기 때문에 성능이 더 좋다. 하지만 데이터를 변경하기 전에 조회를 해야하기 때문에 데이터가 자주 변경되는 경우에는 비관적 잠금보다 성능이 떨어질 수 있다.

비관적 잠금과 다르게 직접 복구 작업을 처리해주어야 하는 점이 있다.

이제 낙관적 잠금을 사용하는 방법을 알아보자.

## JPA 버전 관리

JPA에서는 버전 관리를 통해 낙관적 잠금을 사용할 수 있다. 버전 관리는 엔티티의 변경이 일어날 때마다 버전을 증가시키는 방법이다.

JPA에서는 `@Version` 어노테이션을 사용해 쉽게 버전 관리를 할 수 있다.

낙관적 잠금은 Database락을 사용하지 않기 때문에 락 순서에 대해 신경쓸 필요가 없다. 

만약 중간에 다른 트랜잭션이 데이터를 수정하여 버전이 변경되었다면, 충돌이 발생했다고 판단하고 OptimisticLockException을 발생시키는데 이 예외를 처리해줘야한다.

재시도 예외 처리를 그냥 무한 반복으로 처리할 수도 있지만 이는 좋은 방법이 아니다. 조금 더 나은 방법은 spring-retry 라이브러리를 사용해 재시도 횟수를 제한하는 방법이 있다.

다음 두가지 의존성을 추가해주자 
```groovy
implementation 'org.springframework.retry:spring-retry'
implementation 'org.springframework.boot:spring-boot-starter-aop'
```

이 후 `@EnableRetry` 애노테이션을 스프링 부트 애플리케이션 클래스에 추가해주자. 

```kotlin
@SpringBootApplication
@EnableRetry
class SpringConcurrencyApplication
```

낙관적 잠금으로 데이터를 조회하는 메서드를 작성한다

```kotlin
    @Lock(LockModeType.OPTIMISTIC)
    @Query("SELECT a FROM Account a WHERE a.id = :id")
    fun findByIdWithOptimistic(id: Long): Optional<Account>
```


```kotlin
 @Transactional
@Retryable(
  value = [ObjectOptimisticLockingFailureException::class],
  maxAttempts = 3,
  backoff = Backoff(delay = 1000, multiplier = 2.0)
)
fun transferMoneyWithOptimisticLock(fromAccountId: Long, toAccountId: Long, amount: Double) {
  try {
    println("Transfer money $amount from $fromAccountId to $toAccountId")
    val fromAccount = accountRepository.findByIdWithOptimistic(fromAccountId).orElseThrow()
    val toAccount = accountRepository.findByIdWithOptimistic(toAccountId).orElseThrow()

    fromAccount.balance -= amount
    toAccount.balance += amount

    accountRepository.save(fromAccount)
    accountRepository.save(toAccount)
  } catch (e: Exception) {
    throw ObjectOptimisticLockingFailureException("낙관적 락 실패 from account ${fromAccountId}, to account : $toAccountId", e)
  }
}
```

```kotlin
@Retryable(
  value = [ObjectOptimisticLockingFailureException::class],
  maxAttempts = 3,
  backoff = Backoff(delay = 1000, multiplier = 2.0)
)
```

`@Retryable` 어노테이션을 사용해 재시도 횟수는 물론 재시도 간격을 설정할 수 있다.
메서드 실행 중 ObjectOptimisticLockingFailureException이 발생하면 최대 3번까지 재시도하고 1초 간격으로 재시도하며 한 번 재시도할 때마다 시간이 2배씩 간격이 늘어난다.


# 트랜잭션 격리(Transaction Isolation)

트랜잭션 격리란 동시에 여러 트랜잭션이 실행되어 한 공유 데이터에 접근하는 상황일 때 각 트랜잭션 간의 고립되는 수준을 말한다.

즉, 트랜잭션이 실행하는 동안 다른 트랜잭션이 데이터 변경하는 것을 어떤 수준까지 허용할 것인지를 결정하는 것이다. 

트랜잭션 격리 수준은 데이터베이스마다 지원하는 수준이 다르다.

- READ UNCOMMITTED : 커밋되지 않은 데이터를 읽을 수 있다.
- READ COMMITTED : 커밋된 데이터만 읽을 수 있다.
- REPEATABLE READ : 트랜잭션 내에서 조회한 데이터는 다른 트랜잭션이 변경해도 같은 결과를 보여준다.
- SERIALIZABLE : 트랜잭션 간의 격리 수준이 가장 높다. 트랜잭션 간의 격리를 완벽하게 보장한다.

SERIALIZABLE 격리 수준은 동시성을 제어하기 위해 가장 높은 수준의 격리를 제공한다. 데이터 일관성이 매우 중요한 경우에 사용한다. 하지만 성능이 가장 떨어지기 때문에 주의 해야한다.

이 수준에서는 여러 트랜잭션이 동시에 실행되더라도 마치 순차적으로 실행되는 것처럼 보인다. 한 트랜잭션만 레코드에 접근할 수 있기 때문에 다른 트랜잭션이 접근하지 못한다. 

하지만 그렇다하더라도 잠금 순서에 따라 데드락이 발생할 수 있기 때문에 주의해야한다.

```kotlin
@Transactional(isolation = Isolation.SERIALIZABLE) 
```





