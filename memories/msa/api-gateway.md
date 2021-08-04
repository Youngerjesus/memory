## API Gateway 

모든 API 들이 단일 지점인 API Gateway 를 통과해서 지나가도록 하는 것 

멘 처음 요청을 받아서 처리핸주는 스프링 MVC 의 DispatcherServlet 과 유사한 느낌도 있는 것 같다. 

요청의 공통적인 처리를 해준다는 것.

- 요청의 로그를 남기는 것. 

- 인증과 인가 처리

- 클라이언트 마다 다르게 API 를 처리해줄 수 있는 BFF(Backend for Frontend) 패턴 적용

  - 프론트엔드 유형에 따라서 API 를 나눠서 줄 수 있는 것. 모바일만을 위한 API, 웹만을 위한 API  

- HTTP Request URI 를 보고 어떤 서비스에 대한 요청인지 서비스를 찾아주는 Service Discovery 패턴

- 찾은 서비스가 여러개의 인스턴스가 있다면 로드밸런싱을 해줄 수 있는 것. 
 