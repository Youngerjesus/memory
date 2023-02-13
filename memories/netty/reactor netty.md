# Reactor Netty 

https://projectreactor.io/docs/netty/snapshot/reference/index.html

*** 

## 6.7. Connection Pool

- HTTP Client 는 고정된 커넥션 풀을 가진다. 이건 최대 500개의 활성화 채널을 만들 수 있다. 그리고 최대 1000개의 채널까지 확장가능. pending 된 상태로
  - 커넥션의 개수는 processor * 2, 최소 숫자는 16개. 

- 한 개의 커넥션이 Active 한 channel 을 조회할 수 있는, metric 을 수집할 수 있는 방법이 있을 거 같다. 

- available configuration 
  - `disposeInactivePoolsInBackground`
    - empty or inactive 한 connection 은 제거한다. 기본적으로 비활성화.
  - `disposeTimeout`
    - `ConnectionProvider#dispose()` or `ConnectionProvider#disposeLater()` 호출 될 떄 graceful shutdown 을 지원하는 timeout 을 준다.
  - `evictInBackground`
  - `fifo`
  - `lifo`
  - `maxConnections`
  - `maxIdleTime`
  - `maxLifeTime`
  - `metrics`
  - `pendingAcquireMaxCount`
  - `pendingAcquireTimeout`

- 커넥션의 수를 과도하게 증가해서 과도한 open/acquire 때문에 `reactor.netty.http.client.PrematureCloseException` 이 안나도록 해라. 
  - 서버가 과도한 부하 떄문에 에러를 만날 수도 있고, 
  - idle connection 이 닫혀서 그런 것일수도 있고,

## 6.16. Timeout Configuration

여러 종류의 타임아웃을 다룸. 

- Connection Pool Timeout 
- HttpClient Timeout 
  - Response Timeout 
  - Connection Timeout 
  - SSL/TLS Timeout 
  - Proxy Timeout 
  - Host Name Resolution Timeout 

### 6.16.1. Connection Pool Timeout

- connection 은 사용된 후 close 되는게 아니면 connection pool 로 돌아간다.
  - 이 커넥션은 재사용되거나 idle 상태가 된다. 
- 여기서 설정할 수 있는 옵션들
  - `maxIdleTime`: connection 이 idle 상태로 머무룰 수 있는 최대시간. 기본적으로 무제한임.
    - idletime 은 target server 를 고려해서 써야한다. target 서버보다 작게 써야함.
  - `maxLifeTime`: connection 이 살아있을 수 있는 최대시간. 기본적으로 무제한. 
  - `pendingAcquireTimeout`: connection 을 획득하는걸 기다리는 최대 시간. 기본값 45초.
    - 이 시간이 지나면 `PoolAcquireTimeoutException` 이 발생한다.
- 커넥션을 제거하려면 `evictInBackground` 를 설정해야한다.

### 6.16.2. HttpClient Timeout

- Response Timeout
  - 모든 요청에 대한 응답인 response timeout 이다.

- Connection Timeout
  - `CONNECT_TIMEOUT_MILLIS`
    - remote peer 와 connection 을 establishment 하는데 걸리는 타임아웃 
  - `SO_KEEPALIVE`
    - `TCP_KEEPIDLE`
    - `TCP_KEEPINTVL`
    - `TCP_KEEPCNT`
    - 이 `TCP KEEPALIVE` 를 통해서 silence drop idle connection 문제를 해결해줄 수 있다. 
      - 주로 client-server 간의 네트워크 구성요소가 껴있는 경우에 idle connection 이 있으면 자동으로 drop 시킨다. 이 문제를 해결하는 거임.
      - TCP KEEPALIVE 를 키면 연결이 끊어졌는지 계속 조사할거니까.

- SSL/TLS Timeout
  - `handshakeTimeout`
    - SSL Handshake timeout 을 말함.
    - 느린 네트워크에선 이 값을 올리는게 좋다고 하네. 
  - `closeNotifyFlushTimeout`
  - `closeNotifyReadTimeout`

- Proxy Timeout
  - connectionTimout 지정가능

- Host Name Resolution Timeout
  - DNS 조회도 async 하게 한다고함.
  - `cacheMaxTimeToLive`
    - dns cache 가 살 수 있는 최대시간 
    - DNS Server 로부터 유효한 시간을 받는데 이 값이 캐시 시간보다 크다면 그냥 캐시를 쓴다고함.
    - 기본 값은 Integer.MAX_VALUE
  - `cacheMinTimeToLive`
  - `cacheNegativeTimeToLive`
    - dns 쿼리가 실패했을 떄 cache 가 살 수 있는 시간
  - `queryTimeout`
    - dns 조회 타임아웃

## Question 

### A.5. How can I debug "Connection prematurely closed BEFORE response"?

- Reactor Netty Clients 에서는 connection pool 을 쓴다. 커넥션풀에서 커넥션을 가지고 올 때 유효한지 체크한다. 
- 이렇게 유효한 커넥션을 가지고 와도 커넥션은 종료될 수 있다. 
  - 이 경우 에러는 크게 이렇게 난다. `Broken Pipe" or "Connection Reset by Peer`
  - 이때는 다양한 이유가 있을 수 있다고 한다. 왜냐하면 둘 사이엔 load balancer 나 proxy 같은 것들이 끼어있을 것이니.
    - 이유중 하나는 `idle timeout` 특정 시간동안 들어오는 데이터가 없을 떄 닫힘.
  - 대상 서버 쪽에서 이슈로도 날 수 있다.
    - idle timeout (the connection is closed when there is no incoming data for a certain period of time)
    - limit for buffering data in memory 
    - multipart exceeds the max file size limit 
    - bad request 
    - max keep alive requests (the connection is closed when the requests reach the configured maximum number)
