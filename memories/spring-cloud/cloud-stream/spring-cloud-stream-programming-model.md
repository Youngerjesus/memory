## Programming Model

- 프로그래밍 모델을 이해하기 위한 코어 개념 소개. 

- Destination Binders
  - External messaging system 과 통합하기 위한 책임을 가진 binder
- Bindings
 - External system 과 application 에 제공된 producer 와 consumer 를 Bridge 하는 애
   - Input Binding 과 Output Binding 이 있음. 
- Message
  - Canonical data structure (표준 데이터 구조)
  - 생산자와 소비자가 커뮤니케이션하기 위한 데이터

## Destination Binders 

- External messaging system 과 integration 을 책임지는 확장 구성요소
  - 필요한 설정과 구현을 제공해준다. 
  - Integration 은 connectiity, delegation, routing of message 를 말한다. 

## Bindings 

```java
@SpringBootApplication
public class SampleApplication {

	public static void main(String[] args) {
		SpringApplication.run(SampleApplication.class, args);
	}

	@Bean
	public Function<String, String> uppercase() {
	    return value -> {
	        System.out.println("Received: " + value);
	        return value.toUpperCase();
	    };
	}
}
```

- 이렇게 하면 바인딩 된다. 
- binder dependencies 와 auto-configuration 에 의해서 만들어짐  
- context beans 는 `Supplier`, `Function`, `Consumer` 이렇게 있다. 

## Binding and Binding Names 

- binding 은 추상화 된 과정 
- 이런 추상화는 이름이 필요하다. 만약에 추가적인 binding 을 더 할려면 이름을 줘야할 것. 
  - `spring.cloud.stream.bindings.input.destination=myQueue`
    - 에서 `input` 이 binding 이름이다. 

### Functional binding name

```java
@SpringBootApplication
public class SampleApplication {

	@Bean
	public Function<String, String> uppercase() {
	    return value -> value.toUpperCase();
	}
}
```

- `input`: <functionName> + -in- + <index>
- `output`: <functionName> + -out- + <index>
- in 과 out 은 바인딩 타입을 말함.
- index 는 input ouput binding 의 index 를 말한다. 하나의 binding 만 있다면 0임.
  - Functions with multiple input and output arguments 방식도 있음. 
- 예제
  - `--spring.cloud.stream.bindings.uppercase-in-0.destination=my-topic`
    - function 의 input 을 destination 에 매핑 시킬 때 
    - `uppercase-in-0` 이 바인딩 이름. 
  - `--spring.cloud.stream.function.bindings.uppercase-in-0=input`
    - readability 를 위해서 implict binding 이름을 explicit binding 이름으로 매핑하는 법
    - `spring.cloud.stream.function.bindings.<binding-name>` 을 이용한다.
    - 이제 `input` 이라는 이름을 쓸 수 있음. 

### Functions with multiple input and output arguments

- spring cloud stream 3.0 이후 버전부터 나온 것.
- multiple inputs 과 multiple output 을 위해서 나옴. 
  - BigData 처리와 Data aggregation 을 위해서
    - 들어오는 데이터 타입이 다양하니까. 다양한 유형을 분류하기 위해서
- project reactor 사용 가능.

### Explicit binding creation 

- binding 이 function 과 연결되지 않을 때 바인딩을 생성하는 방법.
  - `Spring Integration Framework` 와 같은 다른 프레임워크를 쓸 때 필요. 
  - `MessageChannel` 을 통한 직접적인 접근이 필요함.
- `spring.cloud.stream.input-bindings` 와 `spring.cloud.stream.output-bindings` 를 통해 생성이 가능하다.

## Producing and Consuming Messages 

- spring cloud stream 어플리케이션은 function 을 작성하고 @Bean 을 붙이면 된다. 

### Spring Cloud Function support 

- `java.util.function.[Supplier/Function/Consumer` 을 써야한다. 
- external destination 과 function 을 바인딩하기 위해선 `spring.cloud.function.definition` 을 써야한다.
  - 하나의 빈만 쓰고 있다면 생략해도 됨. 자동으로 찾아줌. 
- Consumer (Reactive)
  - `Function<Flux<?>, Mono<Void>>` 와 `Consumer<Flux<?>>` 의 차이는 뭐지
    - Consumer 타입은 들어오는 Flux 에 대해서 subscribe 해야한다 이런건가? ㅇㅇ 

### Suppliers 

- Function 과 Consumer 는 이벤트에 의해서 트리거 되고 destination 에 데이터를 보낸다. 
- Supplier 는 자신만의 트리거 되는 범주가 있다. 그리고 in-bound destination 을 subscribe 하고 있지 않다.
- 누가 supplier 를 호출할까? 
  - 프레임워크는 매초마다 호출하도록 제공해준다. 
  - 물론 이런 Polling configuration 설정도 있다.

```java
@SpringBootApplication
public static class SupplierConfiguration {

    @Bean
    public Supplier<Flux<String>> stringSupplier() {
        return () -> Flux.fromStream(Stream.generate(new Supplier<String>() {
            @Override
            public String get() {
                try {
                    Thread.sleep(1000);
                    return "Hello from Supplier";
                } catch (Exception e) {
                    // ignore
                }
            }
        })).subscribeOn(Schedulers.elastic()).share();
    }
}
```

- 이런 리액티브 스타일이라면 딱 한번만 호출되도록 한다.
  - 그래서 딱 한번의 유한한 스트림만 생성된다. 
- 리액티브 스타일로 주기적으로 생성하게 하려면 어떻게 하면 될까? 

```java
@SpringBootApplication
public static class SupplierConfiguration {

	@PollableBean
	public Supplier<Flux<String>> stringSupplier() {
		return () -> Flux.just("hello", "bye");
	}
}
```

- 이 매키니즘은 unpredictable threading mechanism 에 의한 Polling 방식으로 구현된다.  

### Consumer (Reactive)

- return type 이 void 임. 
- 그래서 framework 에서 subscribe 할 수가 없음.
- 대부분의 상황에서 `Consumer<Flux<?>>` 이건 적합하지 않고 `Function<Flux<?>, Mono<Void>>` 가 적합할 것. 

