# Scaling up the Prime Video audio/video monitoring service and reducing costs by 90%

https://www.primevideotech.com/video-streaming/scaling-up-the-prime-video-audio-video-monitoring-service-and-reducing-costs-by-90?fbclid=IwAR1LDrZmyf9Wvzo9u2BDfsjmu-WjybzvFoFtVG74NTr2HdFS5NGC0kputYk

***

- 목적: reduce cost and increase scaling capabilities.
  
- 정리
  - AWS Step Function (or Lambda Function) 은 호출할 때마다 가격이 든다. 이런 Serverless function 들을 하나의 프로세스로 묶어서 (= EC2) 로 만들어서 호출하는 비용을 줄였다. 
  - 하나의 Process 로 들어오면서 Scaling 을 늘릴 수 있었다. 
  - 그리고 Video Frame 을 잘개 쪼개서 S3 에 기록해놓는 형태여서 이를 호출하는 비용도 들었는데 이를 메모리단에서 전달하도록 해줘서 비용을 줄였다.  