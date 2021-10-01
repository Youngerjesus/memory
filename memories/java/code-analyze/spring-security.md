# 스프링 시큐리티 코드를 분석하면서 얻은 팁

***

### CollectionUtils

- 그냥 컬렉션을 만들지 않고 CollectionUtils 를 사용하는 이유는 뭔데? 몰라 내부 구현체도 같음.
- newLinkedHashMap()

### LinkedHashMap

- 일반 HashMap 을 쓰지않고 LinkedHashMap 을 쓰는 이유는 순서를 보장하기 위함인가?
- HashMap 같은 컬렉션은 어느정도 채워지면 자동으로 리사이즈를 한다.
- 기본적으로 75% 정도 차면 리사이즈를 한다. (리사이즈는 두배로 하고  두배로 하는 연산은 << 1 이런 비트 연산자를 사용한다.)
- afterNodeAccess 메소드를 통해서 해시테이블에 들어온 순서대로 LinkedList 를 만든다.  여기서는 참조만 가지고 있다.

### Operation

- 두 배를 하는건 << 비트 연산자를 사용하네.

### Concurrency

- 한번만 딱 실행하도록 하는걸 AtomicVariable 을 통해서 가능하네.

```java
AtomicBoolean building = new AtomicBoolean();

if (building.compareAndSet(false, true) {
	this.object = doBuilding();
	return this.object;
}

throw new AlreadyBuiltException("This object has already been built");
```

### OOP

- AuthenticationManager 인터페이스는 하나의 메소드만 가지고 있다. authenticate()
    - 하나의 동작만 있으니까 무슨 역할을 하는지 명확하게 이해가 가능하네.
    - public 메소드의 숫자가 너무 많다면 많은 역할을 하는게 아닌가 라고 생각을 해볼 필요가 있겠네.
- 내가 이 객체를 처리할 수 있는 객체인지 확인하고 싶은 경우에는 support 메소드 같은 이름을 사용하고 구현에는 구현체를 비교하는 식으로 사용된다.
    - 이렇게 안하고 instanceof 메소드를 통해서 비교하는 방법도 있다.
    
    ```java
    public boolean support(Class<?> authentication) {
    	return (AnonymousAuthenticationToken.class.isAssignableFrom(authentication)); 
    }
    ```
    
    - Abstrct Class 에서 추상화 메소드를 사용하는 타이밍을 알곘다.
        - 일반적인 데이터 처리 플로우가 있다면 거기서 실제로 너가 해줘야하는 작업이 있다면 그걸 추상화 메소드로 해놓는 것.
        - **이렇게 추상화 메소드로 남겨놓았다면 너가 제대로 했는지 체크를 하는 작업이 필요하다고 생각하는데 이런 경우는 Null Checking 만 하네. Null 이 아니라면 제대로 스펙을 보고 사용했곘지 라는 생각이 있는것 같다. 그리고 이러렇게 체킹을 할 떄는 Assert 클래스를 좀 사용하네.**

### Collecition

- 변하면 안되는 객체가 있다면 Collections.unModifiableXXX() 메소드를 통해서 가능하다.

### Checking Mechanism

- PreCheck , PostCheck 이런 것들이 있네.
- 미리 할 필요가 없는 것들은 이렇게 단계를 나눠서 하구나.

### Caching Mechanism

- 한번 인증 처리를 한 유저는 캐싱을 해둔다. **(재사용될 경우가 높다면 캐싱을 적극적으로 사용하구나.)**

### Logging Mechanism

- FilterChain 들이 하나씩 수행을 완료하면 그거에 대한 로그를 찍네.  **(즉 특정 스텝마다 로그를 남기는 것 같다.)**

```java
if (logger.isTraceEnabled()) {
	logger.trace(LogMessage.format("Invoking %s (%d/%d)", nextFilter.getClass().getSimpleName(), this.currentPosition, this.size)
}
```

### TemplateMethod

- TemplateMethod 패턴이 적용된 경우가 많다. 사용자가 정의할 수 있는 부분의 메소드를 빼놓고 그 부분은 오버라이딩 하게 두고 그 외 플로우는 추상 클래스에서 정의를 해놓는 방식.

### Dictionary

- Access Control 은 인가쪽에서도 사용하는 용어다.

### Callable

- No Argument 이면서 Return 을 해주는 Functional Interface 이다.
- Runnable 과 유사하며 Callable 에서 정의한 Task 는 다른 스레드에서 실행된다.
- Spring MVC 컨트롤러 핸들러에서 Callable 로 리턴타입을 정의했다면 Async 하게 처리하는게 가능하다.
    - 현재 진행중인 스레드는 바로 반환하고 Callable 에서 스레드를 만들어서 실행하고 완료되면 응답이 나가는 구조로 진행된다.