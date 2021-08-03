## OAuth 2.0 

https://velog.io/@tmdgh0221/Spring-Security-%EC%99%80-OAuth-2.0-%EC%99%80-JWT-%EC%9D%98-%EC%BD%9C%EB%9D%BC%EB%B3%B4
***

용어 정리 

- Resource Owner: 사용자

- Resource Server: 구글 같은 서버  

- Client: 직접 개발한 웹사이트를 말한다. 

#### OAuth 2.0 

OAuth(OpenID Authentication) 은 타사에 저장되어 있는 사용자 정보의 접근 권한을 얻고 그 
정보를 이용해서 AccessToken 을 발급받아서 회원가입이나 로그인을 해줄 수 있는 방식이다.

진짜 간단하게만 말하면 사용자는 구글 같은 타사의 사이트에서 로그인하면 우리 서버는 거기서부터 AccessToken 을 전달받고 
그 AccessToken 을 바탕으로 사용자의 정보를 받아올 수 있다. 

타사의 서비스를 이용하는 것이므로 신청을 해야한다. Client ID, Client Secret, Authorized Redirect URI, 

플로우는 다음과 같다. 

1. 유저가 우리 웹사이트에 로그인을 할려고 한다. 

2. Resource Server 를 구글을 기준으로 하면 구글 로그인 창이 딱 뜰것이고 거기에 사용자가 Id 와 Password 를
입력한다. 

3. 그러면 이제 Resource Server 로 요청이 가는데 AccessToken 을 받을 대상에 대한 검증이 들어간다. 이 웹사이트가
우리 구글에 신청을 한 애인가? 등록된 애 인가? 를 검증하고 만약 통과가되면 Resource Owner 에게 너 정보 이제 
이 웹사이트에게 보여준다? 라는 걸 묻고 그걸 승락하면 웹사이트는 리다이렉트 되고 이때 Authorization Code 를 받는다. 
이 Authorization Code 를 가지고 Resource Server 에게 요청을 하면 Access Token 을 받고 유저 정보에 접근할 수 있다. 
  