## Generic

자바가 겪는 제네릭의 타입 시스템 문제를 코틀린은 겪지 않는다.

- 이 문제는 effective-java Item 31 에서 자세히 설명되어 있다.
- Item 31: *Use bounded wildcards to increase API flexibility*

자바의 제네릭의 타입 시스템은 *invariant* 하다. 이 말은 List<String> 은 List<Object> 의 서브 타입이 아니라는 뜻이다. 왜냐하면 List 자체가 Invariant 하니까.

만약에 List 가 Invariant 하지 않다면 무슨 일이 일어날까? 다음 자바 예를 보면 알 수 있다.

```java
// Java
List<String> strs = new ArrayList<String>();
List<Object> objs = strs; // !!! A compile-time error here saves us from a runtime exception later.
objs.add(1); // Put an Integer into a list of Strings
String s = strs.get(0); // !!! ClassCastException: Cannot cast Integer to String
```

- List 가 invariant 하지 않으므로 `List<Object>` 변수인 objs 에 `List<String>` 타입 변수인 strs 를 대입하는게 가능해질 것이다.

- 그리고 실제 런타임 시점에 String s = strs.get(0) 을 하면 ClassCastException 이 발생할 것이다.

하지만 이런 invariant 는 유연함을 제공해주지 않는다. 다음 예시를 보자.

```java
public class Stack<E> {
	public Stack(); 
	public void push(E e);
	public E pop(); 
	public boolean isEmpty(); 
}
```

```java
// 와일드카드 타입을 사용하지 않은 pushAll 메소드는 결함이 있다.
public void pushAll(Iterable<E> src) {
	for (E e : src) {
  	    push(e); 
	}
}
```

- 이 pushAll() 메소드는 컴파일 되지만 문제가 있다.
- Iterable src 의 원소 타입이 Stack 과 완벽하게 일치하면 잘 작동하지만 다음의 경우 문제가 된다.
    - `Stack<Number>` 로 선언하고 pushAll() 메소드에 Integer 타입을 넘기는 경우.
    - Integer 는 Number 의 하위 타입이니까 잘 작동해야 한다고 생각한다. (논리적으로는 문제 없다.)
    - 하지만 실제로는 오류 메시지가 뜰 것이다.

자바는 이런 상황을 해결하기 위해서 ***한정적 와일드 카드 타입  (? extends E)*** 이라는 걸 제공해준다. 위 예시를 해결하기 위해서 pushAll() 의 파라미터는 다음과 같이 변경될 것이다.

```java
public void pushAll(Iterable<? extends E> src {
	for (E e : src) {
	    push(e); 
	}
}
```

- ? extends E 뜻은 E 와 E 의 서브 타입만을 매개변수로 받을 수 있다. 즉 E 의 uppercase 타입은 받을 수 없다는 뜻이다.
- 이처럼 A 가 B 의 하위 타입일 때 `List<A>` 가 `List<B>` 의 하위 타입이 되는 것을 *Covariant* 라고 부른다.

자 이번에는 반대의 경우를 한번 보자. Stack 에서 pushAll() 과 짝을 이루는 popAll() 메소드를 작성할 차례다.

```java
public void popAll(Collection<E> dst) {
	while(!isEmpty()) {
	    dst.add(pop());
	}
}
```

- 이것도 dst 의 타입과 Stack 의 타입이 같다면 문제없이 잘 작동한다.
- 하지만 Stack<Integer> 의 요소를 Collection<Number> 로 옮길려고 하면 또 타입 변환에서 에러가 날 것이다. 왜냐하면 Java 의 제네릭 타입 시스템은 Invariant 하니까.

이번에도 와일드 타입을 사용해야한다. Stack 의 popAll() 메소드에 인자로 자신의 상위 타입은 허용해줘야한다. 즉 다음과 같이 될 것이다.

```java
public void popAll(Collection<? super E> dst {
	while(!isEmpty() {
	    dst.add(pop()); 
	}
}
```

- 이 처럼 A 가 B 의 상위 타입일 때 `Collection<A>` 가 `Collection<B>` 의 상위 타입인 경우를  *contravariance* 라고 한다.
- 이제 모두 문제 없다. 여기서 건네는 메시지는 이렇다. ***자바에서 제네릭을 사용하는 경우 유연성을 제공하고 싶다면 와일드 카드 타입을 적절하게 이용해야한다.***
- 와일드 카드의 사용 규칙은 ***Producer-Extends, Consumer-Super*** 이다.
    - E 입장에서 Input Parameter 를 읽어오는 연산에는 Producer 룰인 와일드 카드 ? extends E 를 사용하면 되고 Input Parameter 에게 객체를 전달하는 연산은 Consumer 룰인 와일드 카드 ? super E 를 사용하면 된다.

***

## Declaration-site variance

자바에서 다음과 같은 Source<T> 인터페이스가 있다고 보자. 그냥 단순히 자신의 타입 객체를 리턴해주는 메소드가 있다.

```java
// Java
interface Source<T> {
    T nextT();
} 
```

- 이런 타입의 경우에는 `Source<Object>` 에 `Source<String>` 을 넣어도 문제 없다.

하지만 자바에서는 다음과 같은 작업은 허용되지 않는다.

```java
// Java
void demo(Source<String> strs) {
    Source<Object> objects = strs; // !!! Not allowed in Java
    // ...
}
```

- 왜냐하면 자바에서 제네릭 타입 시스템은 Invariant 하니까.
- 이 작업을 하기 위해서는 `Source<Object>` 타입을 `Source<? extends Object>` 로 선언을 해줘야한다.
- 자바에선 이렇게 선언해주는 것도 문제다. Type safety 를 보장해주기 위해서 모든 메소드마다 `Source<? extends Object>` 로  만들어줘야한다.

코틀린에서는 이 문제를 ***Declaration-site variance*** 를 통해서 해결한다. 이걸 이용하면 클래스 선언부에 타입을 선언해 놓는 것만으로도 *invariant* 가 아닌 *Covariant* 와 *contravariance* 를 제공해준다.

- 즉 코틀린에선 매번 와일드 카드를 사용하지 않고 *Producer* 와 *Consumer* 의 역할을 사용할 수 있다.

바로 예시로 보자. 다음은 Declaration-site 를 Producer 로서 사용하는 예제다.

```kotlin
interface Source <out T> {
    fun nextT(): T
}

fun demo(strs: Source<String>) {
    val objects: Source<Any> = strs // This is OK, since T is an out-parameter
    // ...
}
```

- 여기서 Declation-site 에 out 으로 타입이 선언되어 있다면 Source <T> 는 안전하게 부모 타입에 대입하는게 가능하다.
- 즉 Source 클래스가 타입 T 의 Producer 로서 역할을 해줄 수 있다. Covariant 가 가능해진다.

다음은 Declartion-site 를 Consumer 로서 사용하는 예제다.

```kotlin
interface Comparable<in T> {
    operator fun compareTo(other: T): Int
}

fun demo(x: Comparable<Number>) {
    x.compareTo(1.0) // 1.0 has type Double, which is a subtype of Number
    // Thus, you can assign x to a variable of type Comparable<Double>
    val y: Comparable<Double> = x // OK!
}
```

- in 을 통해서 Comparable 클래스는 T 의 Consumer 로서 역할을 해줄 수 있다. Contravariance 가 가능해진다.

정리하면 틀린에서는 제네릭 타입 시스템에서 Declaration-site 를 통해 Contravaraicne 와 Covariant 를 제공해준다. 자바에서는 invariant 밖에 되지 않는데.

***

## Type projections

### **Use-site variance: type projections**

Use-site variance 는 Declartion-site 를 메소드 레벨로 줄인 것이다.

- Declaration-site 는 클래스 레벨에서 covariant 와 contravariance 를 제공해준 것이라면 Use-site variance 는 지정된 메소드에서만 이를 제공해준다.

바로 실제 예시로보자.

```kotlin
class Array<T>(val size: Int) {
    operator fun get(index: Int): T { ... }
    operator fun set(index: Int, value: T) { ... }
}
```

- Kotlin 에서 Array 는 Covariant 와 Contravariance 를 막아놨다. 그러므로 다른 서브 타입으로 변환되거나 하지 않는다.

이 경우에는 다음과 같은 copy() 작업에서 유연함을 제공해주지 못한다.

```kotlin
fun copy(from: Array<Any>, to: Array<Any>) {
    assert(from.size == to.size)
    for (i in from.indices)
        to[i] = from[i]
}
```

```kotlin
val ints: Array<Int> = arrayOf(1, 2, 3)
val any = Array<Any>(3) { "" }
copy(ints, any)
//   ^ type is Array<Int> but Array<Any> was expected
```

- Array<Int> 는 Array<Any> 로 변환되지 못하기 때문에.

하지만 다음과 같이 선언 해놓으면 딱 이 메소드의 경우에만 타입 변환 기능을 제공해줄 수 있다.

```kotlin
fun copy(from: Array<out Any>, to: Array<Any>) { ... }
```

- 이건 뭐 사실 자바에서 제공해주는 한정적 와일드 카드와 별반 다르지 않다.

### Star-projections

제네릭에서 아직은 어떤 타입이 올 지 모르는 경우, 하지만 타입의 관계는 명확히 정의해놓고 싶은 경우에 Star-projection 을 사용하면 된다.

바로 예시를 보자.

```kotlin
class Other

open class TUpper

class TChild : TUpper()

class Foo<out T : TUpper>
```

```kotlin
fun foo(bar: Foo<*>) {..}

fun demo() {
    foo(Foo<TChild>()) // compile success
    foo(Foo<Other>())  // compile error 
}
```

- star-projection 은 `*` 를 통해서 사용할 수 있다.
    - `*` 는 구체적인 타입이 정해지기 전에는 Any? 로 취급된다.
- function foo 를 보면 Foo 객체에서 어떠한 타입이 올지 몰라 `*` 를 선언해놨다. 하지만 그렇다고 해서 아무 타입의 Foo 객체를 다 받는 건 아니다. TUpper 의 하위 클래스만 들어올 수 있다.

다음은 Star-projection 만 이용한 경우다.

```kotlin
fun acceptStarList(list: ArrayList<*>) {
    list.add("문자열") // error!
}
```

- 이 경우에는 어떤 타입이 올 수 있는지 정해지지 않았기 떄문에 String 타입의 요소를 추가하는게 불가능하다.