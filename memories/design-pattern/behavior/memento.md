# Memento

객체 내부의 상태를 외부에 저장해놓고 이후 필요하면 복원을 하는 방식을 이용하는 패턴이다.

- 외부에 저장을 하지만 캡슐화를 통해 외부에서 보이도록 하면 안된다. 클라이언트가 이 정보를 알아서 강하게 결합이 되면 안된다.
- 에디터 예로 보면 이해하기 쉽다. ctrl + z 키를 누르면 undo 연산을 하는데 이를 통해서 복원하는게 가능하다.
- 또 다른 예로 게임을 중지시키고 다시 재개하는게 가능한데 이것도 외부에 상태를 저장해놓고 복원하는 패턴이다.
- CareTaker 가 Originator 의 상태를 Memento 에 저장하고 이 정보를 가지고 복원 역할을 해준다.
    - 그래서 Originator 에서는 내부의 정보를 캡슐화해서 Memento 에게 전달해주는 메소드가 필요하고 Memento 로 부터 저장된 정보를 다시 복원하는 restore() 메소드가 필요하다.
    - Memento  의 경우 값이 변경되면 안되니까 저장하는 데이터는 final 로 선언해서 변경되지 않도록 하고 클래스도 그냥 Originator 의 데이터를 저장할 목적이니, 상속할 필요가 없으니까 fnal 을 붙인다.

### 장점

### 단점

### 실제로 사용하는 예

```java
Game game = new Game(); 

game.setScore(10);

// 직렬화 
try(FileOutputStream fileOutput = new FileOutputStream("GameSave.hex");
		ObjectOutputStream out = new ObjectOutputStream(fileOutput))
{
	out.writeObject(game); 
}

game.setScore(15); 

// 역직렬화
try(FileInputStream fileInput = new FileInputStream("GameSave.hex");
		ObjectInputStream in = new ObjectInputStream(fileInput)) 
{
	game = (Game) in.readObject();
	System.out.println(game.getScore(); 
}

```

- 직렬화를 통해서 객체를 저장하고 역직렬화를 통해서 객체를 복원하는 예제
- 이렇게하면 ByreStream 이 CareTaker 역할을 해주고 Originator 가 직렬화를 지원해야 한다.