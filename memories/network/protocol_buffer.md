# Protocol Buffer 

https://developers.google.com/protocol-buffers/docs/overview

## Overview 

- Protocol Buffer 는 언어 중립적인, 플랫폼 중립적인 방식으로 데이터를 전달하는 방식이다. JSON 과 같음.
  - 이 방법은 forward-compatible (상위 호환), backward-compatible (하위호환) 을 지원한다.
    - 데이터를 네트워크를 통해서 전달할 떈 이 요소들을 고려해야한다. 
- Protocol Buffer 는 최대 몇 MB 정도되는 정형화된 데이터를 직렬화해서 전송할 떄 유용하다.
  - 구글에서는 데이터를 디스크에 보관할 떄, 서버간 통신할 때 사용한다.
  - gRPC 로 통신할 떄 주로 사용하는 듯. 
  - 주로 장점은 Compact data storage, Fast parsing, Availability in many programming languages, Optimized functionality through automatically-generated classes 에 유용하다.
- Protocol Buffer 를 쓰면 안되는 경우는 다음과 같다. 
  - 데이터의 크기가 MB 를 초월할 떄. Protocol Buffer 를 쓰면 한번에 데이터를 메모리에 로드하기 때문에 이것보다 크다면 메모리 사용량이 스파이크 칠 수 있다.
  - Protocol Buffer 에 의해서 직렬화가 된다면 바이너리 포맷으로 되고 다른 종류의 포맷이 있을 수 있다. 이것들을 비교할려면 파싱을 해야만 비교 가능. 그렇지 않다면 비교하기 힘들다.  
