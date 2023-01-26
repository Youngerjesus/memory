# 메모리 진단하기  
    
## 학습 목표

- JVM 내의 메모리 문제는 어떤 것들이 있는지. 
- 해당 문제를 해결하는 방법들 

## 배경지식 

JVM 내에서 존재하는 메모리 영역은 다음과 같다. 

- PC 레지스터
  - 각 스레드의 JVM 인스터럭션 주소가 담겨있다. 
  - 인스트럭션은 바이트 코드라고 생각해도 됨.
- JVM 스택
  - 스레드가 생성될 떄 스택도 같이 생성된다. 
  - 지역 변수와 부분 결과 그리고 메소드 호출 및 리턴과 같은 정보가 저장된다.
- 힙 
  - 모든 클래스의 인스턴스들이 여기에 담긴다. 
  - GC 가 관리하는 영역 
  - 힙 영역은 eden, survivor, tenured, permanent 로 나뉜다. 
    - eden, survivor 는 young 영역 
    - tenured 는 old 영역이다. 
    - permanent 는 java 8 이후에 사라졌다고 본 것 같기도하고 ㅇㅇ 맞음 이거 사라지고 metaspace 가 생김.
    - metaspace 영역은 클래스 로더가 로더한 정보의 metadata 가 저장된다. 그리고 이건 heap 영역이 아닌 native 메모리 영역임.
      - 이걸로 인해서 JVM 이 관리안함. OS 가 관리한다. 상한 없이 늘어날 수 있다. 그래서 힙 뿐만 아니라 MaxMetaspace 값도 결정해주는 게 좋다.
      - metaspace 와 method area 의 차이는 뭐지? 
        - method area 는 permanent 영역 중 하나였다. 지금은 metaspace 로 넘어감
- 메소드 영역
  - 모든 JVM 스레드들이 공유한다. 각 클래스의 정보들을 저장하는 영역 
  - 더 자세하게 말하면 런타임 상수 풀, 필드, 메소드 데이터, 메소드와 생성자 코드, 클래스와 인터페이스 인스턴스의 초기화를 위한 특수 메소드 들에 대한 정보가 있다.
- 런타임 상수 풀 
  - 각각의 클래스나 인터페이스에 대한 여러 종류의 상수들이 저장되어 있는 곳.  
- 네이티브 메소드 스택 
  - 자바 언어 이외의 네이티브 메소드를 호출할 경우 타 언어의 스택 정보를 여기에다가 저장된다.
  - JNI 라고 불리기도 한다.
  - 이것도 스레드 별로 만들어진다.
    - stack 의 사이즈가 고정될 수도 있고, dynamic 하게 늘어날 수 있다.
      - 고정된 사이즈의 경우 스레드에서 한계보다 더 많이 요구하게 된다면 StackOverflowError 가 날 수 있다.
      - dynamic 하게 만드는 경우에는 무한하게 stack 이 늘어날 수 있다. 근데 thread 를 만들 때 swap 공간을 포함해서 메모리를 할당하는 공간이 부족하다면 OOM 이 날 수 있다.  
  - 여기에 있는 영역이 OOM 의 원인이 될 수 있는건가?
    - ㅇㅇ  

- 크게는 jvm heap 과 off-heap (= native memory) 영역으로 분리되어 있다고 생각하면 된다. 
  - native memory 에 metaspace 가 들어가있다. 

## OOM 이 발생하는 경우 

일반적으로 OOM (OutOfMemoryError) 가 발생하는 경우는 다음과 같다. 
- GC 가 새로운 객체를 할당한 공간이 없다고 판단하고 더 이상 힙메모리가 증가할 수 없다고 판단했을 때.
- 네이티브 라이브러리를 쓰고 코드에서 swap 영역이 부족해서 네이티브를 메모리에 할당할 수 없을 때. 
  - JNI 와 같은 네이티브 라이브러리르 호출한 경우에는 발생할 수 있다. 

여기서는 네이티브 영역이 아닌 경우에서만 다룸. 

OOM 이 났을 때 시스템 로그를 잘봐야한다. 

```text
Exception in thread "main" java.lang.OutOfMemoryError: java heap space 

Exception in thread "main" java.lang.OutOfMemoryError: Metaspace 

Exception in thread "main" java.lang.OutOfMemoryError: Requested array size exceed VM limit

Exception in thread "main" java.lang.OutOfMemoryError: request <size> bytes for <reason> Out of swap space

Exception in thread "main" java.lang.OutOfMemoryError: <reason> <stacktrace> <Native Method>  
```

#### Exception in thread "main" java.lang.OutOfMemoryError: java heap space 

- 힙 영역에 객체를 더 이상 생성하기 어려울 때 발생한다.  

- 여러가지 경우에 생길 수 있다. 
  - 메모리의 크기를 애초에 너무 잡게 잡아 놓은 경우 
    - `-Xms` 는 JVM 내의 최소 메모리 크기로 잡아놓는 설정이고, `-Xmx` 는 JVM 내의 최대 메모리 크기로 잡아놓는 설정이다. 
    - 최대 메모리가 너무 작은 경우에 발생할 수 있다.
  
  - 오래된 객체들이 계속 참조 되고 있어서 GC 가 안되는 경우
    - static 을 잘못 사용하거나, 오래된 객체들을 계속 참조해서 GC 가 안되는 경우에 메모리 릭이 발생해서 OOM 이 발생할 수 있다.
  
  - finalize 메소드를 개발자가 클래스에 구현한 경우 
    - GC 가 되야하는 애들은 큐에 쌓여있다. 그리고 이를 처리하는 데몬 스레드가 있다. 
    - GC 처리 전에 finalize 가 되야할 애들이 큐에 쌓여있는데 너무 많이 쌓여있고, 이게 처리도 불가능한 경우에는 발생할 수 있다. 
    - 주로 개발자가 이를 구현하고 별도의 처리를 해놓는 경우에 잘못 해놓으면 발생할 수 있다.
    - finalize 는 java 9 이후에 Object.finalize() 를 사용할 수 없다. 
    - (너무 많이 쌓여있어서 청소가 안된다는 건가)
  
  - 스레드 우선순위를 너무 높인 경우 
    - GC 를 처리하는 스레드보다 우선순위가 너무 높다면 쌓이는 속도가 더 빨라서 문제가 생길 수 있다.
  
  - 너무 큰 덩어리의 메모리를 가진 객체가 생겨나는 경우

- java.lang.OutOfMemoryError: GC Overhead Limit Exceeded 와 같은 경우도 있다. 
  - GC 를 자주하고 했는데 메모리가 해제가 아주 적게 되는 경우 

#### Exception in thread "main" java.lang.OutOfMemoryError: Metaspace 

- Java 7 이전에서는 permanent 영역이 있었지만 8 이후에는 사라져서 metaspace 에러가 난다. 
- 너무 많은 클래스를 로딩하는 경우에 이 에러가 발생할 수 있다.
- 익명클래스를 다량 생산하거나, 3rd party class 에서 다량의 클래스를 생산하는 경우.
- 관련 옵션으로 `-XX:MaxMetaspaceSize=128m` 와 같은 옵션을 쓰면 된다. 

#### Exception in thread "main" java.lang.OutOfMemoryError: Requested array size exceed VM limit

- 배열의 크기가 힙 영역의 크기보다 더 큰 경우에 생긴다. 
- 배열의 크기를 계산된 변수로 지정하는 경우에 이런 문제가 생길 수 있다.

#### Exception in thread "main" java.lang.OutOfMemoryError: request <size> bytes for <reason> Out of swap space

- native 메모리 영역이 부족한 경우에 샌긴다. 즉 OS 의 메모리의 영역에서 swap 까지 부족한 경우에 생길 수 있다. 
- 자바 어플리케이션에서 네이티브 메소드 쪽에서 메모리를 반환하지 않거나 다른 어플리케이션에서 메모리를 반환하지 않는 경우에 생긴다. 
- 이 메시지가 나타나면 JVM 에서 fatal error log 를 발생시킨다. 

#### Exception in thread "main" java.lang.OutOfMemoryError: <reason> <stacktrace> <Native Method>

- 위의 예시와 비슷하다. 네이티브 메모리 영역에 메모리를 할당할 때 발생한다. 
- JNI 나 네이티브 코드에 의해서 발생하는 경우가 많다. 

## 메모리 릭의 세 종류 

- 수평적 메모리 릭
  - 하나의 객체가 여러 메모리의 참조를 해놓고 놔주지 않는 경우
  - ArrayList 와 같은 예 
- 수직적 메모리 릭 
  - 하나의 객체가 다른 객체를 참조하고 이 객체가 또 다른 객체를 참조하는 경우 
  - LinkedList 와 같은 예
- 대각선 메모리 릭 
  - 이 둘을 합친 예시

## OOM 말고 다른 메모리 문제

시스템이 크래쉬 나는 경우가 있다. 크래시 원인이 메모리 때문은 아닐 수 있다. 

보통 네이티브 메모리에 메모리 할당이 실패하면 생긴다. malloc 이라는 시스템 콜로 메모리를 할당하는데 메모리가 없어서 null 로 온 경우.

크래쉬가 나면 `hs_err_pid` 로 시작되는 덤프 파일이 생기는데 이걸 통해서 분석할 수도 있다.

그리고 이것말고 또 다른 문제는 잦은 GC 를 생각할 수 있다. 

- GC 튜닝은 나중에 하는 것. 잘못쓰고 있는 메모리가 있기 때문에 잦은 GC 가 일어나는 것이다. 이것들을 먼저 해결하자. 

메모리 분석 툴 
- jstat 을 쓰면 각 영역별로 메모리를 얼마나 쓰는지 볼 수 있다.

## 대략적인 메모리 leak 분석 과정 

- 메모리 leak 이 발생한다는 사실부터 인식해야한다. 
  - full GC 이후에 메모리 사용량이 점점 증가하는지를 보면 된다. 
  - old 영역의 증가를 메모리 릭이라고 잘못알고 있는 경우가 있는데 원래 증가할 수 있다.
  - 메모리 leak 이 있다는 건 해제되지 않은 메모리가 있다는 것.
  - 이전 full gc 와 다음 full gc 를 비교해봤을 때 메모리가 증가해있다면 메모리 릭
  - full gc 와 해당 메모리 사용량들을 볼려면 jstat 을 이용하면 된다. 

- jmap 으로 메모리 단면을 자른다. heap dump 생성.

- MAT 이나 IBM Heap Analyzer 로 Heap dump 를 분석한다. 
  - 어떤 객체가 가장 메모리에서 차지하는 비율이 높은지 
  - 실제로 메모리를 점유하고 있는 객체가 누구인지. 최상단의 객체를 보기. Top Consumer 와 같이 클래스, 패키지 등의 각종 그룹에 따라서 많은 메모리를 점유하고 있는 클래스를 볼 수도 있다.  


## fatal error log 분석

- JVM Crash 가 발생했을 떄 에러 파일을 남기는 방식 
  - `-XX:ErrorFile=/var/log/java_error%p.log`

주로 남겨지는 정보는 시그널 정보, 프로세스 정보, 시스템 정보, 스레드 정보가 남겨진다. 
