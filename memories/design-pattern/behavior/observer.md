# Observer

객체의 상태 변화를 감지하고 이 변화에 따라 반응해야 할 때 사용하는 패턴이다.

이를 통해 publish/subscribe 패턴을 구현할 수 있다.

- Subject 의 목적은 Observer 를 등록해놓고 해지하는 역할을 한다.
- 다수의 객체가 특정 객체 상태 변화를 감지하고 알림을 받는 패턴

### 장점

- 상태를 변경하는 객체와 상태 변경을 감지하는 객체간의 느슨하게 결합할 수 있다.
- Polling 하는 방식 말고 상태 변경을 감지할 수 있다.
- 런타임에 옵저버를 추가하거나 제거하는게 가능하다.

### 단점

- 역시 코드의 복잡도가 좀 증가한다.
- Observer 해지에 대해서 신경쓰지 않는다면 많은 메모리 누수가 발생할 수 있다.
    - Map 을 사용해서 객체를 등록할 땐 메모리 누수에 대해서 조심해야한다.
        - WeakReference 인지, SoftReference 인지에 따라서

### 실제로 사용하는 예

- Observer 라는 인터페이스와 그 구현체인 Observerable 이라는 클래스를 제공해준다. 물론 지금은 사용을 권장하지 않는다. 자바 9 부터
    - 그래서 java.beans 패키지에 있는 다른 인터페이스를 사용하는 걸 권장한다. Subscriber 의 역할로 PropertyChangeListener 와 이 Observer 역할을 해주는 PropertyChangeSupport 를 사용하면 된다.
- 두 번째로 자바 9 에서 리액티브 스트림을 구현할 수 있도록 도와주는 Flow API 가 있다.
    - Publisher, Subscriber, Subscription, Processor 가 있다.
        - Processor 는 Subscriber, Publisher 의 두 역할을 같이하는 것이다.
        - Publisher 가 메시지를 보내고 Subscriber 가 메시지를 받아서 처리하는 역할을 한다.
        - Subscription 을 통해서 저장된 메시지를 가지고 오도록 하는 비동기 처리를 도와준다.
- 스프링
    - ApplicationEventPublisher 를 통해서 이벤트를 발생시킬 수 있고 ApplicationListner 를 통해 이벤트를 받아서 처리할 수 있다. 그리고 ApplicationEvent 를 통해서 이벤트를 만들 수 있다.