## Maven Lifecycle

#### compile 

- 소스코드를 컴파일 하는 단계로 결과를 outputDirectory 에 생성한다.

#### test-compile

- 테스크 코드를 컴파일 하는 단계

#### test

- 빌드 전에 테스트를 모두 돌려보면서 성공하는지 체크하는 단계

#### package

- package 를 실행하면 compile, test-compile, test 순으로 실행하고 jar 파일을 target 디렉토리에
만드는 역할을 한다.

#### install

- 로컬 레파지토리에 패키지를 배포하는 역할을 한다.

#### deploy 

- 원격 레파지토리에 패키지를 배포하고 다른 프로젝트에서 이를 사용할 수 있도록 한다.

#### clean 

- target 디렉토리의 결과물을 모두 지우는 역할을 한다. 