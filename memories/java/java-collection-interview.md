## Java Collection Interview

#### Java 의 Collection 구조도와 Collection 은?  

3가지 타입의 Collection 이 있다. 

- ordered lists

  - 순서가 보장된 걸 말하며 List, Queue, Deque Interface 가 있다. 
  
  - ArrayList, LinkedList
  
  - Stack 
  
  - ArrayDeque, PriorityQueue, LinkedList, BlockingQueue

- dictionaries/map

  - key-value 를 통해 값을 찾는 걸 말하며 map 인터페이스가 있다.
  
  - HashMap, LinkedHashMap, TreeMap, NavigableMap, SoredMap

- sets

  - 순서가 없으며 중복을 포함하지 않는 걸 말하며 set 인터페이스가 있다. 
  
  - HashSet, TreeSet, LinkedHashSet, SortedSet, NavigableSet

#### 4) What is the difference between ArrayList and Vector?

ArrayList 는 Not Synchronization 이지만 Vector 는 Synchronization 으로 Thread-safe 이다.

ArrayList 와 Vector 모두 둘 다 동적 배열인데 ArrayList 같은 경우는 가득차면 Array 의 절반만큼 사이즈가 늘어나지만 Vector 같은 경우는 두배로 늘어난다 

#### 5) What is the difference between ArrayList and LinkedList?

사용하는 자료구조부터 다르다. ArrayList 는 동적 배열을 사용하지만 LinkedList 는 Double-Linked List 를 사용한다.

ArrayList 는 데이터를 저장하고 가져오는데 빠르다. 아무래도 인덱스를 이용하니까

LinkedList 는 데이터 조작에 더 쉽다. 중간 요소에 값을 넣거나 중간 요소를 지우는 것이 쉽다. 

#### 6) What is the difference between Iterator and ListIterator?

방향부터 좀 다르다. Iterator 는 직진할수만 있지만 ListIterator 는 앞뒤로 움직일 수 있다.
 
할 수 있는 행동도 좀 다른데 Iterator 는 Collection 을 순회하면서 지우는 것만 가능하다면 
ListIterator 는 더하고 지우는 것 가능하다. 

ListIterator 같은 경우는 List 형태만 지원하지만 Iterator 는 Set 이나 List, Queue 모두 지원한다. 

#### ConcurrentHashMap vs HashMap 이란?  

Thread Safe 부터 좀 다르다. ConcurrentHashMap 은 Thread-Safe 하지만 HashMap 은 Thread-Safe 하지 않다. 

Collection 같은 경우에 순회를 하면서 수정을 하면 ConcurrentModificationException 이 일어난다. 

ConcurrentHashMap 은 이런 예외가 일어나지 않는다.  

#### ConcurrentHashMap vs HashTable vs SynchronizedMap 이란? 

둘 다 스레드 세이프 하다는 특징이 있지만 한번에 업데이트 할 수 있는 개수가 다르다. 

ConcurrentHashMap 은 기본적으로 락의 개수가 16개로 한번에 16개의 값을 변경하는게 가능하다. 이것보다범위로 변경을 한다면 ConcurrentModificationException 이 발생한다. 

HashTable 과 SynchronizedMap 은 둘 다 값을 업데이트 하기위해선 전체 Map 에 대해서 락을 걸지만

ConcurrentHashMap 은 Bucket 마다 락을 걸 수 있다. 

ConcurrentHashMap 은 멀티스레드 환경에서 사용하는게 가능하지만 HashTable 과 SynchronizedMap 은  
한번에 한 스레드만 이용하는게 가능하다. (Read 도 포함이다.)

ConcurrentHashMap 은 한 스레드가 순회하면서 다른 스레드가 변경해도 ConcurrentModificationException 이
발생하지 않지만 SynchronizedMap 과 HashTable 은 ConcurrentModificationException 이 발생한다.  

ConcurrentHashMap 과 HashTable 은 null 을 허용하지 않지만 SynchronizedMap 은 널 값이 들어갈 수 있다. 

#### HashTable 과 HashMap

스레드 세이프의 유무에서 차이가 일단 난다. HashMap 은 스레드세이프 하지 않고 HashTable 은 스레드세이프하다. 

HashMap 은 Null 을 포함하지만 HashTable 에 key-value 값으로 null 을 넣으면 Exception 이 발생한다. 

 
#### HashMap 과 TreeMap 그리고 LinkedHashMap 의 차이는? 

HashMap 과 LinkedHashMap 은 조회와 삽입시 O(1) 이다. 추가로 LinkedHashMap 은 Double Linked List 를
통해서 입력한 순서를 보장해준다. 하지만 TreeMap 은 조회와 삽입시에 O(logN) 이 걸린다. 대신에 키 값이 정렬되어 있다. 

#### HashSet 과 HashMap  
 
HashSet 은 내부적으로 HashMap 을 이용하고 값을 저장한다. HashSet 을 이용하기 위해서는
equals() 와 hashCode() 를 구현해야 하는데 이를 이용해 중복을 넣었는지 탐색한다. 

#### HashMap 의 내부 동작 

key 값을 hashCode() 메소드에 넣어서 들어갈 버킷을 정해준다. 해시코드가 충돌할 경우에 LinkedList 로 값을 이어서
충돌을 해결한다. 

LinkedList 로 충돌을 해결했으니 실제로 값을 가지고올때는 equals() 메소드를 통해서 실제로 이 값이 맞는지 확인을 하고 가져온다. 
 

   

