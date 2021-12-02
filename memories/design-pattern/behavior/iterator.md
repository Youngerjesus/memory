# Iterator

집합 객체 안에 있는 내부 구조를 드러내지 않고 순회하는 패턴을 말한다. 프로그래밍 언어를 사용하는 개발자라면 익숙할 것.

### 대안

- 이 방법 말고 사용할 수 있는 방법으로, 내부 요소를 드러내는 for 문을 통해서 순회할 수도 있다.  클라이언트 입장에서 이 순회하는 집합의 타입이 변경되는 경우에는 클라이언트의 코드를 고쳐야하는 문제가 생길 수 있다.
    - Iterator 를 쓰면 집합의 자료구조를 숨길 수 있다.  또는 클라이언트의 설계에 따라서 순회의 기준을 숨길 수 있다.

### 장점

- 집합 객체가 어떠한 구조인지 몰라도 된다. Iterator 만으로 순회가 가능하다.

### 단점

- 클래스 구조가 복잡해진다.

### 실제로 사용하는 예

실제로 자바에서는 Iterator 를 제공해준다. 그리고 Iterator 이전에 Enumeration 을 통해서 Iterator 패턴을 제공했다.

- Java 에서 Iterator 를 자세하게 보면 remove() 메소드를 제공해주는데 이는 일반적인 컬렉션에서는 제공해주는 기능은 아니다. 그래서 UnsupportedOperationException() 예외가 발생한다. 물론 Concurrent 가 붙은 컬렉션들은 이를 이용할 수 있다.
- 또 다른 기능으로 forEachRemaining() 을 지원헤주는데 이는 while 문을 통해서 iterating 할 필요없이 순회할 수 있다. Functional Interface 인 Consumer 를 통해서.