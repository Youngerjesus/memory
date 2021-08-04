## CPU 스케쥴링

***

CPU 는 여러가지 프로세스를 실행하는데 이떄 어떠한 순서대로 실행할지를 결정하는 알고리즘을 말한다. 

CPU 스케쥴링의 알고리즘은 Preemptive (선점) 이냐 Non-Preemptive 냐에 따라 나뉜다.

Preemptive 는 한 프로세스가 CPU 를 점유하고 있을때 인터럽트가 발생하지 않았느데도 다른 프로세스에 의해 CPU 할당을 뺴앗길 수 있다.

Non-Preemptive 는 한 프로세스가 CPU 를 점유했다면 인터럽트가 발생하지 않았다면 빼앗기지 않는다. 

### 선점형 스케쥴링

- SRT(Shortest Remaining Time) 스케쥴링: 짧은 시간 순서대로 프로세스 우선순위를 정해서 실행하는 것

- Round Robin 스케쥴링: 일정 시간동안 정해서 프로세스 실행하는 것 

### 비선점형 스케쥴링 

- FCFS(First Come First Served): 먼저 큐에 도달한 스케쥴링을 먼저 처리하는 것 작업이 끝날때까지 실행한다. 



   