# Ten Java coding antipatterns to avoid: Worst practices #5 through #1

https://blogs.oracle.com/javamagazine/post/java-worst-practices-antipatterns-part-two

## 지켜야 하는 규칙들

- duplicated code 를 제거하도록
- Javadoc 을 최신 상태로 늘 유지하도록. (문서를 만들었으면 항상 최신 상태로 유지하도록 하자. 잘못된 정보를 전달하지 말도록 하자.)
- **User Input 을 절대 믿지마라. `Never trust user input`**
    - 그래서 Input 이 오면 늘 검증을 해야한다.
- Unit 테스트가 없는 것.
- Exception 이 났을 때 무시하지 않도록 하는것 최소한 로그라도 남겨야한다.
