# Thread Dump Quiz 

<details> 
<summary> 스레드 덤프란 뭔가? </summary>

스레드 덤프는 현재 스레드가 처리하고 있는 스냅샷을 가리킨다.  

</details>

<details> 
<summary> 스레드 덤프 분석을 하는 방법 </summary>

- local 이라면 visualVm 
- 배포된 환경이라면 jstack
  - 스레드 덤프는 시간 간격으로 계속 떠야한다. shell script 를 통해서 여러개 뜨고 분석하자. 

</details>


<details> 
<summary> 스레드 덤프로 알 수 있는 정보 </summary>

- 스레드 이름 
- 우선순위 
- CPU 점유한 시간
- 스레드 아이디 
- 스레드 상태
- 콜스택 정보

</details>

<details> 
<summary> 스레드 덤프 분석을 언제 해야할까? </summary>

- CPU 사용량이 지나치게 높은 경우 해당 스레드를 찾아보고 분석하기 위해서 
- 어플리케이션 실행이 느려서 thread block 된 상태가 있는지 확인해볼려고 
- 교착 상태가 있는지 확인해볼려고 
- 락과 관련된 문제를 볼려고 
- 스레드가 정리되지 않고 생기는지 볼려고 



</details>
