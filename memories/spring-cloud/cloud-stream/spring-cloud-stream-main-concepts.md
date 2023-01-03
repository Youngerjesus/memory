# Spring Cloud Stream Main Concpets  

https://docs.spring.io/spring-cloud-stream/docs/3.2.4/reference/html/spring-cloud-stream.html#spring-cloud-stream-overview-binder-abstraction

***

## Main Concepts 

- message-driven microservice 를 만들기 쉽도록 해준다. 

### Spring Cloud Stream's application Model 

- spring cloud stream application 은 middleware-neutral core 를 이룬다.
  - neutral 이 중립적인 이란 뜻이다.
  - 특정한 broker 와만 연결되지 않는 다는 뜻이겠지. 
- binding 을 통해서 external service 와 communication 을 할 수 있다. 
  - 정확하겐 external broker 와 input/output argument 를 통해서 통신.  
  - 어떤 broker 와 연결될 수 있는지는 구현 부분에 의존한다. 

### Fat Jar 

- production 에 내보낼려면 JAR 파일을 만들어야 한다고함. 
  - maven 과 gradle 을 이용해서.


### The Binder Abstraction 

- Kafka 와 RabbitMQ 에 대한 Binder 구현을 제공해줌. 
- 또한 Test Binder 까지. 
- Binder Abstraction 을 통해서 너만의 Binder 를 구현할 수 있다.
  - 스프링 특유의 확장 포인트 중 하나. 
  - 해당 문서에선 그것도 있음. 
- binder 를 자동으로 detect 하고 사용한다. binder 를 설정하는 부분도 configuration 파일이라서 동일한 application 코드로 다른 binder 를 사용하는게 가능함.

### Persistent Publish-Subscribe Support 

- publish-subscribe model 을 지원한다. 뭐 이런거.

### Consumer Group 

- consumer 끼리 경쟁해야 하는 상황이 올 수 있다. 스케일링을 지원하려면. 
- 그래서 consumer Group 을 지원하며 이건 Kafka 의 Consumer Group 과 유사하다.
  - 각각의 컨슈머는 `spring.cloud.stream.bindings.<bindingName>.group` 에 속한다.
  - 여러 그룹에 속하도록 할 수도 있네. 
- 그룹을 명시하지 않으면 anonymous 한 독립적인 그룹을 할당해준다. 

### Consumer Type 

- 컨슈머는 두 가지 타입이 있다. 
  - message-driven (Asynchronous)
  - Polled (Synchronous)
  - 2.0 버전 이후로는 asynchronous 방법만 

### Durability 

- consumer group subscription 은 durable 하다. 
- 그룹 내의 모든 어플리케이션은 종료될 때 동안 메시지는 보내진다. 이런 뜻.
- 안보내지고 이런게 없다. 

### Partition Support 

- 파티셔닝을 지원한다. 
