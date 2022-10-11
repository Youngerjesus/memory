# MSA 환경에서 네트워크 예외를 잘 다루는 방법

https://tech.kakaopay.com/post/msa-transaction/

## 글로벌 트랜잭션의 정의

- MSA 환경에서 여러 서버들과 연계해서 트랜잭션을 처리하는 것.

- 여러 DB 에 걸친 트랜잭션을 글로벌 트랜잭션, 분산 트랜잭션 이라고도 한다.

- 예를 들면 페이 상품권을 구매하면 페이 머니 서비스에 잔액이 차감되어야 한다. 잔액 차감이 성공적으로 된다면 페이 머니 서비스에서 상품권이 발송이 됨.

- MSA 에서 글로벌 트랜잭션을 다루는 방식은 여러가지가 있지만 대표적으로는 [Saga Pattern](https://microservices.io/patterns/data/saga.html) 이 있다.

### Saga Pattern 소개 정도만

- local transaction 을 여러개 실행한다고 생각하면 된다.

- 각각의 로컬 트래잭션이 자신의 서버에서 실행되고 처리되었음을 다른 서버에게 이벤트로 알려주거나 매시지를 보내는 것.

- 이떄 이후의 로컽 트랜잭션이 문제가 생겼다면 보상 트랜잭션 (compensating transactions) 를 내보내서 이전에 처리했던 로컬 트랜잭션의 작업을 무효화 (undo) 한다.

## 이 글의 요지

- 글로벌 트랜잭션을 보낼 떄 네트워크 간의 요청을 통해서 보낼 수 밖에 없다.

- 근데 네트워크 간의 요청은 언제든 예외가 날 수 있다.
    - 같은 요청이 짧은 시간에 두 번 이상 발생한 경우
    - 네트워크 순서가 뒤집힌 경우
    - 타임 아웃
    - 인프라 문제로 인한 실패

- 그래서 이 포스팅은 이런 네트워크 예외가 날 때 어떻게 대처하는지를 다루겠다. 그런 것인듯.

## 네트워크 요청 (API) 를 안전하게 다루는 방법

- API 를 요청하는 입장에서는 실패가 올 수 있고, 알지 못하는 에러가 올 수도 있다는 점을 알자.

- API 를 제공하는 입장에서는 두 번 이상의 요청이 올 수 있으니까 멱등성의 API 를 제공하자.

    - 주문 서버와 결제 서버가 나눠져있을 때 결제 서버에 한 번 주문했는데 똑같은 주문이 두 번 올 수도 있다.

### 알 수 없는 에러 처리하기 (API 요청자 입장.)

- 요청에 대한 성공 응답을 받지 못했는 경우로 트랜잭션의 결과가 실패했는지, 성공했는지 알 수 없는 경우에 해당한다.

- 다음과 같은 상황일 수 있다.

    - 요청만 간 경우

    - 요청 갔고 성공까지 한 경우

    - 요청은 갔는데 실패한 경우

- 이런 상황을 그냥 실패로 간주하면 문제가 생길 수 있다.

- 이런 상황에서 대응할 수 있는 후처리 기법은 다음과 같다.

    - 즉시 재시도

    - 일정 시간이 지난 후 재시도

    - 요청이 성공했는지 확인 후 재시도

    - 결제 취소 요청 (보상 트랜잭션)

    - 무조건 성공 후 뒤처리

    - 수기로 처리

    - (나는 조건부 보상 트랜잭션을 생각하긴 했다. 성공했더라면 되돌리도록 하는. 그리고 다시 첨주터 시작하도록 할려고. 이런 경우에는 보상 트랜잭션의 메시지가 절대 유실되면 안되겠다.)

    - 그리고 이런 기법들은 추가로 필요한 API 를 필요로 할 수 있다. 트랜잭션이 성공했는지 확인하는 API 라던지, 결제 취소 API 라던지 그런게 있어야 할 수 있음.\

    - (이걸 고려하면 트랜잭션 성공했는지 알아보고, 성공했다면 문제 없고, 성공했는지 알아보는 API 도 응답이 없다면 조건부 보상 트랜잭션을 넣도록 하면 되지 않을까. 그리고 일정 시간 후 재시도.)

- 후처리도 언제든지 알 수 없는 예외가 날 수 있다는 점을 명심하자.

![](./images/요청_응답_상황.png)

- 카카오페이에서는 한 번의 후처리만 하고 거기서도 알 수 없는 에러가 난다면 이 상황을 기록해놓고 다시 처음부터 재시도를 하도록 해주고, 그때 보정 처리를 한다.

- 어쩄든 API 응답은 다음과 같다.

    - 성공, 실패, 알 수 없음.

### 멱등성 API (API 제공자 입장.)

- 주문 서버에서 결제 서버의 응답이 알 수 없음으로 왔다면 결제 서버를 향해서 후처리 API 를 요청하게 될 것.

- 같은 주문에 대해 재시도를 할 여지가 있으므로 같은 주문을 성공헀다면 두 번째 주문에 대해서는 성공으로 응답을 내보내는게 맞을 것이다.

    - 그렇다고 금액을 차감하면 안될것. 즉 멱등성을 보장해야 할 것.

    - 이렇게 멱등성을 보장하기 위해서 요청에 유니크한 `tx_key` 를 같이 보내면 될 것이다. 그렇다면 똑같은 주문이라는 걸 인식할 수 있음.

## 예외를 잘 다루는 방법

try - catch 로 예외를 다룬다고하면 다음과 같이 될 것.

```kotlin
try {
    paymentAdapter.pay(request)
    markSuccess(payTx)
} catch (e: Exception) {
    if (isFailedException(e)) {
        markFailure(payTx)
    } else {
        // 알 수 없는 예외라면 재시도
        try {
            retry(request)
            markSuccess(payTx)
        } catch (e: Exception) {
            if (isFailedException(e)) {
                markFailure(payTx)
            } else {
                markMarkUnknown(payTx)
            }
        }
    }
}
fun isFailedException(e: Exception): Boolean = when (e) {
    is RuntimeException -> true
    is SocketTimeoutException -> false
    else -> {
        println("unknown exception. $e")
        false
    }
}
```

- 코드의 중복도 있고, 들여쓰기 때문에 읽기가 어렵다.

이를 함수형 프로그래밍 처럼 변경해서 더 읽기 쉽게 만들었다.

```kotlin
class PayService(
    private val paymentAdapter: PaymentAdapter
) {
    fun doPay(request: PayRequest): ActResult<String> =
        ActResult { this.validate(request) }
            .flatMap { // validation이 성공했을 때 flatMap 함수 실행
                paymentAdapter.pay(request)
                  .recoverUnknown { paymentAdapter.pay(request) } // unknown인 경우 재시도 가능
            }
            .onSuccess { markSuccess(it) }
            .onFailure { markFailure(it) }
            .onUnknown { markUnknown(it) }
            .map { payResult -> "SUCCESS" } // 마지막 결과를 응답 형식으로 변환
    private fun validate(request: PayRequest) =
        ActResult { paymentAdapter.validate(request) }
            .onSuccess { println("validate success.") }
            .onFailure { println("validate failed.") }
            .onUnknown { println("validate unknown.") }
}
```
