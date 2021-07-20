## Java Core Interview

#### Wrapper 클래스 와 Primitive Type

기본적으로 Stack 에 저장된다. Primitive 는 Stack 에다가 값이 들어가지만 Wrapper 클래스는 Stack 에다가 들어가는 값은 참조할 주소가
들어간다. 

#### 접근 제어자에 대해서 설명해봐라 


#### Equals 와 HashCode 

HashCode 자체는 Equals 가 될 수 없다. 다른 객체라 하더라도 hashcode 는 같은 값이 될 수 있기 떄문에. 

#### JDK vs JRE vs JVM 의 차이에 대해서 말해봐라

JDK 는 자바 개발 도구 키트 

JRE 는 자바 바이트 코드를 실행시킬 수 있는 실행 환경을 갖추고 있는 것 JVM 을 갖추고 있다. 

JVM 은 Java Virtual Machine 으로 바이트코드가 실행될 메모리 공간을 말한다. 

JDK 는 개발 도구로서 소스 파일을 바이트 코드로 변환시킬 수 있는 컴파일러도 포함을 한다. 그리고
바이트 코드를 실행시킬 수 있는 JRE 도 포함하고 있다. 

#### Java String Pool 에 대해서 말해봐라 

Java String Pool 은 힙 메모리에 저장되어 있는 String 들이다. 이는 String Immutable 한 특성과
재사용성을 위해서 사용되는데 String 을 새로 생성하면 String Pool 에 이미 존재하는지 검색하고 
이미 존재한다면 같은 참조를 가지도록 한다. 

#### StringBuilder vs StringBuffer vs String 의 차이에 대해서 설명해봐라 

String 은 Immutable 하고 String Constant Pool 에서 관리한다.

StringBuilder 와 StringBuffer 는 mutable 하고 Heap 에서 관리한다.

StringBuilder 와 StringBuffer 의 차이는 synchronization 의 유무인데 StringBuffer 는
Thread-Safe 한 대신에 싱글 스레드 기준으로는 StringBuilder 가 더 빠르다. 

#### 자바의 클래스 로더에 대해서 설명해봐라.

자바의 클래스 로더의 역할은 JVM 이 요청하는 클래스 파일을 찾아서 메모리에 로드하는 역할을 한다.

주로 이 역할을 하는게 Java.lang.ClassLoader 의 loadClass 메소드를 통해서 한다. 

만약에 이 클래스로더가 없다면 Parent 에게 시킨다. 담당 책임자에게 묻는 구조.  

JRE 의 구성 중 하나디. 

그리고 클래스 로더는 총 3가지가 있다.

- Bootstrap ClassLoader

  - Java 클래스들은 Java.lang.ClassLoader 에 의해 로드 된다. 이 ClassLoader 도 클래스다. 
  얘도 로딩이 되야한다. 그 역할을 해주는게 Bootstrap ClassLoader 이고 JDK 내부에 있는 핵심 라이브러리들 즉 rt.jar 
  파일들이나 jre/lib 에 있는 파일들을 로드 해주는 역할을 한다.  
  
  - Bootstrap ClassLoader 는 다른 클래스 로더들의 Parent 같은 역할을 해준다. 
  
  - Bootstrap 클래스 로더는 Java 가 아닌 네이티브 코드로 작성되어 있기 떄문에 JVM 마다 다를 수 있다.  
  
- Extension ClassLoader

  - 자바 표준 코어 클래스의 extension 들을 메모리에 로드 하는 역할을 한다.
  
  - JDK extension directory 에 있는 파일들을 로드 하는 역할을 한다.  
  
- System/Application ClassLoader 

  - 이 클래스로더는 클래스 패스에 있는 내가 정의한 클래스들을 메모리에 적재하는 역할을 한다. 
  

#### Dynamic Method Dispatch 란? 

Dynamic Method Dispatch 를 통해서 자바의 런타임 시점에 다형성을 지원하는게 가능하다. 

이를 통해서 상위 클래스에서 런타임 시점에 오버라이딩 된 메소드를 호출하는게 가능해진다. 
 

#### Abstraction 이란? 

구현을 보여주는게 아니라 역할을 보여주는 것이다. 

#### Abstract Class vs Interface 의 차이는?

Instant 변수를 가질 수 있는지의 여부가 있다. 그 다음에 메소드의 접근 제어자의 차이가 있다.

인터페이스는 자바 8 에서 default 메소드와 static 메소드, 자바 9 에서 private 메소드가 들어왔지만

Abstract Class 는 Private Protected Default Public 메소드 모두 지원한다.  

#### 자바에서 private 메소든와 static 메소드를 오버라이딩 하는게 가능한가?

ㄴㄴ

#### 캡슐화랑 무엇인가?

데이터와 메소드를 하나의 클래스로 만들던지 해서 바깥으로부터 숨기는 걸 말한다. 

이를 통해서 책임을 가질 수 있다. 데이터의 변경 같은 것들을 


#### 자바 스트림이란? 

연속된 데이터 처리를 하기 위해서 들어온 기능으로 자바 8에서 들어왔다. 

기본적으로 처리할 때 데이터 소스를 변경시키지 않는다는 점. 

중개 오퍼레이션과 종료 오퍼레이션으로 구분된다는 점. 

중개 오퍼레이션은 스트림에서 스트림을 반환하고 종료 오퍼레이션이 없다면 Lazy 하다는 특징이 있고

종료 오퍼레이션은 스트림 처리를 완료한 후 결과를 반환을 말한다. 다른 타입으로 반환하는게 가능하다. 

#### 자바 Optional 이란? 

자바 8에서 들어온 타입으로 NULL 처리를 위해서 들어왔다. 

Optional 객체안에 다른 객체가 들어와있는 걸로 알 수 있다. 

Optional 을 사용할 땐 리턴타입으로만 사용하는걸 권장한다.

필드 타입에 Optional 이 있다면 설계가 잘못된 것. 값이 있을수도 있고 비어있을 수도 있다는데 클래스 필드가
그런 특징을 갖는다는게 말이 안된다.

메소드 매개변수로 Optional 을 받는것도 말이 안된다. Optional 을 써도 널 체킹 해줘야하는건 똑같다. 

그리고 Optional 로 Collection 을 감싸지 않아도 된다. Collection 자체가 값이 비어있음을 나타내는 메소드가 있기 때문에.

#### 자바 LocalDateTime 이란

기존의 Date/Time 메소드는 setTime() 메소드 같은게 있어서 변경할 수 있었다. 

자바 8 에서는 이게 마음에 들지 않아서 Immutable 한 값을 설정할 수 있도록 LocalDateTime 이 등장했다. 

#### 자바 CompletableFuture 이란 ?

기존에 사용한던 Future 로는 한계가 있었다. Future 끼리 조합하는게 안됐고 Future 의 결과를 콜백받기도 힘등었다.

예외처리도 할 수 없었고 이런 문제 때문에 CompletableFuture 이 등장했다. 

CompletableFuture 은 정상적인 흐름과 예외 처리의 흐름 둘 다 실행시킬 수 있고 콜백으로 결과를 받을 수도 있고 
Future 이 끝난 후 또 다른 비동기 메소드를 실행시키는게 가능하다. 

CompletableFuture 은 내부적ㅇ르로 Fork/Join 풀을 가지고 있다. 

#### 컴파일러가 인터프리터보다 빠른 이유는?

인터프리터는 매 순간마다 번역을 하지만 컴파일러는 그럴 필요가 없다. 컴파일러는 컨택스트를 이해하고 있기 떄문에
최적화 해놓았고 저수준의 기계어나 어셈블리어로 번역해놓았기 떄문에 더 빠르다. 

이런 최적화는 바이너리 문장의 순서와도 관련이 있다. 메모리에 값을 가져오라는 명령은 레지스터 내에서 값을 더하는것보다
느리다. 그러므로 매 순간순간마다 메모리에 가서 값을 가져오는 것보다 미리 가져와서 그걸 재활용할 수 있다면 성능 차이가난다.  
 
#### JIT Compiler 란? 

자바 바이트코드 실행이 어느 임계치를 넘으면 최적화 해서 기계어나 어셈블리어로 가지고 있도록 하는 컴파일러다. 

JIT 컴파일러의 핵심은 어플리케이션에서 자주 실행되는 부분을 얼마나 빠르게 실행하는지가 중요하다는게 핵심이다.

그래서 바이트코드를 여러번 실행을 해보면서 그 코드에 대해서 알 수 있는 정보가 많아지니까 메모리 접근을 줄이고
레지스터내에서 해결할 수 있다던지. 그런 정보를 바탕으로 최적화 해주는게 JIT 컴파일러다. 

#### 자주 사용하는 디자인 패턴은?

프록시 패턴과 전략 패턴

프록시 패턴은 객체를 감싸서 그 객체에 대한 Access Control 을 프록시를 통해서 해준다.

이를 통해서 Entity 같은 경우는 프록시 객체를 통해서 지연 로딩을 하기도 하고 다른 서비스로의 요청을
프록시를 통해서 함으로써 네트워크 연결 오류에 대한 처리같은 것들을 다 할 수도 있다.

그 뿐만이 아니라 프록시를 통해서 캐싱 처리를 할 수 있고 관심사를 나눠서 로그를 남기는 것도 할 수 있다.

전략 패턴은 목표가 있고 그걸 수행하는 전략이 다를 수 있는 경우에 사용하는데 주로 런타임 시점에 
다양한 알고리즘을 넣어야 햔다면 사용했다.

나 같은 경우는 전략 패턴을 사용할 떈 랜덤으로 수를 생성해야 하는 경우에 있어서 이 경우에 테스트 하기가 어려웠다.
그래서 수를 생성하는 걸 전략패턴을 이용해 전략으로 만들고 실제 프로덕션 코드는 랜덤 전략을 사용하도록 하고
테스트 코드에서는 상수를 생성하는 전략을 만들어서 이를 이용했다. 

   




