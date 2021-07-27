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

- 