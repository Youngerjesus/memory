## Spring Cloud vs Kubernetes

#### Spring Cloud Strengths and Weaknesses

- Netflix 에서 오픈소스로 배포한 라이브러리들이 많았다. 서비스 디스커버리, 로드 밸런싱, Circuit Breaking, Config Server 등등 

- 가장 큰 단점은 Java 만 지원해준다.  

#### Kubernetes Strengths and Weakness

- 쿠버네티스에 올리는 서비스는 Java 에 한정되어 있지 않다. 

- MSA 운영 측면에서 좀 더 많은 것들을 지원해준다고 느꼈다. 

  - Container 가 죽으면 다시 시작해주는 Self-Healing
  
  - 급작스러운 트래픽에 대비할 수 있는 Auto Scaling
  
  - 하나의 파드가 너무 많은 리소스를 잡아먹지 않도록 Resource Constraint 가 있다. 