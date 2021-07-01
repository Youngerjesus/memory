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

 


 