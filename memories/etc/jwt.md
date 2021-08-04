## JWT(Json Web Token) 

정의 자체가 Claim 들의 집합을 말한다. 

#### RFC 7519 Spec 

#### Structure 

- Header

- payload 

- signature  

#### JWT 자체 주의할 점

- 누구던지 볼 수 있는 데이터이므로 개인정보를 넣지 않도록 한다. 

- USERNAME 같은 것도 개인정보라고 판단해서 인코딩된 JWT 정보를 한번 더 키로 암호화해서 줄 수도 있지만
그런 경우 복호화 할 때 CPU 연산도 드니까 그런 트레이드 오프를 비교했을 때 그정도까지 신경 쓸 필요는 없다고 생각횄다.

- JWT 탈취 당하는 문제를 생각해서 Expiration Time 을 짧게 줄려고 노력했고 그게 기간이 다되면 Refresh Token 을
기반으로 다시 Access Token 을 발급 받는 걸 생각했다. 

- Expiration Time 을 제외하고 보안에 신경썼던 부분은 개발하지는 않았지만 IP Address 를 기반으로 전혀 다른 IP Address 로 요청이
온다면 패스워드를 통해서 한번 더 인증을 받도록 생각했다.

#### Implement Requirement 

- Encrypt JWT 는 선택사항

- Signature 및 HMAC 에서 SHA-256 해시 알고리즘을 사용해야 한다 라는 것. 