# Mono.just() vs defer() vs fromSupplier() vs create() – Part 2

https://codersee.com/mono-just-defer-fromsupplier-create-part-2/

***

## Null Handling

### Mono.just

- null 을 리턴하면 안됨 에러남. 왜냐하면 내부적으로 MonoJust.class 를 만들어야 하기 떄문에.
- Mono.just 이후에 onErrorReturn() 과 같은 에러 핸들링 콜백을 구현할 수 없음.
    - 대신에 Mono.justOrEmpty() 와 같은 것들을 써야함. 이건 에러 대신에 complete signal 을 내보냄.

### Mono.defer

- null 을 리턴하면 에러가남.
- 에러가나면 error 시그널을 받을 수 있어서 이후 onErrorXXX() 와 같은 콜백 처리 가능.

### Mono.fromSupplier

- null 을 리턴해도 에러가 나지 않음. 널을 리턴해도 complete empty 시그널이 남.
- 이걸 처리하려면 defaultEmpty 에서 처리할 수 있음. 
