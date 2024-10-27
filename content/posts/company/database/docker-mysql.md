+++
categories = ['Docker']
series = ['Database']
title = "Docker MySQL"
date = 2024-09-27T22:50:44+09:00
draft = true
+++
## 도커에서 MySQL 복구

### 시사점, 원리
데이터베이스 쓰기 작업이 진행되는 상태에서 갑자기 도커를 재부팅하면 InnoDB 저장엔진에 문제가 발생할 수 있나봄. 일단 데이터베이스 파일이 손상되었음.



키워드
#### Troubleshooting InnoDB table corruption

- https://www.percona.com/blog/recovering-innodb-table-corruption/
- https://dev.mysql.com/doc/refman/8.4/en/forcing-innodb-recovery.html (공식문서)
- https://www.digitalocean.com/community/tutorials/how-to-fix-corrupted-tables-in-mysql
- https://www.cigatisolutions.com/blog/repair-innodb-tables-in-mysql/

예방
- https://serverfault.com/questions/320827/how-to-prevent-innodb-log-files-from-getting-corrupted
- https://dba.stackexchange.com/questions/106452/mysql-innodb-corruption-recovery
    - innoDB tablespace 개념?


#### innodb_force_recovery
손상된 InnoDB 테이블을 복구하기 위해 사용

#### innoDB tablespace
- https://velog.io/@minstone/MySQL-InnoDB-%ED%85%8C%EC%9D%B4%EB%B8%94%EC%8A%A4%ED%8E%98%EC%9D%B4%EC%8A%A4

### binlog
mysql binary log
-> https://blog.naver.com/pjt3591oo/223302518766
MySQL binlog configuration
MySQL binlog for replication
MySQL binlog recovery
MySQL binlog backup


---
### 문제상황
상황 판단
- MySQL doesn't start
    - https://dba.stackexchange.com/questions/324328/mysql-doesnt-start
    정확히 같은 에러로그
- 에러로그에 forcing recovery..가 있었고,
mysql docker forcing recovery 로 검색.

How do start innodb in recovery mode using docker
```
[mysqld]
innodb_force_recovery = 0 
```

- 에러로그: Assertion 체크 실패 관련부분
`Assertion failure: fut0lst.ic:81:addr.page == FIL_NULL || addr.boffset >= FIL_PAGE_DATA thread 32904 I`
-> this looks like a corruption that even `innodb_force_recovery` can't deal with, and restoring from a good backup seems to be the only option here.

---
### 해결과정
dump 했다는 말.
파일이 손상된 것임을 직감. orbstack을 재설치한다고 될일은 아니구나. 

컨테이너 볼륨이 연결된 data를 복구하고 
컨테이너를 지운후 다시 compose up으로 키고싶은데
로컬pc의 mysql 폴더내 데이터가 손상됐다면.?

어떻게든 기존 컨테이너의 mysql에 접속해서 
데이터를 dump 떠와야한다. 

문제는 mysql을 돌리는 컨테이너가 계속 crash가 난다는건데. 
에러로그들을 검색해보니 recovery log 레벨 얘기가 있는데. normal use인 0(보통은 안쓴다는 의미?)에서 1로 높여야겠다는 생각.
관련 mysql 문서 보면 레벨 6까지 있음을 인지. 

우선 옵션을 어디에? 
my.cnf 라는 게 있다.. 다른 사람들 my.cnf 구성을 복사해다가 여기에 복구레벨을 [mysqld] 블록내 추가. 

docker compose yml에서 
볼륨 연결뿐 아니라 
my.cnf를 컨테이너내 지정 경로 /etc로 넣어야함을. 

그렇게 재시작했으나 여전히 꺼졌고,
레벨 6부터 바로 넣어서 또 재시작해봄. 

복구레벨 6:
bin log가 반영 안된 시점부터 현재 db에 적용해야하나 
crash로 인해 실패하고 죽고있으므로,
아예 로그를 기록하지도 적용하지도 않고 
readonly 모드인 상태에서 mysql을 켤수있게 됨. 

여기서 모든 데이터베이스들을 export 해서 로컬에 저장해두었고. 
기존 컨테이너를 맘편히 삭제. 

새로만든 컨테이너에 접속해서 creat db 후 -
mysql u p db명 - 특정 db에 접속, 
to > from 컨테이너 경로에 있는 sql을 넣어줌으로써 import함. 

---

### 근본적인 원인

1. **데이터 파일 손상**: `Assertion failure` 에러와 같은 문제는 일반적으로 데이터 파일의 손상에서 기인합니다. 이는 하드웨어 문제, 파일 시스템 오류, 비정상적인 종료 또는 디스크 공간 부족 등 다양한 원인으로 발생할 수 있습니다.
2. **비정상적인 종료**: 데이터를 쓰는 도중에 시스템이 비정상적으로 종료되면 데이터베이스 파일이 손상될 수 있습니다. 이는 특히 Docker 컨테이너가 예기치 않게 종료되거나 시스템이 강제로 재부팅될 때 발생할 수 있습니다.
3. **디스크 공간 부족**: 데이터베이스가 사용하는 디스크 공간이 부족하면 데이터가 올바르게 저장되지 않을 수 있습니다. 이는 데이터 파일의 손상으로 이어질 수 있습니다.

### 되짚어볼만한 이슈

1. **백업 전략**: 정기적인 백업을 통해 데이터 손상 시 복구할 수 있는 방법을 마련해야 합니다. 이는 특히 중요한 데이터베이스에서는 필수적입니다.
2. **모니터링 및 알림 시스템**: 시스템 상태를 주기적으로 모니터링하고, 디스크 공간 부족이나 비정상적인 종료와 같은 문제를 사전에 감지하여 알림을 받을 수 있는 시스템을 구축하는 것이 중요합니다.
3. **파일 시스템 및 하드웨어 검증**: 파일 시스템의 무결성을 주기적으로 검증하고, 하드웨어의 상태를 점검하여 문제가 발생하기 전에 예방할 수 있습니다.

### 원리

- **InnoDB Force Recovery**: InnoDB의 포스 리커버리 모드는 손상된 데이터베이스를 읽을 수 있도록 다양한 레벨의 복구 작업을 수행합니다. `innodb_force_recovery` 값이 높아질수록 더 많은 데이터 손상 가능성을 무시하고 데이터베이스를 시작하려고 시도합니다.
  - `0`: 기본값으로 복구 모드가 비활성화됩니다.
  - `1`: 일부 손상된 페이지를 무시하고 읽기 전용 모드로 데이터베이스를 시작합니다.
  - `2` ~ `6`: 점차 더 많은 데이터 손상을 무시하고 데이터베이스를 복구하려고 시도합니다.

### 시사점

1. **데이터 무결성 유지**: 데이터베이스의 데이터 무결성을 유지하기 위해 정기적인 백업과 시스템 모니터링이 중요합니다.
2. **복구 계획 마련**: 데이터 손상 시 빠르게 대응할 수 있도록 복구 계획을 마련해 두는 것이 중요합니다.
3. **테스트 환경 구축**: 복구 절차를 정기적으로 테스트하여 실제 상황에서 복구 절차가 원활하게 작동하는지 확인해야 합니다.

### 해결 과정

1. **`my.cnf` 설정 변경**: `innodb_force_recovery` 값을 설정하여 손상된 데이터베이스를 복구하려고 시도합니다.
   ```ini
   [mysqld]
   innodb_force_recovery = 1
   ```
   점차 값을 높여가며 데이터베이스를 시작할 수 있는지 확인합니다.

2. **Docker Compose 파일 수정**: `my.cnf` 파일을 Docker 컨테이너 내에 마운트합니다.
   ```yaml
   services:
     mysql:
       image: mysql:latest
       volumes:
         - ./my.cnf:/etc/mysql/my.cnf
         - ./data:/var/lib/mysql
   ```

3. **컨테이너 재시작**: Docker Compose를 사용하여 컨테이너를 다시 시작합니다.
   ```bash
   docker-compose down
   docker-compose up -d
   ```

4. **데이터 덤프**: 데이터베이스가 시작되면 가능한 데이터를 덤프하여 백업합니다.
   ```bash
   docker exec -it mysql_container mysqldump -u root -p database_name > backup.sql
   ```

5. **복구 및 재설치**: 데이터가 덤프되면 손상된 데이터 파일을 삭제하고, 새로운 데이터베이스 인스턴스를 설치하여 데이터를 복구합니다.
   ```bash
   docker-compose down
   rm -rf ./data/*
   docker-compose up -d
   docker exec -i mysql_container mysql -u root -p database_name < backup.sql
   ```

이 과정을 통해 데이터베이스를 복구할 수 있습니다. 그러나 데이터 손상의 근본 원인을 해결하지 않으면 동일한 문제가 재발할 수 있으므로, 시스템 모니터링과 정기적인 백업을 통해 예방 조치를 취하는 것이 중요합니다.