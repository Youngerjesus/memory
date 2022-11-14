# SixLog 


https://velog.io/@pk3669/Mysql-Redo-Undo-Log
https://www.alibabacloud.com/blog/what-are-the-differences-and-functions-of-the-redo-log-undo-log-and-binlog-in-mysql_598035
https://velog.io/@youngerjesus/%ED%8A%B8%EB%9E%9C%EC%9E%AD%EC%85%98Transaction
https://dev.mysql.com/doc/refman/8.0/en/server-logs.html

*** 

## SixLog Type  

- redoLog
- undoLog
- binLog
- errorLog 
- slowQueryLog 
- generalLog 
- relayLog 



## redoLog 

- Redo Log File 은 Redo Log Buffer 라는 메모리 공간에 기록되다가 디스크에 플러쉬됨.
- 하나의 파일이 가득차면 log Switch 가 발생하면서 checkpoint 이벤트가 발생함. 이떄 디스크에 기록됨.
- 즉 현재 체크포인트 전에 장애가 발생나면 이전 체크포인트까지의 redoLog 만이 기록됨. 
- 트랜잭션 커밋이 수행되면 redoLog 도 File 에 기록됨.
- Physical file
  - `innodb_log_group_home_dir`: log file 이 기록되는 곳. 기본 값은 `./` 으로 데이터베이스 데이터 디렉토리에 있다.
  - `innodb_log_files_in_group`: redoLog 파일 그룹의 숫자로 기본 값은 2임.
  - `innodb_log_file_size`: redoLog 파일의 사이즈      

## UndoLog 

- 변경 전의 값을 기록함. 되돌리기 위해서 (트랜잭션이 발생하기 전에 기록한 데이터라고 생각하면 됨. MVCC (Multi-version Concurrency Control) 에도 사용됨. 락 없이 읽는거 가능.) 
- 체크포인트 할 때 기록함. 
- 트랝잭션 시작전에 만들어짐. (현재 버전의 문서를 바탕으로.)
- 롤백을 위해서 사용됨. 
- 트랜잭션이 커밋된다고 바로 삭제되지 않고 삭제되는 링크드 리스트에 저장됨. 

```text
innodb_undo_directory = /data/undospace/ -- The Directory of an undo independent tablespace 
innodb_undo_logs = 128 -- the size of the rollback segment is 128KB. 
innodb_undo_tablespaces = 4 -- specify four undo log files.
```

## BinLog 

- replication 에서 사용됨. Slave 에서 Master 에 있는 로그를 replay 하는 용도 (= Statements that change data (also used for replication))
- 복구에도 사용됨.
- 기록되는 건 트랜잭션에서 실행되는 SQL 문.
  - 그리고 들어가는 정보는 reverse information 을 넣는다는데 왜 그런지는 잘 모르겠음.
    - delete 는 delete
    - update 는 before 과 after 
    - insert 는 delete 후 insert 
- `expire_logs_days` 이후에 제거됨.
- 파일이 저장되는 위치는 설정 정보안에 있는 `log_bin_basename` 에 있음

## errorLog

- MySQL server 인 `mysqld` 가 starting, running, stopping 중에 만났던 에러를 기록함. 
- errorLog 는 설정에 따라서 에러 메시지를 Performance Schema 에 있는 `error_log table` 에서 조회하는게 가능. 

```sql
mysql> SELECT * FROM performance_schema.error_log\G
*************************** 1. row ***************************
    LOGGED: 2020-08-06 09:25:00.338624
 THREAD_ID: 0
      PRIO: System
ERROR_CODE: MY-010116
 SUBSYSTEM: Server
      DATA: mysqld (mysqld 8.0.23) starting as process 96344
*************************** 2. row ***************************
   LOGGED: 2020-08-06 09:25:00.363521
 THREAD_ID: 1
      PRIO: System
ERROR_CODE: MY-013576
 SUBSYSTEM: InnoDB
      DATA: InnoDB initialization has started.
``` 

- mysqld 가 정상적으로 종료되지 않았더라면 stackTrace 를 남기는게 가능함. (특정 운영체제들에서.)

## General query log

- client connect, disconnect 에 대한 정보를 기록하고, 클라이언트로 부터 받은 `SQL statements` 를 기록한다.
- 클라이언트로부터 어떤 SQL 문을 받았는지, 클라이언트로 부터 받은 에러를 의심할 때 유용함.

## Relay Log 

- `Data changes received from a replication source server`

## Slow query log

- `Queries that took more than long_query_time seconds to execute`

## DDL log (metadata log)	

- `Metadata operations performed by DDL statements`

## ETC 

- errorLog 를 제외하고 어떠한 로그던지 기본적으로 활성화 되어있진 않음.
- MySQL Server 는 mysqld 임. 그리고 data directory 에 기본적으로 로그를 기록함. 
- MySQL Server 는 로그 파일을 close 하고 reopen 하는 것 가능. 그렇다면 flush 를 해야함.