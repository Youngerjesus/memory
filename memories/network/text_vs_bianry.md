# text vs binary

*** 

- 데이터가 더 compact 해지고, parsing 할 때 더 빠름 차이가 남.
  - binary format 과 같은 경우 문자열을 읽을 때 문자열 타입이면 다음 2바이트가 문자열의 길이를 나타내서 한번에 읽어들이는게 가능함. (parsing 속도 차이.) 
    - 읽어들일 데이터의 시작과 끝을 표기하는게 가능함.
  - 숫자를 나타내는 경우에 binary format 이 더 사이즈가 작음. 