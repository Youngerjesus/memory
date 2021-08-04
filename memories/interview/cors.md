## CORS (Cross Origin Resource Sharing)

교차 출처 자원 공유

#### Preflight Request

HTTP OPTION 으로 preflight 요청을 보내서 내가 어떤 요청을 할건지에 대한 정보와 나의 ORIGIN 을 HEADER 에 실어서 보내면 
서버가 이 요청을 보고 허용해주면 응답으로 Access-Control_Allow-Origin 에 이 출처를 허용 가능하게 해주면 CORS 정책에 위반되지 않고 
서로 요청할 수 있게 해주는 방식이다. 

#### Simple Request 

Preflight 같이 예비 요청을 보내는게 아니라 바로 Request 를 날리는건데 제한사항이 좀 있다.
HTTP METHOD 에도 제한사항이 있고 Header 에 진짜 기본적인 조건의 헤더만 사용이 가능하고 와 Content Type 에도 application/json 은 포함하면 안되고 다른 제한사항이 있다.

 
#### Credential Request

좀 더 보안적인 요청을 할 때 사용하는 방식