# 작업 실행 (Task Execution)

### 병렬 프로그래밍은 작업인 Task 를 여러 스레드로 실행하는 걸 말한다.
- 뻔하지만 작업은 독립적인 단위로 이뤄져야 한다. 
- 서로 간의 순서가 필요하거나 의존적인 경우는 병렬 프로그래밍에 맞지는 않다. 멀티 스레드 환경에선 순서를 예측할 수 없기 떄문에.

### 병렬 프로그래밍의 장점은 처리량을 높일 수 있고, 클라이언트에게 응답을 주는 경우라면 응답 속도를 높일 수 있다. 
- 여러 스레드를 이용해서 비동기적으로 실행할 수 있기 때문에 응답 시간을 늘릴 수 있다.

### 병렬 프로그래밍에서 주의할 점은 뭐 스레드 풀을 이용하는 것이다. 
- 스레드를 생성하고 폐기하는 비용 자체가 싼 편은 아니다. OS 마다 다르겠지만.
- 작업마다 스레드를 만든다고 생각하면 문제가 좀 많을 수 있다.
  - JVM 기준으로 스레드 하나마다 두 개의 스택이 생긴다. Java 실행을 위해 기본적으로 제공하는 스택과 네이티브 스택. 이것만 하더라도 메모리를 조금씩 먹는다. 너무 많은 스레드가 있다면 OutOfMemoryError 가 발생할 확률이 있음.
  - 그리고 CPU 코어보다 스레드가 월등히 많다면 경쟁 상태가 일어나고 이로인해서 응답시간이 많이 느려질 수 있다. 
  - 딱 CPU 코어를 풀로 사용할 수 있는 정도의 스레드가 나을듯. 남는 코어가 있고 스레드가 대기 상태면 그때마다 하나씩 만들던가. 
- Java 에서는 스레드 풀을 이용하기 위해서 Executor 인터페이스를 제공해주고 이 구현체를 사용하면 된다.
- Executor 는 작업 등록과 작업 실행을 분리시켜 놓고 그 중간의 작업 정책을 유연하게 변경할 수 있다는 장점과 스레드 관리 안정성을 제공해준다. (유연성이란 말은 OCP 를 말함.)
- Executor 을 사용할 땐 종료를 어떻게 할 것인가에 대해서 고민을 좀 해봐야 할 수도 있다.
- 뭐 상황을 예로 들면 현재 종료를 해야하는데 Executor 에서 실행중인 스레드가 있고, 등록되어 있는 작업들이 있다면 어떻게 할 것인가? 에 대한 고민이다.
- 기본적으로 Executor 의 상태는 running (실행 중), shutting down (종료 중), terminated (종료) 이렇게 있는데 안전하게 종료는 shutdown() 메소드를 호출하면 된다. 이러면 이제 작업을 더 이상 받지않고 현재 작업을 모두 처리한 다음에 종료된다.
- 강제 종료는 shutdownNow() 메소드를 호출하면 되는데 이 경우에는 현재 진행중인 작업을 포기하는 것이므로 이에대한 처리를 해줘야 하겠다. 
- ExecutorService 의 하위 클래스인 ThreadPoolExecutor 는 종료 상태로 진행중이면 작업을 받지않고 들어오는 작업이 있다면 RejectedExecutionException 이 발생한다.

### 지연 작업이나 주기적인 작업을 사용할거면 Timer 를 사용하지는 말자.
- 대안으로 ScheduledThreadPoolExecutor 를 사용하자.
- Timer 는 일단 싱글 스레드 기반이라 하나의 작업이 오래 걸리면 다음 작업이 제 시간에 실행하지 못할 수도 있다.
- 그리고 절대 시간을 받을 수도 있어서 운영체제의 시간이 바뀐다면 문제가 생길수도 있고 예외가 발생하면 새롭게 스레드를 만들거나 하는 Failover 정책이 없다. (문제가 많음.)
- 스케쥴 작업이 필요한 게 있다면 DelayQueue 를 이용하는 것도 하나의 방법이 될 수 있다. 
- DelayQueue 는 각 객체마다 Delayed 를 가지고 이 시간이 되야지 처리할 작업으로 가져갈 수 있게 해준다.

### 병렬로 처리할 만한 작업은 어떤게 있을까?
- 브라우저 어플리케이션의 경우 렌더링 작업은 병렬로 실행할만 하다. 
- 렌더링 과정을 심플하게 보면 텍스트 렌더링과 이미지 렌더링 크게 두 개의 독립적인 작업이 있을 것.
- 이미지 작업 같은 경우는 병렬로 나누기 적합하다. 이미지를 다운 받을 수 있는 url 이 있을 것이고 이것들을 다운로드 해야하는데 이 I/O 작업을 병렬로 하면 되니까.

### Callable 과 Future 그리고 CompletionService

- Runnable 의 단점은 실행하고 나서 결과를 가져올 수 없다는 점. 예외가 나는걸 받고 싶다던지 리턴 값이 필요한 경우라던지 이런 경우에는 적합하지 않다.
- Callable 은 이 단점을 보안해서 결과를 가져올 수 있다.
- Future 는 실행의 리턴으로 Future 로 받으면서 사용하는데 결과를 가져올 수 있을 뿐 아니라 작업 진행중에 취소되었는지, 예외가 났는지, 결과가 있는지, 취소할 것인지 등을 할 수 있다.
- Future 를 사용할 땐 get() 메소드를 통해 결과를 가져오는데 아직 실행되지 않았더라면 Blocking 될 수 있다는 점이다.
- 이게 좀 문제가 될 수 있는데 여러 개의 작업을 Executor 에 전달하고 Future 를 통해 받고나서 get() 메소드를 호출하면 모든 작업이 완료되어야지 응답을 받을 수 있다.
- 하나하나가 완료될 때마다 결과를 가져오고 싶다면 CompletionService 를 이용하면 된다. (Java 8 부터는 CompletableFuture 를 사용하자. 더 많은 기능을 제공해준다.)
- Future 을 사용하는 예제 중에는 여행사 예약 어플리케이션에서 사용하는 게 있다. 
- 여행사 어플리케이션에서는 가격을 제공해주는 여러 업체들이 있을 것이고 병렬적으로 실행해서 모두 결과를 가져오면 성능상에 이점이 확실히 있다.
- 근데 이제 모든 업체에서 가격을 가져오기에는 응답 시간이 좋지은 않을 것이다. 그래서 특정 시간동안만 실행하도록 하고 그 시간동안 실행되지 않았으면 작업 취소를 하도록 해서 리소스를 덜 쓰면서 응답시간을 줄이는 방법을 쓸 수 있다. ((이는 invokeAll() 메소드를 사용하면 된다.))
- 여기에 Future 를 써서 가격을 가져온 집합중에 가격을 보고 노출 우선순위를 결정하도록 하는 방식으로 사용하면 된다.
- CompletionService 같은 경우는 완료되는 작업마다 Queue 에 하나씩 쌓이고 이를 꺼내서 사용하는데 이를 통해서 브라우저 어플리케이션 렌더링 작업의 효율을 높일 수 있다. 
- 이미지 다운로드가 완료된 것만 가져와서 하나씩 렌더링 하면 되니까. 이 작업은 Future 보다 CompletionService 가 우월하다. 


