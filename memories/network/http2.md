## HTTP 2 

***

#### HTTP 1.0

HTTP 1.0 에서는 요청 하나당 연결 하나를 지향했기 때문에 성능이 안나왔다.

#### HTTP 1.1 

HTTP 1.0 에서는 Persistent Connection 이라는 걸 지원해서 지정된 시간동안 커넥션을 닫지않는 방식을 이용했다.

- 이는 Keep Alive 옵션을 통해서 사용이 가능하다. 

그리고 Pipelining 을 지원한다.  

- 하나의 커넥션 안에서 요청을 여러개 보낼때 요청 응답을 받고 보내는게 아니라 여러 요청을 동시에 보내고 순서대로 응답을 받는 방식을 말한다. 

하지만 Pipelining 의 문제는 Head Of Line Blocking 이 생긴다. 

- 첫번째 요청이 굉장히 시간이 오래걸리는 요청이라면 나머지 요청도 블락이 되는 문제다. 

- 그리고 연속된 요청의 경우에 헤더가 중복되는 문제가 생긴다.

#### HTTP 2 

HTTP 1.X 버전에서 성능 향상을 위해서 나온 프로토콜이다.

기존의 HTTP 1.X 버전과 완전히 호환된다.  

HTTP 2 에서는 메시지 전송 방식이 변화되었다. 

Binary framing layer 라고 해서 기존에 문자열로 전송된 헤더와 데이터를 바이너리 프레임으로 변환해서 전송한다.
보내는 데이터는 Header Frame 과 Data Frame 으로 나눠져서 보내질 수 있다. 바이너리로 인코딩되서 보내지므로 더 작은 데이터가 보내지는 장점이 있다.  

또 Stream 을 이용해 클라이언트-서버는 양방향으로 데이터를 보낼 수 있고 한 TCP 연 Stream 에서 여러개의 메시지를 보내는 것도 가능해졌다. 프결
즉 TCP 커넥션을 여러개 맺을 필요도 없어졌다. 

Stream 의 또 장점으로 HTTP 1.X 에서는 Multiple TCP Connection 을 맺어서 한번에 여러 요청을 보내고
순서대로 응답을 받는게 가능했다. 하지만 이 방법의 문제는 첫번째 요청이 시간이 걸리면 그만큼 대기하게되는
Head of line Blocking 이라는 문제가 생겼는데 Stream 에서는 데이터를 Frame 으로 전송하고 Interleaving 을 지원하며
클라이언트와-서버는 받은 Frame 을 재조립하면 되므로 이런 Head of Line Blocking 문제를 해결했다.  
즉 Multiplexing 을 이용해 문제를 해결했다. 

또 다른 HTTP2 의 기능으로는 다음과 같다. 

Stream Prioritization

- 리소스간 우선 순위를 설정 가능하다. 

Server Push 

- 클라이언트가 요청하지 않은 데이터를 서버쪽에서 너 이거 필요할거 같애 하고 보내주는게 가능하다. 

Header Compression

- 연속적인 비슷한 요청이라면 헤더를 압축해서 필요한 부분만 보내도록 한다.  
 