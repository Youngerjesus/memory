# Reactor Handling Errors

https://projectreactor.io/docs/core/release/reference/#error.handling

***

## Handling Errors

- Error 는 Reactive Streams 에서 종료 신호로 해석. 에러가 발생하면 이후 후속 처리를 하는게 아니라 `onError` 와 관련된 메소드로 보낸다.
    - 에러를 처리하는 메소드를 정의하지 않으면 `UnsupportedOperationException` 예외가 난다.
    - 또 `Exceptions.isErrorCallbackNotImplemented` 를 통해서 탐지하는 것도 가능.
- 어플리케이션 레이어에서 에러를 처리하는 방법은 많을 것.
    - UI 로 표시할 수도 있고 REST API 에서 전달하는 것도 가능하다.

## Error Handling Operators

에러를 처리하는 연산자들을 소개.

- catch 한 후 정적인 값을 리턴하는 것
    - `onErrorReturn()` 으로 리턴하는게 가능.
    - `onErrorReturn()` 으로 예외를 보고 recovery 하는 것도 가능.

```java
Flux.just(10)
    .map(this::doSomethingDangerous)
    .onErrorReturn(e -> e.getMessage().equals("boom10"), "recovered10"); 
```

- catch 한 후 fallback 메소드를 실행하는 것.
    - `onErrorResume()` 으로 가능.
    - 여기서 들려주는 예는 외부 시스템에서 데이터를 가지고 오려고 했는데 실패하는 경우 로컬 캐시에 있는 기본 값으 가져오는 `대안` 이 있는 경우에 이렇게 처리할 수 있다고함.
    - `onErrorResume()` 도 `onErrorReturn()` 과 같이 예외를 보고 상황별로 처리하는게 가능.

- catch 한 후 다이나믹하게 fallback value 를 계산하는 것.
    - `onErrorResume()` 과 어떻게 보면 동일. 대체 값을 계산하는 것.
- catch 한 후 비즈니스 예외로 감싸서 ReThrow 하는 것.
    - throw 하면 됨.
    - 에러를 변환할거면 `onErrorMap()` 이 있음.
- catch 한 후 로그를 남기고 rethrow 하는 것.
    - 에러는 남기고 계속 처리를 이어서 할거면 `doOnError()`
    - `doOnError()` 에서 에러는 계속 `throw` 되고 있고 계속 이후 처리는 진행됨.
- finally 블록이나 try-with-resource 블록에서 자원 처리하는 것.
    - `doFinally()` 를 이용하면됨.
    - `try-with-resource` 를 이용할거면 `using()` 을 쓰면 됨.

- retry 해보는 것.
    - `retry()` 도 있음.
    - 다만 알아야 할 사실은 retry 할 때 처음부터 다시 시작한다는 거임. 예로 `Flux.interval()` 을 예외가나서 retry 해보면 처음 값부터 다시냄.

- `onError` 와 관련된 메소드는 종료 신호임.
