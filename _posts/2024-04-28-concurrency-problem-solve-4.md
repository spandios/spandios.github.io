---
layout: post
title: 동시성 제어 - 메시징 큐
date: 2024-04-28 10:43 +0900
img_path: /assets/images/
category: [ concurrency ]
tags: [ Concurrency, Message Queue, RabbitMQ]
---

이번에는 메시징 큐를 통해 동시성 문제를 해결하는 방법을 알아보자.

# 메시징 큐 

메시징 큐(Message Queue)는 분산 시스템에서 다양한 컴퓨넌트 간에 비동기적으로 메시지를 전송하고 처리하기 위한 중간 매개체이다.

메시징 큐는 주로 다음과 같은 목적으로 사용된다.

- 비동기 통신: 메시지 큐를 사용하면 발신자(Producer)와 수신자(Consumer) 간의 통신이 비동기적으로 이루어진다. 메시지를 생성한 발신자는 메시지 전송만 하면 되기 때문이다.

- 시스템 간 결합: 메시징 큐를 사용하면 다른 시스템 간에 데이터를 교환할 수 있습니다. 시스템 간의 결합을 최소화하고 유연성을 높일 수 있습니다.

- 탄력성 및 확장성: 메시징 큐는 대량의 메시지를 안정적으로 처리하고, 시스템 부하에 따라 탄력적으로 확장할 수 있습니다. 이를 통해 시스템의 성능과 신뢰성을 향상시킬 수 있습니다.

주요한 메시징 큐 시스템으로는 Apache Kafka, RabbitMQ, Amazon SQS 등이 있다.


# RabbitMQ로 동시성 문제 해결하기

RabbitMQ는 AMQP(Advanced Message Queuing Protocol)를 구현한 오픈소스 메시징 시스템이다. 

동시성 문제는 여러 스레드가 동시에 공유 자원에 접근할 때 발생한다. 

메시징 큐를 사용하면, 많은 공유 자원에 대한 접근 수정 요청을 순차적으로 처리할 수 있기 때문에 동시성 문제를 해결할 수 있다.

간단히 살펴보기 위해, 최대한 간단히 구현해보자.

먼저 docker를 통해 RabbitMQ를 실행한다. 나는 docker-compose를 사용했다.

```bash

version: '3'
services:
  rabbitmq:
    image: rabbitmq:management
    container_name: rabbitmq
    ports:
      - "5672:5672"
      - "15672:15672"
    volumes:
      - rabbitmq_data:/var/lib/rabbitmq
    environment:
      RABBITMQ_DEFAULT_USER: user
      RABBITMQ_DEFAULT_PASS: password

volumes:
  rabbitmq_data:


```


이 후 http://localhost:15672/에서 직접 큐와 exchange를 만들어도 되지만 RabbitMQ 설정을 통해서도 가능하다. 

```kotlin

@Configuration
class RabbitConfig {
    @Bean
    fun queue(): Queue {
        return Queue("testQueue",true)
    }

    @Bean
    fun exchange(): TopicExchange {
        return TopicExchange("testExchange")
    }

    @Bean
    fun binding(queue: Queue, exchange: TopicExchange): Binding {
        return BindingBuilder.bind(queue).to(exchange).with("routing.key")
    }
}

```

이 후 메시지를 보내는 Producer와 메시지를 받는 Consumer를 구현하면 된다.

먼저 Producer는 다음과 같이 queue의 이름을 지정해주고 메시지를 보낸다. 
```kotlin
@Service
class MessageSender(private val rabbitTemplate: RabbitTemplate) {

    fun send(productId: String) {
        rabbitTemplate.convertAndSend("testQueue", productId)
    }
}
```

Consumer는 다음과 같이 @RabbitListener를 사용하여 메시지를 받는다. queue의 이름을 지정해주면 해당 queue에 있는 메시지를 받아온다.

이 때 concurrency를 1로 설정해 주면 한번에 하나의 메시지만 처리하게 된다.
```kotlin
@Component
class MessageReceiver(private val stockService: StockService) {
    @RabbitListener(queues = ["testQueue"], concurrency = "1")
    fun receive(productId: String) {
        stockService.decreaseStock(productId.toLong(), 1)
    }
}

```

실제로 동시성 문제가 해결되었는지 확인하기 위해 기존 테스트 코드를 messageSender를 사용하도록 변경해보자. 테스트가 잘 통과되는 것을 확인할 수 있다.

```kotlin

 @Test
    fun 동시에_100개의_요청하기() {
        val threadCount = 100
        val executorService = Executors.newFixedThreadPool(threadCount)
        val countDownLatch = CountDownLatch(threadCount)

        for (i in 1..threadCount) {
            executorService.submit {
                try {
                    messageSender.send("1")
                }finally {
                    countDownLatch.countDown()
                }

            }
        }

        countDownLatch.await()

        Thread.sleep(3000)

        val stock = stockRepository.findById(1).orElseThrow()
        assertEquals(0L, stock.quantity)
    }


```





