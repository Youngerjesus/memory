## Auto Scaling 

쿠버네티스 오토 스케일링은 크게 3 종류가 있다. 

- Horizontal Pod Autoscalar

  - Replication Controller 에 의해 pod 의 개수를 늘려주는 방법
  
  - 주로 CPU 지표를 이용해서 늘리거나 Request 기반으로 늘리는 방법이 있다.  

오토 스케일링은 메트릭 값의 비율에 따라서 달라진다. 

메트릭 값은 CPU 기반으로 알고있다. 

``
현재 레플리카 수 * (현재 메트릭 값 / 원하는 메트릭 값)
``

***

