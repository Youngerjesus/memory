# Adapter

***

110V -< 220V 로 꼽을 때 (변환이 필요한 경우에 사용하는게 어댑터)

- 클라이언트가 사용하는 인터페이스가 다를 때 변환한다는 것.
- 디테일하게 들어가면 클라이언트가 사용하는 인터페이스는 이미 만들어져있고, 내가 개발한 인터페이스도 이미 정해져있다. 이렇게 확정이 나있는 상황에서 클라이언트가 사용하고 있는 인터페이스로 변환해주고 싶을 때 사용한다.

### 장점

- 기존 클래스를 수정하지 않고 원하는 인터페이스로 만들 수 있다. 즉 기존 클래스가 하던 일은 하던 일대로 하고 변환하는 작업만 새로 만드는 것.

### 단점

- 새 클래스가 생겨서 복잡도가 증가할 수 있다. 경우에 따라서는 어댑터를 만드는 게 아니라 기존의 클래스가 클라이언트의 인터페이스에 맞추도록 변경하는게 더 나을수 있다.

### 실제 예

```java
List<String> list = Array.asList("a", "b","c"); // 가변 인자로 배열을 넘겼고 이게 리스트로 변환된다.

Enumeration<String> enumeration = Collection.enumeration(lists); // List 를 넘기면 Enumeration 으로 변환해준다. 
```

```java
try (InputStream is = new FileInputStream("input.txt");
			InputStreamReader isr = new InputStreamReader(is); 
			BufferReader reader = new BufferReader(isr)) {
	
	while(reader.ready() {
		System.out.println(reader.readLine()); 
	}
} catch (IOException e) {
	throew new RuntimeException(e); 
}
```

- 여기서 변환하는 부분은 다음과 같다.
- 처음 문자열을 넣으면 InputStream 이 나오고
- InputStream 을 넣으니 InputStreamReader 가 나온다. 그리고 InputStreamReader 를 넣으니 BufferReader 가 마지막으로 나온다.

```java
HandlerAdapter ha = new HandlerAdapter(); 
```

- Spring MVC 에서 HandlerAdapter 가 어댑터 패턴을 이용한 예
- Spring MVC 에서 요청을 처리하는 Handler 는 여러가지 형태로 작성할 수 있다.
    - 그 중 대표적인 경우가 Bean 이름을 기반으로 Handler 를 작성하는게 가능하고
    - @RequestMapping 을 이용한 컨트롤러를 기반으로 Handler 를 작성하는 것도 가능하다.
    - 이런 다양한 핸들러로 확장할 수 있고 이것들을 처리하기 위해서는 DispatcherServlet 이 원하는 형태의 응답을 만들어줘야하는데 이것들을 해주는게 HandlerAdapter 이다.
