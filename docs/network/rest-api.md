## REST API 

REST 아키텍처 스타일을 따르는 API 

#### REST API 3단계

HATEOAS (Hypermedia As The Engine Of Application State) 라고 하는 
요청의 응답으로 결과와 함께


#### REST 를 구성하는 스타일  

제일 안 지켜지는게 __Uniform Interface__ 다. 

Uniform Interface

- identification of resources (리소스는 URI 로 식별된다 OK)

- manipulation of resources through representations (HTTP METHOD 를 통해서 리소스의 동작을 제어하는 것)

- self-descriptive messages (안 지켜지는 애 1)

  - 정의는 메시지는 스스로 설명되어야 한다.   
  
  - 전달받은 메시지가 무슨 의미인지 알 수 있어야 한다. 
  
    - 이는 content-type 으로 데이터를 설명인지 알 수 있어야 한다.   

- hypermedia as the engine of application state (안 지켜지는 애 2)

  - 정의는 애플리케이션의 상태는 Hyperlink 를 이용해 전이되어야 한다. 라는 것이다. 
  
    - 게시판을 기준으로 게시판의 글 목록을 보는 것, 글 쓰기 링크를 가져오는 것, 글 저장을 하는 것, 생성된 글을 보는 것 
    이런 상태를 전이하는 걸 HyperLink 를 볼 수 있어야 한다.  
    
    - HTML 인 경우 OK
    
- JSON 으로 전달하는 경우에는 이를 만족하지 못한다.

  - HyperLink 가 없고 JSON 데이터가 무슨 의미인지 알지 못한다. 
  
  - 해결하는 방법은 Content-Type 을 새로 정의하고 IANA 에 등록하는 방법이 있다.
  
  - 데이터를 설명하는 문서의 링크와, 데이터의 상태를 전이시킬 수 있는 문서의 링크를 링크 헤더에 같이 실어서 보내는 방법과
  Response Body 에 이 링크를 넣는 방법이 있다.  
