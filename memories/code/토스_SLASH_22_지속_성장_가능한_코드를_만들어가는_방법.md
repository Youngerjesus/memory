# 토스| SLASH 22 - 지속 성장 가능한 코드를 만들어가는 방법

https://www.youtube.com/watch?v=RVO02Z1dLF8&t=25s

***

주제는 코드에 대해서 다룬다.

- 확장 가능한 코드가 뭔지.

- 어떤 코드가 품질이 좋은가?

#### 이 코드를 보고 어떤 역할을 하는지 알 수 있는가?

```kotlin
class HamburgerService( 
    private val hamburgerSelector : HambergerSelector, 
    private val breadOven : BreadOven,
    private val vegetablesBucket : VegetablesBucket, 
    private val cheeseStorage : CheeseStorage,
    private val beefManager : BeefManager
)
```

- **이름과 생성자의 요소들만 보고도 어느정도 추리가 가능하다.**

    - 의존관계를 어떻게 맺느냐도 중요하네. 너무 하위 클래스와 의존관계를 맺고 있으면 무슨 일을 하는지 예측이 힘들듯.


package -> layer -> module 구조로 설명

- 코드를 통제 하기 위함. 이런 통제를 바탕으로 제어를 함.

### Package 관리

package 에 아무거나 다 담으면 안된다.

- **package 는 전략에 따라 응집도를 나타내야한다.**

- **햄버거가 하나의 포장지에 담기듯이 해야함. 빵과 치즈 패티를 각각의 포장지로 넣는게 아니라.**

- **같은 개념이 패키지에 있다면 import 문을 없앨 수 있다. 기본적으로 참조가 되기 때문에. import 문을 보고 내가 모듈 관계를 나타내는 패키지 구성을 잘못한거 아닌가 생각할 수 있ㅇ르듯.**

- 한 개념에 클래스가 너무 많다면 어떻게 해야할까? 적절하게 쪼개야한다.

### Layer 관리

Layer 관리는 어떻게 해야할까

토스 페이먼츠의 래이어 관리를 보자.

![](./images/토스페이먼츠의_레이어관리.png)

- 밑에 있는 레이어는 건너뀌기가 아님.

- 만약에 프레젠테이션 레이어가 컨트롤러에서 받은 객체를 비즈니스 영역으로 그대로 전달해준다면 비즈니스 레이어에 있는 객체는 컨트롤러를 참조하게 되면서 역류하게 된다.

    - 이를 막으려면 프레젠테이션 레이어에서 비즈니스 레이어로 전달할 때 변환해서 넣어줘야함.

### module 관리

토스 페이먼츠는 모듈 관리를 어떻게 하는지 보자.

![](./images/토스페이먼츠_모듈관리.png)

- Runtime 시에는 Runnable 한 모듈을 중심으로 의존성이 주입된다.

- 회색 모듈은 외부 기능을 확장할 때 새로운 모듈을 만들어간다.

- 모듈을 나눔으로써 역할과 경계가 명확해지고 테스트하기도 용이하다.

- **비즈니스 로직에 라이브러리 코드가 참조되지 않도록 하자. 외부 모듈은 격리하자.** 
