# Best Pratices for REST API Error Handling

https://www.baeldung.com/rest-api-error-handling-best-practices?fbclid=IwAR3xzfhzxU6Y0C1z_q4ADq5ljpTqx7Hi9c6wSt-LBxbnEpM0NXnL5zP8Eew

***

## HTTP Status Codes 

응답 코드는 카테고리 별로 나뉜다. 특징은 다음과 같다. 

- 100-level (Informational): server acknowledges a request 

- 200-level (Success): server completed the request as expected 

- 300-level (Redirection): client needs to perform further actions to complete the request 

- 400-level (Client Error): client send an invalid request

- 500-level (Server Error): server failed to fulfill a valid request due to an error with server 

## Handling Errors

에러 핸들링의 첫번째 원칙은 요청을 한 클라이언트에게 적절한 상태 코드를 내려주는 것이다. 추가로 Response Body 에 상태 코드를 설명해주는
추가적인 정보도 주면 좋다. 

### Basic Responses 

가장 대표적인 응답들을 보자. 

- 400 Bad Request: 클라이언트가 유효하지 않은 요청을 보낸 경우에 주는 응답 코드다. 주로 요청에 정보가 부족한 경우에 준다. 

- 401 UnAuthorized: 클라이언트의 요청이 인증이 되지 않아서 실패했을 경우에 주는 응답 코드다.

- 403 Forbidden: 클라이언트의 요청이 인증은 되었지만 인가에서 실패했을 경우에 주는 응답 코드다.

- 404 Not Found: 클라이언트가 요청한 리소스를 서버에서 찾지 못하는 경우에 주는 응답 코드다. 

- 412 Precondition Failed: 요청 헤더에 있는 필드 값의 조건이 실패한 경우를 말한다. 

- 500 Internal Server Error: 서버에서 요청을 처리하는데 에러가 나는 경우를 말한다. 

- 503 Service Unavailable: 요청된 서비스가 사용가능하지 않을 때 주는 에렄 코드를 말한다.

### More Detailed Response

Spring 에서는 에러가 생기면 자동으로 형식을 갖춰서 다음과 같은 응답 메시지를 만들어준다.

```json
{
    "timestamp":"2019-09-16T22:14:45.624+0000",
    "status":500,
    "error":"Internal Server Error",
    "message":"No message available",
    "path":"/api/book/1"
}
```

하지만 이런 메시지보다 보다 에러를 설명하는데 도움이 되는 메시지를 클라이언트에게 줘야 하는 경우도 있다. 

예를 들면 유니크한 에러 코드를 줘서 보다 어떠한 에러인지 확실한 정보를 줄 수 있고 메시지도 국제화에 맞춰서 줄 수도 있다. 

그 다음에 에러에 대한 보다 detail 한 정보도 줄 수 있다.  

### Standardized Response Bodies

이러한 정보를 바탕으로 IETF 는 오류 처리에 맞는 스키마를 고안했었다. 

```json
{
    "type": "/errors/incorrect-user-pass",
    "title": "Incorrect username or password.",
    "status": 401,
    "detail": "Authentication failed due to incorrect username or password.",
    "instance": "/login/log/abc123"
}
```

- type 은 에러를 식별할 수 있는 URI 에 대한 정보다. 이런 URI 를 따라가면 그 에러를 보다 자세하게 설명하는 사이트가 나오도록 설계한다.

- title 은 인간이 알아볼 수 있는 에러 메시지의 요약본이다.

- status 는 HTTP 상태 코드다.

- detail 은 인간이 알아볼 수 있는 에러 메시지를 보다 자세하게 설명한 메시지다.

- instance 는 에러가 발생한 요청의 URI 를 말한다. 

실제로 facebook 은 다음과 같은 에러 메시지를 반환한다. 

```json
{
    "error": {
        "message": "Missing redirect_uri parameter.",
        "type": "OAuthException",
        "code": 191,
        "fbtrace_id": "AWswcVwbcqfgrSgjG80MtqJ"
    }
}
```
