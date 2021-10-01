# @Transactional 

****

#### @Transaction 에노테이션을 붙이면 생기는 일 

```java
public Object invoke(MethodInvocation invocation) throws Throwable {
    // 트랜잭션 시작 
    TransactionStatus status = this.transactionManager.getTransaction(new DefaultTransactionDefinition()); 
    
    try{
        Object ret = invocation.proceed(); 
        this.transactionManager.commit(status); // 트랜잭션 종료  
    } catch(RuntimeException e) {
        this.transactionManager.rollback(this); // 트랜잭션 종료  
        throw e; 
    }
}
```
- 새로운 트랜잭션을 만들고 해당 작업을 실행하고 commit() 을 한다. 만약 중간에 런타임 예외가 난다면 롤백이 된다.

- 트랜잭션 성질상 Atomic 하기 때문에 모든 작업을 완료하던지 실패하던지 한다. 

- 트랜잭션을 만들때 트랜잭션매니저의 getTransaction() 메소드를 통해서 만드는데 트랜잭션 전파 속성에 따라 트랜잭션을 만들기도 하고 참여시키기도 하고 제외시키기도 한다. 
 
#### 트랜잭션 전파

만약 기존의 트랜잭션 작업 중에 새로운 트랜잭션을 만나면 두개의 트랜잭션을 합칠건지 분리할건지 결정하는 옵션 

#### PROPAGATION_NOT_SUPPORTED 를 사용하는 이유 

트랜잭션 전파 속성이 NOT_SUPPORTED 라면 트랜잭션 참여를 안한다. 트랜잭션 없이 동작한다. 

이 말만 들으면 그러면 애초에 트랜잭션 에노테이션을 붙이지 않으면 되지 않냐고 물을 수 있는데 한가지 상황을 가정해보자 A 메소드에 기본 @Transactional 이 붙어있고
B 메소드에는 @Transactional 이 없다. A 메소드가 B 메소드를 호출하면 B 메소드도 트랜잭션에 포함된다. 그렇기 때문에 이게 마음에 안들 수 있으니까 그냥 NOT_SUPPORTED 를 통해서 제외시키는게 낫다. 


#### PROPAGATION NESTED 

중첩 트랜잭션을 만드는 전파 속성이다. 이 속성은 메인 트랜잭션이 롤백되면 중첩 트랜잭션도 같이 롤백되지만 중첩 트랜잭션이 런타임 예외를 만든다고 해서 메인 트랜잭션이 롤백되지는 않는다.

주로 로그를 DB 에 남길때 많이 사용했는데 메인 비즈니스 작업이 런타임 에외가 나면 로그도 같이 롤백되는게 맞지만 메인 비즈니스 로직은 정상적으로 작동하지만 로그만 DB 에서 예외가 나는 경우에는 롤백되면 안된다.
이런 경우에 주로 사용했다. 
  
#### Serializable

인덱스 칼럼이나 Where 절애 쓰는 칼럼에 대한 잠금을 걸 수 있다.  

이 격리를 이용할때는 Where 절에 쓰는 칼럼을 기반으로 잠금을 걸어야할 때 주로 사용한다.

***

## Spring @Transactional mistakes everyone did

https://medium.com/javarevisited/spring-transactional-mistakes-everyone-did-31418e5a6d6b

### 1. Invocations within the same class

다음과 같이 같은 빈에서 @Transactional 메소드를 호출한다면 이는 적용되지 않는다.

```java
public void registerAccount(Account acc) {
    createAccount(acc);

    notificationSrvc.sendVerificationEmail(acc);
}

@Transactional
public void createAccount(Account acc) {
    accRepo.save(acc);
    teamRepo.createPersonalTeam(acc);
}
``` 

- createAccount() 메소드는 @Transactional AOP 가 적용되지 않는다. 

왜냐하면 스프링 AOP 는 프록시 빈을 통해서 AOP 를 적용하는데 내부에서 메소드를 호출하는건 프록시를 통한 호출이 아니라 
직접 Access 하는 거기 떄문이다. 이와 비슷한 경우가 @Cacheable 메소드도 self-invocation() 문제가 발생한다. 

이를 해결하는 방법은 다음과 같다. 

- Self-Inject

- Create another layer of abstraction

- Use TransactionTemplate in the registerAccount() method by wrapping createAccount() call 

#### Self Inject Solution

````java
@Service
@RequiredArgsConstructor
public class AccountService {
    private final AccountRepository accRepo;
    private final TeamRepository teamRepo;
    private final NotificationService notificationSrvc;
    @Lazy private final AccountService self;

    public void registerAccount(Account acc) {
        self.createAccount(acc);

        notificationSrvc.sendVerificationEmail(acc);
    }

    @Transactional
    public void createAccount(Account acc) {
        accRepo.save(acc);
        teamRepo.createPersonalTeam(acc);
    }
}
````

- @Lazy 에노테이션은 spring 3.0 부터 등장했는데 IoC 컨테이너에게 빈의 초기화를 늦춰서 생성하도록 하는걸 말한다.
원래는 어플리케이션이 시작할때 start up 할 때 빈이 만들어진다. (이렇게 해야 초기에 빈의 생성의 문제를 감지할 수 있기 떄문에)
 

#### 2. Handling not all Exceptions 

스프링 트랜잭션 같은 경우는 RuntimeException 과 Error 의 경우에만 자동으로 rollback 된다. 만약에 어떠한 checked Exception 에 대해서 롤백을
해야하는 경우라면 이를 따로 @Transactional 에 다음과 같이 추가해줘야 한다.
 
```java
@Transactional(rollbackFor = StripeException.class)
public void createBillingAccount(Account acc) throws StripeException {
    accSrvc.createAccount(acc);

    stripeHelper.createFreeTrial(acc);
}
``` 

#### 3. Transaction isolation levels and propagation 

각각의 Transaction 격리 수준에 대해서 정확하게 이해하지 않으면 발견하기 어려운 문제들을 만날 수 있다. 

하나의 트랜잭션에서 단순히 다른 메소드에서 실행하는게 아니라 일부는 다른 트랜잭션을 사용하도록 하고 싶다면 propagation 옵션으로 
REQUIRED_NEW 옵션을 주면 된다. 

#### 4. Transaction do not lock data 

다음과 같은 트랜잭션이 있다고 가정해보자. 

```java
@Transactional
public List<Message> getAndUpdateStatuses(Status oldStatus, Status newStatus, int batchSize) {
    List<Message> messages = messageRepo.findAllByStatus(oldStatus, PageRequest.of(0, batchSize));
    
    messages.forEach(msg -> msg.setStatus(newStatus));

    return messageRepo.saveAll(messages);
}
```

트랜잭션의 경우 lock 을 따로 걸지 않기 때문에 SELECT 이후 UPDATE 를 실행하는 구문의 경우 연속으로 실행되는 경우가 있다.

이를 해결하기 위한 방법은 다음과 같다.

- REPEATABLE READ 로 격리 수준을 올리는 것. 

- SELECT FOR UPDATE 쿼리로 데이터베이스 단게에서의 락을 거는 것. 

- Optimistic Lock 을 이용하는 것.

#### 5. Two different data sources 

다음과 같이 Datasource 가 두 개 있고 여기에 데이터를 저장한다고 가정해보자. 

```java
@Transactional
public void saveAccount(Account acc) {
    dataSource1Repo.save(acc);
    dataSource2Repo.save(acc);
}
```

이 경우 기본 값으로 설정한 하나의 Datasource 에만 성공적으로 트랜잭션이 처리된다.

만약 여러 개의 Datasource 를 사용하고 싶다면 ChainedTransactionManager 나 JtaTransactionManager
에 대해서 알아봐야 한다. 

ChainedTransactionManager 를 사용할 때 주의점은 에러가 나면 롤백이 나는 과정을 이해하는게 중요하다. 다음과 같다. 

```java
1st TX Platform: begin
  2nd TX Platform: begin
    3rd Tx Platform: begin
    3rd Tx Platform: commit
  2nd TX Platform: commit <-- fail
  2nd TX Platform: rollback  
1st TX Platform: rollback
```  

- 즉 3번 트랜잭션은 롤백이 되지 않는다.

JtaTransactionManager 은 분산 트랜잭션 시스템에서 사용되며 2-phase commit 을 지원한다. 


  
 


  