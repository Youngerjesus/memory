# Unit Testing

- 테스트를 위해서 Application Context 가 필요하다면 두 개의 에노테이션을 사용하면 된다. 
  - @SpringJUnitConfig indicates that the class should use Spring’s JUnit facilities
  - @SpringBatchTest injects Spring Batch test utilities (such as the JobLauncherTestUtils and JobRepositoryTestUtils) in the test context

- JobLauncherTestUtils 을 통해서 Job 과 step 을 실행하면 된다.

- launchJob() 을 통해서 Job 을 실행하고 JobExecution 객체를 통해서 assertion 하면 된다.
- launchStep() 을 통해서 Step 을 실행할 수 있다. 그리고 이것도 JobExecution 을 통해서 assertion 하면 된다. 
- step-scope 영역을 테스트 하고 싶다면 @SpringBatchTest 통해서 Context 를 주입받거나 MetaDataInstanceFactory.createStepExecution() 을 통해서 StepExecution 을 만들어서 실행해보면 된다.

- MetaDataInstanceFactory 를 통해서 test module 을 만들 수 있다. 




