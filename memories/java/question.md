## Collection.stream().forEach() vs Collection.forEach()

https://www.baeldung.com/java-collection-stream-foreach

이 둘은 Processing Order 에서 차이가 있다.

Parallel 프로그래밍에서 차이가 나는데 Collection.stream().forEach() 는 처리해야하는 순서가 정의되어 있지 않아서

요소중에 뭐가 먼저 실행될 지 알 수 없다. 

하지만 Collection.forEach() 는 순서대로 실행을한다. __(이건 병렬 프로그래밍에 열려있지 않구나.)__

Collection.forEach() 같은 경우는 다른 Iterator 방식을 지정하는 것도 가능하다. __(거꾸로 실행하는 Iterator 를 만들어서 넣는 것도 가능하다.)__

하지만 Collection.stream().forEach() 는 Iterator 가 처리하는 방식이 Undefined 로 되어있다. 

정리하자면 stream.forEach() 는 병렬처리에 용이하게 되어있다는 뜻