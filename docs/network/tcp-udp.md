## TCP vs UDP 

***

#### Transport Layer

신뢰성 있는 데이터 전송을 담당하는 계층 

이 레이어가 없다면 어떻게 되는가 

- 데이터 순차 전송이 안된다.

- Flow 문제가 생긴다 - 송수신자간의 데이터 처리 속도 차이 때문에 일어나는 문제 

  - 송신자는 수신자한테 데이터를 보내는데 수신자는 이 데이터를 처리하는 속도가 느리다. 그러면 데이터를 잘 받기 힘든 문제
  
- Congestion 문제가 생긴다 (네트워크 레벨 즉 라우터에서 유실되는 문제가 생긴다)

#### TCP(Transmission Control Protocol)

- 신뢰성있는 데이터 통신을 가능하게 해주는 프로토콜 

- Connection 연결을 위해 3 way-handshake 가 일어난다. 

- 순차 전송 

- Flow Control

- Congestion Control 

- Error Detection 

- TCP 의 데이터 전송단위는 세그먼트로 데이터에 TCP 헤더를 붙인다. 


#### Connection Handshake
1. SYN 플래그를 통해서 커넥션 연결 시작 

2. SYN 플래그와 함께 ACK 플래그를 통해 패킷 송신

3. ACK 비트를 1로 설정해서 패킷을 송신한다.  

#### Connection Close Handshake
 
1. 데이터를 전부 수신한 Client 가 연결을 종료하는 FIN 을 보낸다. 

2. 서버가 ACK 를 보낸다.

3. 서버에서 남은 패킷을 모두 보낸다. 

4. 서버가 FIN 송신한다.

5. 클라이언트가 ACK 를 보낸다. 

#### UDP

TCP 와는 다르게 연결을 맺지 않는다

데이터 전달을 보장해주지도 않고 순서도 보장해주지 않는다. 하지만 빠르다. 

IP와 거의 유사한데 전달하는 PORT 와 체크섬 정도만 추가된다.  