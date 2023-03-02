# Kafka 실시간 데이터 처리: Exactly-Once 

실시간 처리에서 중요한 건 장애가 나더라도 정확히 한번 재개할 수 있느냐, 중복된 결과 없이이다.
- 중복된 결과나 부정확한 결과를 낼 수 있다고 함. 

exactly once 의 정의 자체는 이거임. 

> for each received record, its processed results will be reflected once, even under failures.

## Exactly-Once Semantics 

Publish-Subscriber 기반의 분산 시스템 기준으로 설명하자면 Producer 가 데이터를 여러번 보내도 Subscriber 는 이를 정확히 한 번만 처리하는 것.

## Failures that must be handled

- A broker can fail: 
  - broker 가 죽어도 괜찮다. 카프카에서는 N-1 브로커가 죽어도 여전히 작동할 수 있다.

- The producer-to-broker RPC can fail: 
  - producer 가 broker 에게 응답을 못받은 경우를 말한다. 
  - 메시지를 못받았다는게 반드시 메시지 처리를 실패한 건 아니다. broker 가 message 를 저장은 했지만 producer 에게 보냈을 때 실패했을 경우가 있다. 
  - 물론 메시지 처리에 실패했을 경우가 있을 수도 있다.
  - 이 경우 producer 는 실패의 종류를 알지 못하니 retry 를 할 수 있고, 중복된 메시지가 생성될 수 있다. 

- The client can fail:
  - 이미 죽은 클라이언트에서 보낸 메시지는 버리도록 처리되자 않도록 하기 위해서.

## ETC 

질문도 정리해보자. 
- Kafka 가 어떻게 정확히 한 번 처리할 수 있는지 를 알아야한다. (다양한 장애 상황속에서)

Kafka Streams 에 대해서도 공부를 해야할듯. 

count 를 올리는 어플리케이션을 설계한다고 해보자.

실패의 이유도 정리해보자. 그리고 이것들을 다 극복하는지 
- Network 이슈.
- broker 가 crash 나거나.

## 질문 

Kafka 와 Kafka Streams 의 차이
- Kafka Streams 는 lightweight version 의 카프카. 카프카 내부적으로 처리된다. external system engine or framework 없이.
  - input 과 output 은 모두 카프카 클러스터에 저장된다.
- Kafka Streams 에서 처리할 수 있는 옵션은 filtering, aggregation, joining, and transforming data in real-time
- Kafka 에서도 물론 실시간 처리를 할 수 있다.

Kafka 와 Kafka Streams 에서 exactly-once 처리하는 방법은?
- Kafka 는 transactional Producer 와 transactional consumer 를 통해서. 
- Kafka Streams 는 configuration option 을 통해서 가능하다.
  - 이렇게. `processing.guarantee=exactly_once`


## 질문 

<details>
<summary> 
부정확한 결과를 내는 경우는 언제지?
</summary>
- Failure and restart 할 때 non-deterministic operation 과 어플리케이션에 의한 외부 저장소의 상태 변경이 발생한 경우에 부정확한 결과를 내거나 중복된 결과가 발생할 수 있다고 한다.
- Failure 가 났을 때 external system 을 쓰고 있고 이를 롤백하는 기능이 없다면 incorrect 한 결과를 낼 수 있다고 한다.
  - count 의 state 를 DB 에 기록했고 처리헀다는 offset 을 날리려고 하는데 장애가 나서 안된 경우
    - offset 을 먼저 날리고 DB 에 기록한다면?
      - offset 성공 but DB 실패가 일어났다고 생각해보자.
        - offset 되돌려야한다. 누군가는 되돌려야하네.
        - DB 를 되돌리는게 맞지. offset 자체의 의미가 처리가 완료되었다는 뜻이니까.
  - 외부 시스템을 쓰고있고 롤백하는 시스템이 없다면 부정확한 결과를 낼 수 있네.
</details> 

<details> 
<summary>
Kafka 내부 시스템만 쓰고 있는 경우에는 exactly-once 를 보장할 수 있는가?
</summary>
`processing.guarantee=exactly_once` 이 옵션을 설정하면 되니까 가능함.

Kafka 에서 데이터를 처리하는 과정은 다음과 같은 루프를 도는 과정이다.

1) Topic A 에서 데이터를 읽어옴.

2) Processor 에서 데이터를 처리하고 상태를 저장함.

3) Output Message 를 Topic B 에 날림.

4) Topic B 에서 완료했다는 ACK 를 받음.

5) Topic A 에 데이터를 처리했다는 commit 을 날려줌.


여기서 4) 과 5) 에 문제가 생기면 중복 process 가 되거나 중복 Write 가 생긴다.

- 4) 에 응답이 늦게오거나 시스템이 데이터는 썼는데 응답을 못날린 상황이라면 retry 를 할거고 그러면 중복 데이터가 생길 수 있다.

- 5) 에서 commit 을 날려주기 전에 어플리케이션이 죽었다면 중복 처리하는 문제가 생겨서 상태가 두 번 업데이트 되는 문제가 생길 수 있다.


exactly-once 를 보장하려면 상태 업데이트와, ack 를 받는 과정과 commit 을 하는 과정이 한번에 되면 된다.

Kafka Streams 에서는 이를 다 Topic 에 메시지를 write 하는 과정으로 매핑해서 해결한다.

- 상태 업데이트는 Kafka Change log Topic 에 상태 메시지를 보낸는 걸로.
- commit 을 날리는 것은 Kafka Offset Topic 에 offset 메시지를 날리는 걸로.

이렇게 다 Topic 에 메시지를 날리는 과정으로 변환되었다면 Transaction API 를 써서 atomic 하게 처리하도록 한다.
- 하나가 실패하면 다 실패. 성공하면 다 성공

어떻게 이게 exactly once 를 보장할까? (= 간단한 문제로 변환되는 이유는 뭘까?)
- 실패했을 때 rollback 하는 과정이 있기 때문에.
- 하나가 실패했다면 성공한 쪽의 topic 에서 해당 메시지를 버리면 되는거니까.

실제로 해결이 되나?
- 4) 에서 ack 응답이 없다면? Topic 에 하나만 쌓이도록 Stream App 의 producer 에 `enable.idempotence` 이 옵션이 true 로 설정되어 있으면 된다.
- 에러가 생기면 해당 진행중인 트랜잭션을 버리면 된다. 그럼 해당 트랜잭션 ID 부터 다시 시작할 수 있는 것.
  - change log topic, offset topic, topic B 다 해당 트랜잭션 ID 의 offset 부터 재계산 하면 한번만 처리되도록 복구할 수 있다.

코드로 보면 이렇다.
```java
/** when commit() is called */

try {
  // send the offsets to commit as part of the txn
  producer.sendOffsets(“inputTopic”, offsets);

  // commit the txn
  producer.commitTxn();
  
} catch (KafkaException e) {
  producer.abortTxn();
}

/** in the normal processing loop */

try {
  recs = consumer.poll();
  
  for (Record rec <- recs) {

    // process ..

    producer.send(“outputTopic”, ..);         
    producer.send(“changelogTopic”, ..);   
  }
} catch (KafkaException e) { 
  producer.abortTxn();
}
```
- 그냥 예외가 나면 트랜잭션을 버림.
</details>

<details> 
<summary> Kafka Streams 에서 exactly-once 를 보장하고 싶다면 Kafka Connect 같은 것을 써야하나? </summary>

chatGPT 답변은 일단 그렇다고 함.

</details>

deterministic operation 이 뭔데?
- deterministic 은 결과가 항상 예측가능하고 일관성이 있어야한다는 것.

<details>
<summary>
모든 실패하는 케이스를 생각해보자.
</summary>
![](./kafka%20streams%20processing.png)

- 크게 나누면 Network 와 App, 좀비 인스턴스가 있다.
- App 은 처리전과 처리후 다운이 잇다.
- Network 는 응답이 느린 경우.
- 좀비 인스턴스는 분산 환경에서 생길 수 있는 문제다.
</details>

<details>
<summary> 
Kafka 에서 트랜잭션은 어떻게 동작하는 건가?
</summary>

Topic A 에서 메시지 a 를 읽어오고 처리한 후 Topic B 파티션 tb 에 메시지 b 를 보내는 작업을 한다고 생각했을 때

offset topic 에 topic A 의 메시지 a 에 대한 offset 을 기록하는 것과 파티션 tb 에 메시지 b 를 보내는 것이 atomic 하면 된다.

트랜잭션이 가지는 의미
- atomic multi-partition write
- zombie instance fencing
- reading transaction message

트랜잭션 처리되는 과정

![](./kafka%20transaction.png)

1) transaction register a transaction.id with coordinator
- 이 단계에서 pending 된 transaction 은 종료된다. transaction.id 당 하나의 transaction 만 적용하도록.
- 이 기준을 잡기 위해서 epoch 를 쓴다.

2) transaction 은 coordinator 에 의해서 transaction log 에 기록됨. (ongoing -> prepare -> committed)
- coodinator 가 트랜잭션을 관리한다. (읽고 쓰기 가능.)

3) producer 은 data 를 partition 으로 다 보냄.
- 이건 데이터 보내는 과정이다. 다만 유효한 producer 인지 검사한다.

4) producer 가 data 를 다 보낸후 commit 을 하면 two phase commit 을 통해서 완료됨.
- two phase commit 을 하는 이유는 데이터를 안쓸가능성이 있기 때문에.
- 파티션에 쓰여진 데이터는 커밋된거와 롤백된 거가 있다. 다음 consumer 가 가져가야 되는 데이터를 마킹하기 위해서 토픽 파티션에 마킹하는 작업인 transaction marker 가 추가됨.
</details>

<details>
<summary>
transactional.id 가 뭐지? producer 에게 붙는걸까?  
</summary>

메시지를 트랜잭션으로 처리하기 위해 식별성을 위해서 필요한 것.

트랜잭션에 참여하는 프로듀서와 컨슈머를 나타낼 수 있고 트랜잭션에 참여하는 메시지를 구별할 수도 있다.

transactional.id 는 증가하는게 아니라 재사용하는거네.
</details>

## 새로 안 사실 

external system 을 이용해서도 한번만 처리되게 만들 수 있다. 
- 처리된 여부를 같이 기록해놓는거지.

Kafka Streams 에서 performance 를 위해서 하나의 메시지씩 처리하는게 아님. 트랜잭션을 하나당 여러개의 메시지를 실행. commit_interval 주기로 배치로 함.
- 이 주기마다 새로운 트랜잭션이 만들어진다.
- 이 주기를 올리면 latency 가 그만큼 올려지는 대신 작은 트랜잭션이 생긴다. 

스트림즈 어플리케이션에서는 이전 계산이 부정확하면 다음 계산에도 영향을 준다.

Read-Process-Write 가 일반적인 처리 과정이다. 

카프카 버전 0.11 이전에서는 exactly-once 가 없었네. at-least-once 만 있었음.

스트림 어플리케이션의 시작 단계에서는 부정확한 처리가 일어날 수 있다는 것을 아는 것. 
- 하지만 이런 부정확한 처리를 견디지 못할 수도 있다.

Kafka 에서 트랜잭션이 나온 배경은 처리는 했지만 commit 을 못해서 중복 처리되는 문제와 좀비 인스턴스로 중복 output 이 나올 수 있는 문제를 해결하기 위해서 나왔다. 
- 원래는 네트워크 이슈로 producer 가 중복으로 메시지를 쓰는 경우도 있는데 이건 idempotent producer 설정을 쓰면 해결할 수 있어서 문제없다.  

kafka 에서 메시지가 consume 되었다는 걸 나타내느 증거로 offset topic 에다가 데이터를 쓴다. 

## References 

- https://www.confluent.io/blog/enabling-exactly-once-kafka-streams/

*** 

# Kafka Streams 는 어떻게 exactly-once 로 처리할 수 있을까? 

`processing.guarantee=exactly_once` 설정만 하면 Kafka Streams 는 정확히 한 번 처리하는게 가능해진다. 

- default value 는 `at_least_once` 이다. 


## What is Exactly-Once for Stream Processing?

> for each received record, its processed results will be reflected once, even under failures. 

## Exactly-Once: Why is it so Hard?

![](./images/kafka%20streams%20main%20loop.png)

1) Kafka Topic 으로 부터 message A 를 읽음. 

2) processing function 이 트리거 되서 기존 상태 S 를 S` 으로 업데이트 시킨다.

3) output message B1~Bn 까지 만들어서 output kafka topic 에다가 쓴다.

4) Kafka Broker 로 부터 send 요청에 대한 응답을 기다린다. 

5) 처리된 message A 에 commit 한다. 메시지 A 는 완전히 처리된 것. 

### Failure Scenario #1: Duplicate Writes

![](./images/duplicated%20write%201.png)

- output Topic 으로의 쓰기 요청 이후 네트워크 상황이 잠시 안좋아서 ack 를 못받았다고 가정해보자.
- retry 를 통해서 재전송할 것이고 이로인해서 topic 에 같은 결과물이 여러개가 생길 수 있다. 

![](./images/duplicated%20write%202.png)

### Failure Scenario #2: Duplicate Processing & Duplicate Writes 

![](./images/duplicate%20processing%201.png)

`4)` 까지의 작업이 완료되고 source Topic 에 commit 을 하려고 할 때 App 이 갑자기 죽는다면 어떻게 될까? commit 을 하지 못하고 죽었기 때문에 다시 해당 메시지를 처리한다. 

그래서 processing 도 두 번 일어나서 상태도 두 번 업데이트 되고 output topic 에다가도 두 번 쓰게되는 문제가 생긴다. 

## How Kafka Streams Guarantees Exactly-Once Processing

결국에 exactly-once 를 보장하려면 다음 3가지 step 에서 트랜잭션이 보장되면 된다. 

- Update the application state from S to S` 
- Write result messages  B1, … Bn to output Kafka topic(s) TB.
- Commit offset of the processed record A on the input Kafka topic TA

이 세가지 일을 다 성공시키거나 하나라도 실패하면 다 롤백 시키면 된다. 

어떻게 그게 가능할까? 위 세가지 작업은 다 토픽으로 데이터를 전송하는 과정으로 변환시켜 볼 수 있다. 

- `Update the application state from S to S` -> `Kafka changelog topic`
- `Write result messages  B1, … Bn to output Kafka topic(s) TB.` -> `Kafka sink topic`
- `Commit offset of the processed record A on the input Kafka topic TA` -> `Kafka offset topic`

Kafka changelog 에 기록하는 작업은 모든 state 들을 capture 해서 changelog 에 보관하는 작업이다. 새롭게 상태가 업데이트 될 때마다 그것들을 기록해놓는 것. 

이 세가지 topic partition 에다가 offset 을 쓰는 것은 Kafka 의 `Transactional API` 를 쓰면 가능하다.

Kafka Streams 에서 `processing.guarantee=exactly_once` 이 설정을 키면 내부적인 `embedded producer client` 가 transaction.id 를 이용해서 atomic 하게 모든 토픽 파티션에 쓰게 보장해준다.
- 만약 일시적인 network 장애가 일어났다면 `idempotent producer` 로 인해서 duplicate message 는 버려지게 된다.
- 만약 치명적인 error 가 발생했다면 exception 을 던지기 전에 카프카는 해당 트랜잭션을 abort 한다. 이로 인해서 같은 transaction.id 를 재시작할 수 있다.

![](./images/transaction%20abort.png)

````java
/** when commit() is called */

try {
  // send the offsets to commit as part of the txn
  producer.sendOffsets(“inputTopic”, offsets);

  // commit the txn
  producer.commitTxn();
  
} catch (KafkaException e) {
  producer.abortTxn();
}

/** in the normal processing loop */

try {
  recs = consumer.poll();
  
  for (Record rec <- recs) {

    // process ..

    producer.send(“outputTopic”, ..);         
    producer.send(“changelogTopic”, ..);   
  }
} catch (KafkaException e) {
  producer.abortTxn();
}
````

## How transactional work in Kafka  


예외 경우 생각 
- producer 가 데이터를 중간에 보내다가 죽는 경우 
- producer 가 ack 응답을 못받은 경우. (중복 패킷 버리면 됨)
- 브로커가 죽은 경우. 완료 처리가 안된거니까 다음 컨슈머에서 읽을 수도 없고 재시작하면 됨. 

two-phase-commit 을 하는 이유
- 컨슈머가 데이터를 읽을려고, 버려진 트랜잭션을 구별하려고

## idempotent producer
