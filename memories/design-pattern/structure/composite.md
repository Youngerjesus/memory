# Composite

***

객체와 이 객체들이 계층 구조로 이뤄진 객체들을 클라이언트의 입장에선 동일하게 느끼도록 하는 패턴.

- 클라이언트는 이게 전체의 요소인지, 전체 중 하나인지 알지 못하고 사용한다.
- 이 패턴을 이용할 땐 트리 구조로 이뤄진다.
- 클라이언트는 Component 의 인터페이스만 바라본다. 즉 결합을 낮출 수 있다.

### 장점

- 트리 구조를 이용하는 경우 좀 더 편하게 사용하는게 가능하다.
- 클라이언트와의 결합을 낮출 수 있다. 이로인해 새로운 타입을 추가하는, 확장에 열려있다.

### 단점

- 공통된 인터페이스를 뽑기 위해 지나치게 일반화를 할 수 있다.  (디자인 패턴을 위해서 코드를 작성하면 안된다.)

### 실제로 사용하는 예

```java
JFrame frame = new JFrame(); 
JTextField textField = new JTextField(); 
JButton button = new JButton(); 

frame.add(textField); 
frame.add(button); 

frame.setVisible(true); // 모든 구성요소가 다 보임.
```

- javax.swing 에 있는 클래스들인 JFrame, JTextField, JButton 모두 Component 인터페이스를 상속받은 구조. Component 인터페이스는 setVisible() 메소드를 가지고 있고 JFrame.setVisible() 메소드를 통해서 안에 있는 모든 요소들이 다 이 메소드를 호출한다.