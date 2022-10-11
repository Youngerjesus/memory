# 토스ㅣSLASH 22 - Java Native Memory Leak 원인을 찾아서

https://www.youtube.com/watch?v=w4fWgLgop5U&t=1s


## 시작

- 서버가 OOM Killed 가 되면 system.log 가 남게 되는데 이를 슬랙 알람으로 오도록 했다.

### system.log

- system log (SYSLOG) 라고 불리며 OS 이벤트와 같은 정보가 포함되어 있다.

    - job time, step time, data from job, EXEC statement

### OOM Killer

- 전체 시스템 안정성을 위해서 희생할 작업을 선택해서 죽이는 애

- 프로세스가 메모리를 과도하게 가지고 있다면 그녀석을 죽임 (OS 에서 메모리가 없으면 안되므로.)

### 문제 서버 상황

- Pod 의 max Memory 는 4GB 로 설정했고 Java 의 Max Heap Size 는 1.5GB 로 설정했다.

- OOM Killer 가 Java 를 죽일 땐 4GB 의 메모리를 사용한 상황이다. 즉  HeapSize 를 제외한 2.5GB 를 더 쓴것.

## 실제로 측정

- java 를 구동할 때 NativeMemoryTracking 옵션을 주면 jcmd 옵션을 통해서 JVM 의 실제 메모리 사용량을 측정할 수 있다.

![](./images/Java_Native_Memory_leak_측정.png)

- `-XX:NativeMemoryTracking=summary` 옵션으로 본 것과 `top` 을 통해서 본 것.

- memory 사용량은 2GB, HeapSize 는 1.5GB, Class 는 190MB 정도이다.

- top 을 통해 실제 메모리 사용량을 보니 (RSS 수치를 통해서 보면 된다.) 여기서 보면 RSS 수치는 2GB 보다 많은 3.1 GB 를 사용하고 있다.

- (여기서는 실제로 앱을 켜보면서 문제 상황을 재현해보고 있음.)

- 여기서 이 사람은 이 1.1 GB 오차와 OOM Killer 가 작동했을 때 2GB 정도의 메모리 오차를 Native Memory Leak 이라고 생각했다. (실제로 JVM 내에서 메모리를 쓰는 부분을 쪼개서 살펴보면 될 듯.)

## 예상 범위 찾기

- 가장 쉬운 부분부터 찾았다. JNI 와 JNA

    - Java Native Interface, Java Native Access 으로 C 로 구현한 모듈을 자바에서 직접 호출하는 기능이다.

    - C 구현체에서 메모리 릭이 생길 경우 JVM 은 이를 탐지할 수 없다고 한다.

    - 근데 문제의 서버에서는 JNI 와 JNA 를 쓰는 부분이 없었음. (어떻게 알았을까)

- 두 번째로 의심했던 부분은 JDK 1.4 부터 지원하는 DirectBuffer 이다.

    - Direct Buffer 로 메모리를 할당하면 GC 되지 않는 네이티브 메모리로 할당받게 된다.

    - 이 양은 NativeMemoryTracking 에서 조회가 가능하다. (이게 매우 작아서 무시했다.)

- 세 번째로 의심했던 부분은 APM (Application Performance Management) 툴이다.

    - APM 툴은 Java Agent 와 동작하며 instrument 를 변경하는 형태이니 네이티브 메모리를 쓰는 부분은 없을까 조심했다.

    - 근데 이도 특별한 건 없었음.

## 범위를 못찾아서 프로세스를 조사했다.

- 프로세스 메모리를 dump 하고 dump 된 파일 내에 포함된 문자열을 뽑아서 힌트를 얻으려고 했다.

    - 메모리 dump 는 pmap, smaps, gdb 를 통해서 알 수 있다.

    - 문자열을 뽑아내는 건 strings 라는 명령을 통해서 알 수 있다.

- 두 번째는 Memory profile 툴을 적용해봤다.

    - linux 에서 메모리를 할당할 때는 malloc 이라는 라이브러리를 사용한다. malloc 을 jemalloc 으로 변경하게 되면 jemalloc 에서 제공하는 profiling 툴을 통해서 어떤 모듈에서 얼마의 메모리를 썼는지 알 수 있다.

![](./images/jemalloc_을_통한_결과물.png)

- 결과물은 이와 같았는데 C2Compiler 에서 프로세스 메모리의 90% 이상인 1.9GB 를 사용하고 있었다.

- 그래서 C2Compiler 가 문제인 것 같아서 찾아보니 openjdk 1.4 의 bug tracking system 을 살펴보니 메모리 릭이 발생했다는 사실을 알 수 있었다.

### C2Compiler 가 무슨 일을 하는가

- JIT 컴파일러가 클래스 파일을 기계어로 변경하면서 level 0 ~ level 4 까지 한다.

    - level 0: Interpreter
    - level 1: C1-no profiling
    - level 2: c1-limited profiling
    - level 3: c1-full profiling
    - level 4: c2
    - 여기서 level 4 를 C2Compiler 가 한다. (C2Compiler 가 Server Compiler 이고 C1Compiler 가 Client Compiler 이다.)
    - 자바에서는 `-XX:TiredStopAtLevel=[0~4]` 를 주면 JITCompiler 의 레벨을 선택할 수 있다.
    - 여기서는 이 옵션을 줘서 C2Compiler 가 작동하지 않고 C1 만 작동하도록 해보면서 메모리 릭이 생기는지 아는지 체크할 수 있었다.
    - 대신에 이건 해결 방법은 되지 않음. C1Compiler 의 최적화는 C2Compiler 의 최적화 보다 약하기 때문에.
    - 다른 방법으로 JDK 버전을 11 그리고 17 로 올려봤는데도 해결은 되지 않음.
- 해결은 JDK 의 다른 컴파일러를 찾아보니 옵션을 주면 Graal Compiler 를 JIT 컴파일러로 쓸 수 있었고 이걸 사용해봤다.
    - 옵션은 `-XX:+UnlockExperimentalVMOptions` 와 `-XX:+EnableJVMCI -XX:+UseJVMCICompiler` 를 줘서 적용했다.
    - 다만 실험적인 설정이므로 다른 문제가 생길 수도 있지만 그건 롤백하면 되는 문제니 일단 적용해보고 실험해봤다.
    - 그리고 Release Note 를 찾아보며 테스트를 해보소 있다.
