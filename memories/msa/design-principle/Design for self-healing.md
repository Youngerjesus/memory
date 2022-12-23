# Design for self-healing 

https://learn.microsoft.com/en-us/azure/architecture/guide/design-principles/self-healing

***

- In a distributed system, failures happen. Design your application to be self healing when failures occur.
- 실패를 탐지할 수 있어야 함.
  - 실패가 났을 때 우아하게 대처할 수 있도록
- **실패는 늘 발생할 수 있다가 전제임.**
  - 네트워크 이슈나, 데이터베이스 커넥션 이슈 
- Log 와 Monitoring 은 필수.

## Recommendations

- Retry failed operations
  - 일시적인 실패의 경우 (= loss Network connectivity) retry 를 하는것.

- Protect failing remote services (Circuit Breaker)
  - 실패가 지속된다면 실패하는 서비스를 너무 많이 호출하는 문제가 생김.
  - **이로인해서 cascading failures (계단식 실패) 가 발생하지 않도록하는게 중요.** 

- Isolate critical resources (Bulkhead)
  - **서브시스템에서의 하나의 실패가 cascade 가 되지 않도록 하는게 중요.**
  - 이게 메인 아이디어중 하나.
  - 실패가 스레드나 소켓 리소스를 끊기게 하지 않도록 리소스를 고립시켜두는게 중요.

- Perform load leveling.
  - 지나친 부하가 올 수 있다면 Queue-Based Load Leveling pattern 을 쓰는게 중요. 이 경우에는 asynchronous 하게 대처가 가능하다.

- Fail over
  - Instance 가 응답하지 않는다면 다른 인스턴스로 fail over 하는 것 .
  - **상태가 없는 웹서버의 경우라면 Load Balancer 뒤에 인스턴스들을 배치해두고, 상태가 있는 경우라면 replica 를 써서 fail over 를 하도록.**
    - 데이터베이스의 결우 결과적 일관성을 필요로 할 것.

- Compensate failed transactions.
  - 일반적으로 분산 트랜잭션은 피하는게 맞다.
    - 여러 서비스들의 coordination 을 필요로 하기 떄문에. 대신에 smaller individual transactions 을 compose 해서 쓰고, 이것이 중간에 실패한다면 undo 연산을 하는 보상 트랜잭션을 내라고 하는듯.
  - 난 Compensating Transaction pattern 이 좋다고 생각하지 않았는데 이건 유용한건가?
    - 이건 일단 결과적 일관성 모델을 따르는 경우에 씀.
    - 복잡한 비즈니스 처리에서 발견되는듯.
    - Eventual consistency vs strong transactional consistency 를 비교.
      - Eventual consistency 가 경쟁을 피하고 성능을 개선할 수 있다고함.
      - **경쟁을 피하는것도 하나의 원칙인듯. 동시성 문제와 병목이 될 수 있어서.**
    - 이 Eventual consistency 와 distributed transaction 은 다르다.
    - Distributed transaction 은 분산환경에서 한번에 다 성공시키는 것으로 strong transactional consistency 를 보장.

- Checkpoint long -running transactions
  - Checkpoints 는 **resiliency** 를 준다. Long-running operation 이 실패했을 경우에.
  - **resiliency 도 메인 키워드 중 하나.** 
  - Last checkpoint 에서 시작할 수 있도록. Mechanism 이 필요하다. 상태를 durable storage 에 저장해서 다시 재개.

- Degrade gracefully
  - 문제가 해결되지 않은 상황에서는 functionality 를 reduce 해서 제공하는 것도 유용할 수 있다.
  - 예로 책의 썸네일 이미지를 가져오는게 실패한다면 placeholder image 를 가져가도록 하는 것.
  - 이로인해서 전체의 서브시스템들이 치명적이지 않도록 하는 것이다.

- Throttle clients
  - 때떄로 작은 유저들이 엄청난 부하를 만들어 내서 다른 유저들이 어플리케이션을 이용하지 못하기도 한다.
  - 이런 경우에 특정 시간동안 throttling pattern 을 써서 유지시키는 것도 방법이다.
    - 라인에선 유명인들의 오픈챗에 대한 것이엇음.

- Block bad actors
  - Client 가 지나치게 악의적으로 행동한다면 블락을 해야한다.

- Use leader election
  - Zookeeper 와 같은 것. **Coordinate 할 작업이 필요하다면 leader 를 세우도록 하는 것.**
  - Leader 가 다운되면 다른 멤버가 리더로 다시 선출되서 a single point of failure 를 막는 것.
  - **A single point of failure 를 막는 것도 중요한 원칙 중 하나.**

- Test with fault injection
  - 실패를 주입해서 회복하는 것을 테스트 할 필요가 있다.

- Embrace chaos engineering
  - Fault injection 의 확장판. 랜덤적으로 실패를 주입하거나 비정상적인 조건들을 production instance 에 주입하는 것. 
