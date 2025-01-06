+++
categories = ['Database']
series = ['Postgres']
tags = ['Postgres']
title = "Postgres"
date = 2024-08-12T22:50:44+09:00
draft = true
+++

# 💎 Postgres 슬롯페이지 구조 직접 들여다보기
https://drew.silcock.dev/blog/how-postgres-stores-data-on-disk/#what-about-indexes

~/db/postgresql/pg-data/base
```
drwx------  296 heojisu  staff   9.3K  8 10 13:39 1
drwx------  296 heojisu  staff   9.3K  8 10 13:39 13822
drwx------  297 heojisu  staff   9.3K  8 10 15:24 13823
drwx------  296 heojisu  staff   9.3K  8 10 13:39 16384
drwx------  232 heojisu  staff   7.3K  8 10 15:26 16385
```
- 데이터베이스가 4개 존재하고, 각각 oid를 가짐 (object id는 Database든 Table이든 항상 생성되는 값인듯)

```
select * from pg_class
where relnamespace = to_regnamespace('public')::oid;

oid   |      relname       |
16386 | countries_id_seq
16387 | countries
16392 | countries_pkey
16394 | countries_name_key 
```
- blogdb인 16385 안에는 public 네임스페이스 테이블 4개가 존재하고
연속적인 id를 생성 후 실제 데이터 테이블, 이어 인덱스 키 파일들을 생성함. (index file은 search key, pointer로 구성)
- countries는 row가 249개인데 그 폴더내에는 303개나 되는 파일들이 있음. -> row 개수와 전혀 상관없다는말.
- pkey 폴더를 생각.
	- 가장 중요한 건 ctid. 블록 넘버와 블록내 row 넘버.
	- page 계층구조로 page0 -> ctid로 다른 블록의 페이지내(page1) 인덱스 위치를 가리키고, page1은 또 page3의 특정 row를 가리키는 식.
	- page3의 pointer는 Real Data가 있는 countries의 특정 블록 + row 넘버를 가리킬 것이다.
	- 이 pkey 인덱스가 클러스터링 인덱스(연속적)이기 때문에 앞단에서 page 계층구조로 sparse index를 가질 수 있는 것이다.

- 보면 pkey 인덱스 파일들만 special 값이 들어있음.




---
* 용어
스냅샷: 각 트랜젝션은 자신만의 db copy를 바라봄. 
스키마: logical namespace for tables, functions, triggers, ...
- 데이터베이스는 파일의 물리적 위치. postgres에서는 구분됨. auto 디폴트 스키마가 public이 됨.
데이터베이스 안에 논리적 스키마라는 구분단위가 있는데, 지정안하면 public을 쓰면 됨.
-> 권한관리같은 것이 편해질수있어서.? 스키마를 구분하는듯

* 폴더 구조
base/ 실제 데이터있는
global/ 클러스터 와이드. infor_schema같은. lock상태? 
dynamic shared memory/ 
- mysql은 스레드(스레드 풀링으로 동시연결), postgres는 프로세스 기반 아키텍처(프로세스간 독립성 보장됨)
- 프로세스는 메모리를 따로쓰므로 share하기 위한 서브 시스템을 가진 것.

pg_logical/
-> 파일에 적용하는데 시간이 걸리니(헤더 바꾸고. 다른 페이지에 있을수) 우선 WAL 로그를 남겨둠
- WAL은 데이터베이스가 변경 사항을 실제로 적용하기 전에 이러한 변경 사항을 로그에 기록
논리적 디코딩은 WAL을 고수준의 작업으로 변환
PostgreSQL은 이 논리적 디코딩 과정과 관련된 파일들을 pg_logical/ 디렉토리에 저장

xact/ 다중 트랜잭션 상태 데이터. 
- multitransaction: 두 세션이 동일한 행에 대해 잠금을 시도하는 경우. PostgreSQL은 이러한 상황을 다중 트랜잭션으로 처리하며, 한 세션이 잠금을 획득할 때까지 다른 세션은 대기

postgresql.conf: 실제로 우리가 조정할 수 있는 파일. 

postmaster.pid : 프로세스가 실제로 돌아갈 때. 

---
Object IDentifier (OID) 
postgres, template0, 1, 나는 test까지. 

docker exec -it 5c7290e899db psql -U root -d postgres
-> 데이터베이스 입력 -d

---
행 수는 249인데
base/{database} 보면 300개가 넘음.

* id 생성을 위한 폴더들. id를 생성하고 난 후 나머지 작업들이 이뤄졌나봄.
id_seq: id 생성
pkey: id 프라이머리 키가 저장된. 
name_key: 유니크. 

* filenode_fsm
Free Space Map: 각 페이지별로 공간이 얼마나 남아있는지. 
- 힙구조. 비어있는 데 먼저 쓰는.

fsm, vm은 힙파일을 위한 보조테이블이라
이거 제외한 세그먼트 데이터 파일들이 '힙'이라 불림

키가 있어도 순차적인 저장을 보장하지 않음.
페이지 안에서는 순차적이겠지만.

postgres는 페이지에 4kb가 아니라 8kb를 씀.
32kb -> 4개의 페이지가 있을 것.🔺

❯ ll | grep 16387                                                                                              ─╯
-rw-------  1 heojisu  staff    32K  8 10 14:31 16387
-rw-------  1 heojisu  staff    24K  8 10 14:31 16387_fsm

힙 내-
16387 하나가 세그먼트.
1GB까지 늘어날 수 있고 넘어가면 .1, .2 이런식으로.-

세그먼트 안에는 여러 페이지가 들어가있고.
13만개 페이지가 들어갈 수.

페이지 안에는 페이지 헤더, line pointer . 
item 자체는 뒤에서부터 채워지는 
-> slotted page 구조.

---
# Page Layout
free space의 lower~upper bound.
free space가 그 중간에 비어있는 공간.

페이지 헤더는 
checksum: 오류가 있는지 체크.

32KB -> 4페이지. 0,1,2,3
페이지 3은 별로 안썼음. free space가 3kb나 남아있는.
-> final page.

* 익스텐션을 통해 쓸 수 있는 함수들(EXTENSION) - 마치 duckdb .readparquet처럼
page_header
heap_page_items

🔺 select ctid, id -> (페이지번호, 페이지 안에서 row 넘버)
ctid: current tuple id.

---
#### MVCC (Multiversion Concurrency Control)
동시접근컨트롤.
행을 수정할 때 기존 튜플을 전혀 건드리지 않는다. 
대신 수정된 행을 마지막 페이지 끝에 새로운 행으로 생성한다.
새로운 트랜젝션이 실행되면 기존 것을 대체한다.
- 마치 git 처럼. 반영되고 나면 새로운 것을 바라보도록. '보는 위치'를 바꾸는식. (linked list??)

 lp | lp_off | lp_len | t_ctid | t_data
----+--------+--------+--------+--------
  9 |      0 |      0 |        |
-> lp(line pointer) ctid의 두번째값.
🔺 변경된 게 있으면, 기존 것이 어떻게 되었는지 확인해야 완전히 확인한 것. 
더이상 9번 row가 ctid를 갖지 않음을 확인.
그대로 있긴 해도. 

select *
from heap_page_items(get_raw_page('countries', 0))
offset 8 limit 1;

select lp, lp_off, lp_len, t_ctid, t_data
from heap_page_items(get_raw_page('countries', 1))
limit 1;

---
#### 인덱스
- postgres는 세컨더리 인덱스를 pk처럼 쓰는 것.
어차피 힙파일 구조로 비연속적인 저장이기 때문에.
pk도 논리적으로 쓰는것.


select lp, lp_off, lp_len, t_ctid, t_data
from heap_page_items(get_raw_page('countries_pkey', 1))
limit 10;
-> 키라서 t_data에 아무것도 저장되어있지 않음. 가리키기만 하면 되어서.
- lp_off : line pointer offset은 뒤에서부터 자라나니까.

 lp | lp_off | lp_len | t_ctid | t_data
----+--------+--------+--------+--------
  1 |   8160 |     16 |        |
  2 |   8144 |     16 |        |
  3 |   8128 |     16 |        |
  4 |   8112 |     16 |        |
  5 |   8096 |     16 |        |


---
### 첫 번째 Line Pointer의 Offset이 8160으로 나타나는 이유
ㄴㄴ 아닌듯?
gpt.
PostgreSQL에서 테이블의 각 페이지는 여러 개의 레코드(튜플)로 구성되어 있으며, 각 레코드는 "라인 포인터"에 의해 참조됩니다. Line Pointer는 페이지 헤더 다음에 위치하며, 각 레코드의 위치와 크기를 나타냅니다.

각 Line Pointer는 4바이트(32비트)를 차지합니다. 페이지의 크기가 8192 바이트이고, 페이지 헤더는 24 바이트를 차지합니다. 따라서 첫 번째 레코드의 오프셋을 계산하는 과정은 다음과 같습니다:

1. **페이지 크기**: 8192 바이트
2. **페이지 헤더 크기**: 24 바이트
3. **라인 포인터 크기**: 4 바이트

라인 포인터는 페이지의 끝에서부터 시작하여 이전으로 채워집니다. 각 라인 포인터는 4 바이트를 차지하며, 아래와 같은 방식으로 오프셋이 결정됩니다:

- 첫 번째 레코드의 오프셋: 8192 - 4 = 8188 바이트
- 두 번째 레코드의 오프셋: 8188 - 4 = 8184 바이트
- 세 번째 레코드의 오프셋: 8184 - 4 = 8180 바이트
- 네 번째 레코드의 오프셋: 8180 - 4 = 8176 바이트
- 첫 번째 라인 포인터의 오프셋: 8176 - 4 = 8172 바이트

그러나 실제로, 레코드 데이터는 페이지의 끝에서 시작하여 위쪽으로 채워집니다. 첫 번째 레코드의 데이터는 페이지의 끝부분에 위치하게 됩니다. 이 경우, 첫 번째 레코드의 오프셋은 다음과 같습니다:

- 첫 번째 레코드의 오프셋: 8192 - 16 = 8176 바이트
  - 여기서 16 바이트는 레코드의 크기입니다.

따라서, 첫 번째 레코드의 오프셋이 8176 바이트로 나타나는 것입니다. 나머지 레코드도 같은 방식으로 계산됩니다.

결과적으로, 첫 번째 Line Pointer의 오프셋이 8160으로 나타나는 이유는, 페이지의 끝에서부터 레코드 데이터가 채워지기 때문입니다. 이 방식은 페이지의 공간을 효율적으로 사용하고, 빠른 레코드 접근을 가능하게 합니다.

---

PostgreSQL에서 인덱스 페이지의 line pointer(또는 item pointer)가 16바이트씩 차지하는 이유
- 블록번호(4바이트) + 오프셋(2바이트: 블록내 특정 위치) + 키값(인덱스 키의 실제값)
- ctid(블록번호, 오프셋)

---
