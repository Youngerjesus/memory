# 스레드 덤프 분석하기 

## 학습 목표
- 스레드 덤프란?  
- 스레드 덤프 생성하는법 
- 스레드 덤프 해석하는법 

- 효과적인 스레드 덤프 분석을 위해서는 언제 생성해야할까? 
  - 어플리케이션이 죽기 직전에 남겨야할까? 
- 스레드 덤프가 제공해주는 정보들
- 스레드 덤프가 풀 수 있는 문제

## 스레드 덤프란? 

java process 의 현재 실행되고 있는 상태의 스냅샷을 뜨는 것. 

## 스레드 배경지식 

### 스레드 동기화

- 자바에서는 Monitor 라는 객체를 통해서 스레드 동기화를 한다.
- Monitor 는 하나의 스레드에서만 가질 수 있다. 
- 다른 스레드가 Monitor 를 가지기 위해서는 대기해야한다. 

### 스레드의 상태들 

- NEW: 생성은 되었으나 실행은 안된 것.  
- RUNNABLE: 현재 CPU 를 점유하고 있는 상태 
- BLOCKED: Monitor 를 소유하기 위해서 대기하고 있는 상태
- WAITING: wait(), join(), park() 메소드를 통해서 대기하고 있는 상태. 
- TIMED_WAITING: WAITING 상태와 동일하다. 차이점은 최대 대기 시간을 지정할 수 있다. 시간에 따라서 WAITING 상태가 변경될 수 있다는 점. 
- TERMINATED: 모든 작동을 마친 상태. 

### 스레드의 종류들

- 데몬 스레드와 비데몬 스레드로 나뉜다. 
- 데몬 스레드만 있으면 어플리케이션은 죽는다. 

## 스레드 덤프 생성

- 여기서는 스레드 덤프를 생성하는 방법 3가지 소개.
- 스레드 덤프는 덤프를 생성할 당시의 스레드 상태만 알 수 있다. 그래서 상태 변화를 알려면 5~10 초 주기로 덤프를 생성하는게 좋다. 

- 생성하는 방법 
  - jstack 이용
    - 1) `$ jps -v` 로 PID 를 알아낸다.
    - 2) `$ jstack [PID]` 를 통해서 스레드 덤프를 뜬다. 
  - visual vm 이용 
    - GUI 툴이라서 쉬움. 
  - kill 을 이용 
    - 1) `$ ps -ef | grep java` 로 PID 를 안다.
    - 2) `$ kill -3 [PID]` 를 통해서 스레드 덤프를 생성한다.

### 자바의 기본 스레드들 

스레드 덤프를 떴을 때 여러가지 스레드들을 볼 수 있는데 각 스레드의 정보는 어떤건지 파악하고 거를 수 있도록.

- Compiler Thread: JIT 컴파일러  
- Attach Listener: 모니터링 혹은 디버깅을 위해서 외부 접속을 대기하는 스레드
- Signal Dispatcher: 외부에서 JVM 프로세스로 전달되는 시그널을 처리하기 위한 스레드 
- Finalizer: 인스턴스들이 회수되기 전에 finalize() 함수를 호출하는 스레드 
- Reference Handler: GC 에 의해서 발견되는 weak, soft, phantom reference 들을 referenceQueue 에 추가하는 스레드. 
- main: main() 함수를 실행하는 스레드
- VM Thread: JVM 이 safe 한 상태가 되기를 기다리는 스레드. GC 나 스레드 덤프와 같은 작업을 하기전에 stop the world 상태를 기다리는 스레드.  
- GC Thread: 실제 GC 를 수행하는 스레드 
- VM Periodic Task Thread: 주기적으로 수행되는 실행들을 위한 타이머 이벤트를 담당한다.

## 스레드 덤프 정보 

````text
reactor-http-nio-9" #127 daemon prio=5 os_prio=31 cpu=26.86ms elapsed=3.29s tid=0x00007f9ddd94b800 nid=0x10f03 runnable  [0x000070000e8ac000]
   java.lang.Thread.State: RUNNABLE
        at sun.nio.ch.KQueue.poll(java.base@11.0.17/Native Method)
        at sun.nio.ch.KQueueSelectorImpl.doSelect(java.base@11.0.17/KQueueSelectorImpl.java:122)
        at sun.nio.ch.SelectorImpl.lockAndDoSelect(java.base@11.0.17/SelectorImpl.java:124)
        - locked <0x0000000623de4720> (a io.netty.channel.nio.SelectedSelectionKeySet)
        - locked <0x0000000623de44f0> (a sun.nio.ch.KQueueSelectorImpl)
        at sun.nio.ch.SelectorImpl.select(java.base@11.0.17/SelectorImpl.java:141)
        at io.netty.channel.nio.SelectedSelectionKeySetSelector.select(SelectedSelectionKeySetSelector.java:68)
        at io.netty.channel.nio.NioEventLoop.select(NioEventLoop.java:813)
        at io.netty.channel.nio.NioEventLoop.run(NioEventLoop.java:460)
        at io.netty.util.concurrent.SingleThreadEventExecutor$4.run(SingleThreadEventExecutor.java:997)
        at io.netty.util.internal.ThreadExecutorMap$2.run(ThreadExecutorMap.java:74)
        at io.netty.util.concurrent.FastThreadLocalRunnable.run(FastThreadLocalRunnable.java:30)
        at java.lang.Thread.run(java.base@11.0.17/Thread.java:829)
````

- (1) 스레드 이름: `reactor-http-nio-9`
  - java.lang.Thread 로 생성하면 Thread-(숫자) 로 생성되고 java.util.concurrent.ThreadFactory 클래스로 생성했으면 
  pool-(숫자)-thread-형식 으로 생성된다.

- (2) 우선순위: `prio=5 os_prio=31`

- (3) 스레드 아이디: `tid=0x00007f9ddd94b800 nid=0x10f03`
  - 이 정보를 바탕으로 해당 스레드의 CPU 와 메모리 사용량을 알 수 있다. 

- (4) 스레드 상태: `runnable`

- (5) 스레드 콜스택: 스레드의 콜스택 정보
 
## 스레드 덤프 유형들 

### 잠금을 소유하지 못해서 기다리는 경우. 이로 인해서 어플리케이션 전체적인 성능이 떨어지는 경우.

한 스레드가 monitor 객체를 소유중이다. `0x0000000780a000b0`

```text
BLOCKED_TEST pool-0-thread-1 prio=6 tid=0x00007f9fff94b800 nid=0x28f4 runnable
...
- locked <0x0000000780a000b0> (a com.nbp.theplatform.threaddump.ThreadBlockedState)
... 
```

다른 스레드들이 이 모니터 객체를 기다리는 상황

```text
BLOCKED_TEST pool-0-thread-2 prio=6 tid=0x0000766fff94b800 nid=0x260c waiting for monitor entry 
    java.lang.Thread.State: BLOCKED (on object monitor)
...
- waiting to lock <0x0000000780a000b0> (a com.nbp.theplatform.threaddump.ThreadBlockedState)
...
```

### 교착 상태인 경우 

- 위와 같은 상황이 꼬여있는 경우다. 스레드 1은 A 모니터 객체를 가지고 있으면서 B 모니터 객체를 위해 blocked 되어있고, 스레드 2는 B 모니터 객체를 소유하고 있으면서 C 모니터 객체를 기다리고 있다 그리고 스레드 3은 C 모니터 객체를 가지고 있으면서 A 모니터 객체를 위해 대기하고 있는 상황이다. 

### 원격 서버로부터 메시지를 수신받기 위해서 대기하고 있는 경우 

```text
"socketReadThread" prio=6 tid=0x0000766fff94b800 nid=0x260c runnable 
    java.lang.Thread.State: RUNNABLE
        at. java.net.SocketInputStream.socketRead0(Native Method)
        ...
```

- 이 경우에는 runnable 상태라서 괜찮다라고 생각할 수 있는데 socket 에서 데이터를 읽어오기 위해서 계속 실행되고 있는 상태다. 즉 대기중인 상태임. 

### WAIT 상태에 있는 경우 

스레드가 계속해서 WAIT 상태에 머물러 있는 경우를 말하는 것 .

```text
"ioWaitThread prio=6 tid=0x0000766fff94b800 nid=0x260c waiting on condiiton
    java.lang.Thread.State WAITING (parking) 
```

질문) 스레드의 WAIT 상태는 꽤 다양한 거 같다. 어떤 것들이 있는지?
- 먼저 `Object.wait()` 상태로 waiting 되고 있는 것.
- 다음으로 `Thread.join()` 으로 대기하고 있는 것. 
- 마지막으로 `LockSupport.park()` 으로 인해서 대기하고 있는 것. 
- Waiting 상태는 Blocking 상태랑 다르다. 

#### Object.wait()

```text
"WAITING-THREAD" #11 prio=5 os_prio=0 tid=0x000000001d6ff800 nid=0x544 in Object.wait() [0x000000001de4f000]
   java.lang.Thread.State: WAITING (on object monitor)
```

- monitor 객체를 가지고 실행되었지만 스케줄러에 의해서 다음 실행을 위해서 대기하고 있는 상태다.
  - 그러면 이떄는 monitor 객체를 반납한 상태인가? ㅇㅇ 반납한 상태다. 
- 다른 스레드가 notify(), notifyAll() 해줘야 꺠어날 수 있다.
- wait() 를 하는 경우는 다음과 같다. 
  - synchronized instance method 를 호출할 때 
  - synchronized block 의 body 를 실행하 때 
  - synchronized static method 를 실행할 때 

#### Thread.join()

````text
main" #1 prio=5 os_prio=31 cpu=400.54ms elapsed=15.15s tid=0x00007ff68b80bc00 nid=0x1703 in Object.wait()  [0x000070000be26000]
   java.lang.Thread.State: WAITING (on object monitor)
        at java.lang.Object.wait(java.base@17.0.5/Native Method)
        - waiting on <0x000000061e422aa0> (a study.SampleThread)
        at java.lang.Thread.join(java.base@17.0.5/Thread.java:1304)
````

- thread 에서 join() 메소드를 호출하면 호출한 스레드는 waiting 상태가 된다 reference thread 가 종료될 때까지.
- 즉 main thread 에서 다른 other thread 를 가지고 join() 메소드를 호출하면 이 스레드가 종료될 때까지 메인 스레드가 WAITING 상태가 된다. 
- join() 메소드를 호출한 스레드가 너무 오래동안 대기하지 않도록 timeout 을 주는 옵션도 있다. 
- join() 을 이용해서 happen-before relationship 을 만들 수 있다. 
  - 동기화 방법 중 하나. 
  - 한 스레드가 행동한 모든 액션의 수행을 join() 을 통해서 기다리고 해당 변화를 통해서 작업을 하는 것. 
  - t1 스레드가 t2 스레드의 join() 메소드를 호출했다면, t2 스레드가 모든 행동을하고 종료되면 t1 스레드는 변화를 관찰할 수 있는거지. 

- join() 내부적으로 while 문 안에서 wait() 를 호출한다. 그래서 최상위 스택에 wait() 가 있는 것. 


#### LockSupport.park()

```text
"PARKED-THREAD" #11 prio=5 os_prio=0 tid=0x000000001e226800 nid=0x43cc waiting on condition [0x000000001e95f000]
   java.lang.Thread.State: WAITING (parking)
```

- LockSupport class 에 있는 static method 인 park() 을 호출해서 WAITING 상태로 들어갈 수 있다.
- 이걸 호출하면 WAITING (parking) state 에 들어간다.
- LockSupport 클래스는 UnSafe 라는 클래스를 래핑한 클래스. 
  - UnSafe 는 internal java api 를 쓰기 떄문에 직접 접근할 수 없음.
- parking 과 waiting 을 비교해보면 parking 이 약간의 성능이 더좋다.
  - low level 의 os 코드로 parking 이 실행되니까 더 성능이 좋다.  
  - happens-before relationship 을 안가져도 되니까.
    - volatile variable 은 메모리에서 읽지만 이 값에 쓰기전에도 읽을 수 있다. 
    - happens-before 관계를 가지면 volatile variable 의 read 과정은 write 한 이후에만 읽도록 보장할 수 있다.


```java
public class Application {
    public static void main(String[] args) {
        Thread t = new Thread(() -> {
            int acc = 0;
            for (int i = 1; i <= 100; i++) {
                acc += i;
            }
            System.out.println("Work finished");
            LockSupport.park();
            System.out.println(acc);
        });
        t.setName("PARK-THREAD");
        t.start();
    }
}
```

- 그냥 park() 메소드 호출하면 된다. 
- parking 을 풀려면 unpark(thread) 메소드를 호출해야한다. 

```java
t.setName("PARK-THREAD");
t.start();

Thread.sleep(1000);
LockSupport.unpark(t);
```

### 스레드 리소스를 정상적으로 정리하지 못하는 경우 

- 불필요한 스레드가 계속 늘어나고 정리되지 않는 경우. 


## 스레드 덤프 분석을 위한 방법

내 질문은 언제 생성해야할까? 그런거였음. 


## 참고자료 

- https://www.baeldung.com/java-lang-thread-state-waiting-parking
- https://www.baeldung.com/java-analyze-thread-dumps
- https://www.baeldung.com/java-wait-notify
- https://www.baeldung.com/java-thread-join
- https://www.baeldung.com/java-semaphore
