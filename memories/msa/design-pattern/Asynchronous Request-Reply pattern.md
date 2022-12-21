# Asynchronous Request-Reply pattern

https://learn.microsoft.com/en-us/azure/architecture/patterns/async-request-reply

***

## 이 패턴을 쓰는 이유 

- backend 와 client 를 decoupling 시키고 싶을 떄. 
  - 몇 초를 넘나드는 오래걸리는 요청은 동기식에는 적합하지 않다. 
    - 어떤 아키텍처는 message broker 를 이용해서 요청과 응답을 분리해서 해결한다. 
    - 이건 `Queue-Based Load Leveling pattern` 이라고도 한다. 
      - 대신에 성공했을 때 응답을 받는 어려움이 존재.
      - 서버를 독립적으로 확장할 수 있다는 장점이 있음. 

## 이 패턴 적용법  

- 여기서 해결 방법은 HTTP Polling 이다. 
  - 처음 요청을 하면 Server 는 그 요청을 처리하는 다른 프로세스에게 보내고, 응답으로 202 (Accepted) 를 내보낸다. 
  - 그 다음 status 를 조회하는 API 에게 polling 을 한다. 
  - status 가 완료되었다면 해당 리소스를 받거나, 302 (Found) 를 받아서 redirect 될 수 있다.
    - 302 응답 코드를 받으면 해당 헤더에 있는 URL 로 리다이렉션 되는듯. 
