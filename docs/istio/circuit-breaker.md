## Circuit Breaker 

***

타임아웃이 날 경우 몇번의 retry 를 할건지와 연속으로 에러가 나면 몇초정도 차단할건지를 결정하도록 이런 것들을 설정하도록 했다.

#### Outlier Detection 이 뭔데? 

- Unhealthy Instance 에 요청을 보내는걸 막는 걸 말한다.

- 각 개별 인스턴스에 대한 연속으로 API 호출이 에러가 발생하거나 Latency 가 떨어지는 걸 보고 Instance 를 확인한다.
  