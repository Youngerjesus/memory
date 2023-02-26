# Spring Webflux WebClient 

https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-client

## 2. WebClient 

- webflux 에서 나온 Http request 를 도와주는 애.
- Reactor 기반. asynchronous non-blocking 이다. 
- support Streaming. 
  - 어떻게 지원한다는거지? 
  - streaming 은 response 타입이 `text/event-stream, application/x-ndjson` 인 경우. 
  - 주기적으로 데이터를 계속 보내는게 중요함. disconnect 를 탐지하기 위해서.
- codec 에 의존한다. 
  - request 에 encoding 해주고, response 에 decoding 해주는 것 .

### 2.1 Configuration

- static factory method 로 사용을 지원함. 가장 간단하게 만드는 방법 
  - `WebClient.create()`
  - `WebClient.create(String baseUrl)`
- builder 를 통해서도 지원함.
  - 만들 때 어떤 옵션들이 있는지 알아두자.  
  - `uriBuilderFactory`: baseUrl 등록 가능.
  - `defaultUriVariables`: uri template 을 확장할 떄 쓰는 variables
    - 정확히 뭐지? 
      - uriBuilderFactory 이거와 같이 쓰면 무시됨.

#### uriBuilderFactory example 

````java
 Map<String, ?> defaultVars = ...;
 DefaultUriBuilderFactory factory = new DefaultUriBuilderFactory();
 factory.setDefaultVariables(defaultVars);
 WebClient client = WebClient.builder().uriBuilderFactory(factory).build();
````

- uriBuilderFactory 에 defaultUriVariables 이걸 넣네. 

  - `defaultHeader`: 모든 요청에 사용할 헤더 
  - `defaultCookie`: 모든 요청에 사용할 쿠키 
  - `defaultRequest`: 모든 요청에 커스텀하게 사용할 consumer. 요청에다가 적용해서 바꿀 떄 쓴다. 
    - 정확히 뭐지? 
  - `filter`: 모든 요청에 사용할 filter 
  - `exchangesStrategies`: HTTP message reader/writer customizations
    - 정확히 뭐지? 
      - request body 에 encoding 하고 Writing 할 message writer 가 있다. 
      - response body 에 Decoding 하고 reading 할 message Reader 가 있다. 
  - `clientConnector`: Http Client library settings
    - 정확히 뭐지? 
      - HttpClient 커스터마이징 할 수 있음. 
        - compression 할 건지
        - keepAlive 적용할 건지
        - channelOption 설정도 가능. 
          - https://netty.io/4.1/api/io/netty/channel/ChannelOption.html
      - Reactor Netty 를 쓸 건지도 가능. (기본)  
      - read/connect timeout 

- 한번 WebClient 는 만들어지면 불변이다. 복사해서 사용할 수 있음. 클론해서.
- 기본적으로 global Reactor Netty resources 쓴다. 이걸 권장함. 
  - eventloop 와 thread 그리고 connection pool 를 공유해서 씀.

### 2.1.1 MaxInMemorySize

- codec 이라는 게 있는데 데이터를 메모리에서 버퍼링해서 가지고 있는다는듯. 메모리 이슈 때문에 제한으로 256KB 만 가지고 있다. 

```kotlin
val webClient = WebClient.builder()
        .codecs { configurer -> configurer.defaultCodecs().maxInMemorySize(2 * 1024 * 1024) }
        .build()
```

- 1024 * 1024 니까 이렇게하면 MB 단위로 시작함. 

### 1.2.5. Codecs

- spring-web 과 spring-core 모듈에서 high level object 를 byte 로 직렬화 역직렬화 해주는 기능. non-blocking i/o with reactive streams back pressure 를 통해서.
- 이걸 가능하게 하는 지원들 
  - `Encoder` 와 `Decoder` HTTP 와는 독립적. 너무 low level 이라서. 
  - `HttpMessageReader` 와 `HttpMessageWriter` 내부에 `Encoder` 는 `EncoderHttpMessageWriter` 로 래핑된다. Decoder 도 마찬가지.
  - `DataBuffer` 는 byte 포현을 추상화한다.

## 2.2. retrieve()

- `retrive()` 를 통해서 응답을 어떻게 뽑아낼건지 정의할 수 있다. 

```kotlin
val client = WebClient.create("https://example.org")

val result = client.get()
        .uri("/persons/{id}", id).accept(MediaType.APPLICATION_JSON)
        .retrieve()
        .toEntity<Person>().awaitSingle()
```

- body 만 뽑아낼려면. 

```kotlin
val client = WebClient.create("https://example.org")

val result = client.get()
        .uri("/persons/{id}", id).accept(MediaType.APPLICATION_JSON)
        .retrieve()
        .awaitBody<Person>()
```

- 4xx or 5xx 에러가 발생하면 `WebClientResponseException` 이 발생한다. 
- 에러를 다룰러면 다음과 같이하면 된다. 

```kotlin
val result = client.get()
        .uri("/persons/{id}", id).accept(MediaType.APPLICATION_JSON)
        .retrieve()
        .onStatus(HttpStatus::is4xxClientError) { ... }
        .onStatus(HttpStatus::is5xxServerError) { ... }
        .awaitBody<Person>()
```


