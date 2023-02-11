# Netty - asynchronous and event-driven

## This Chapter Covers

- Networking in Java
- Introducing Netty
- Netty’s core components

## Blocking 의 문제점

- Thread Context Switching
- Blocking Call 의 호출 때문에 걸리는 대기시간들. 요청이 몰렸을 때 특히 심하다.
- 많은 스레드가 생기면 그 수만큼 Stack 이 생긴다. 64KB to 1MB. 메모리 부하.

## Async Native Socket Library

- `setsockopt()` 를 통해서 소켓 설정을 하면 read/write call 에 대해서 데이터가 없으면 즉시 리턴하도록 하는, 블라킹 되지 않도록 할 수 있다.
- 그리고 event notification API 를 쓰면 데이터를 읽을 준비가 되면 이벤트를 받을 수 있도록.
    - 이건 `java.nio.channels.Selector` 를 쓴다.
    - 이 경우에는 Single Thread가 여러개의 Socket (= Connection) 을 처리할 수 있도록 되는 것.
        - 그러므로 절대적인 스레드 수가 줄어드니까 Blocking 의 문제를 해결할 수 있다.

## Netty 를 배우는 이유

- Scalability 를 위해서. 더 많은 규모의 트래픽을 처리하기 위해서.
- 이벤트가 오고 그것에 대해서 처리하는 부분을 가르켜준다고 한다.

## Netty’s core component s

- channels
- Callbacks
- Futures
- Event and Handlers

### Channels

- Java NIO 의 기본 구성요소.
- I/O 요소들과 연결되는 connection 이다.
    - I/O (= reading, writing) 을 하는 요소와의 file, hardware device, network socekt, program component
- 채널을 통해서 데이터가 들어오고 나간다고 생각하면 된다.
- 이건 열려있을수도, 닫혀있을수도 있다.

### Callbacks

- callback 은 메소드다. 다른 메소드에게 전달되는 참조되는 메소드.
- 보편적으로 사용되며 가장 많이 사용되는 곳은 완료된 후에 알려주는 용도로.
- Netty 에서는 Callback 이 내부적으로 이벤트를 처리할 때 쓰인다. Callback 이 트리거 될 때 이벤트는 `ChannelHandler` 에 의해서 핸들링 된다.

```java
public class ConnectHandler extends ChannelInboundHandlerAdapter {
    @Override
    public void channelActive(ChannelHandlerContext ctx) throws Exception {
		    System.out.println("Client " + ctx.channel().remoteAddress() + " connected");
		}
}
```

- `channelActive()` 메소드는 new Connection 이 맺어졌을 때 콜백으로 실행된다.

### Futures

- future 는 callback 과 같이 application 에게 완료를 알리는 또 다른 방법이다.
- future 는 asynchronous operation 결과를 가지고 있다.
- 수동으로 완료가 되었는지 검사해야한다.
    - 그래서 Netty 에서는 이를 해결하기 위해서 `ChannelFuture` 라는 걸 쓴다. 그리고 여기에다가 `ChannelFutureListener` 인스턴스를 등록시킨다. 완료되었을 때 실행하는 콜백 메소드인 `operationComplete()` 를 호출하도록해서 수동으로 검사를 안하기 위해서.
        - Netty 의 outbound I/O 같은 경우에도 `ChannelFuture` 를 리턴한다. 블락당하지 않도록 하기 위해서.

**Callback In action**

```java

// DOes not block
ChannelFuture future = channel.connect(
    new InetSocketAddress("192.168.0.1", 25)); // (1)

future.addListener(new ChannelFutureListener() {
		@Override
    public void operationComplete(ChannelFuture future) {
        if (future.isSuccess()){
            ByteBuf buffer = Unpooled.copiedBuffer(
                "Hello",Charset.defaultCharset());
            ChannelFuture wf = future.channel()
                .writeAndFlush(buffer);
            ....
        } else {
            Throwable cause = future.cause();
            cause.printStackTrace();
		} 
} 
```

- callback 과 future 는 서로 상호보완적인 매커니즘이라는 걸 이해햐야한다.
    - callback 입장에서 future 는 비동기적으로 실행할 수 있도록 해주고,
    - future 입장에서 callback 은 수동으로 완료시점을 검사하지 않아도 되니까.

### Events and handlers

- Netty 는 이벤트를 구별한다. 이벤트는 상태의 변화를 아리기 위해서 쓴다.
- 주로 이벤트는 이런 액션을 하도록 유발한다.
    - Logging
    - Data transformation
    - Flow-control
    - Application logic
- Netty 의 이벤트는 분류될 건데 inbound 의 event 의 경우는 이렇다.
    - Active or inactive connections
    - Data reads
    - User events
    - Error events
- Netty 의 outbound 의 event 는 이렇다.
    - Openong or closing a connection to a remote peer
    - Writing or flushing data to a socket
- inbound 와 oubound 의 event 가 생기면 각각의 ChannelHandler chain 들에 의해서 처리되나간다는 걸 알자. 콜백 방식으로.

### Futures, callbacks, and Handlers

- event 가 들어오면 channelHandler 가 받아서 처리한다. channelHandler 는 chainning 으로 이뤚있어서 쉽고 효율적으로 처리해나간다. 그리고 각각의 처리해 나가는 과정에서 Futures 나 callback 을 쓰는 것.

### SELECTORS, EVENTS, AND EVENT LOOPS

- Netty 는 Selector 의 코드가 어플리케이션에서 추상화되어있다.
- `EventLoop` 는 각각의 `Channel` 에 할당된다. 여기서 이벤트들을 가져오겠지.
    - EventLoop 는 event 를 가져와서  ChannelHandler 에게 dispatching 한다.
    - 추가적인 액션이 있다면 Scheduling 한다.
    - EventLoop 는 하나의 스레드에 의해서 모든 I/O event 를 처리해나간다. channel 에서 발생하는.
        - 그래서 ChannelHandler 의 동기화를 걱정할 필요없다. 각기 다른 스레드에서 but 하나 에서 처리해나가니까.
    - Selector 가 EventLoop 에 의해서 추상화 된 것으로 알면 되겠네.
