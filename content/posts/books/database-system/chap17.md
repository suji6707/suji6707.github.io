+++
categories = ['Database']
series = ['Database-System']
tags = ['데이터베이스']
date = 2024-12-14T22:50:44+09:00
draft = true
+++

# Chap 17. Transaction

### 💎 결론
```
1.직렬: T1이 끝난 후 T2 실행하는 스케줄
2.직렬가능: 동시 수행해도 순차 수행한 것과 결과가 같은 스케줄
3.💎충돌 직렬가능: 동시 수행시 충돌이 있으나 non-conflict한 연산들간 순서를 조정하면 직렬화(T1->T2) 할 수 있는 스케줄.
- 3을 판단하려면 우선순위 그래프를 그려서 cycle이 없으면 직렬 가능하다.
4.뷰 직렬가능: read 시점에 결과가 동일한 스케줄. 더 느슨한 제약.
```

### Transfer 예제
트랜잭션이란, 데이터베이스의 여러 연산이 하나의 단위처럼 여겨지는 경우.
- Atomicity: All or nothing이어야.
- Consistency: 데이터베이스 일관성 보존. all balance 합은 전후로 같아야한다.
- Isolation: i 가 끝나고 j가 실행되는것과 같이 보장해주어야.
- Durability: 시스템 오류에도 영구 반영되어야

isolation은 순차적 실행으로 쉽게 달성할 수 있지만,
여러 트랜잭션을 동시에 실행하는 건 큰 이점이 있으므로 포기할 순 없다.

상태
- aborted: 실패 후 롤백된 상태.
- committed: 마지막 statement가 실행된 후 최종 커밋.
-> 실행에서 끝나는 게 아니라 최종 마무리 단계가 있음!


### Concurrent Executions
동시성 제어 메커니즘

Schedules
- 트랜잭션들의 실행 순서. 

여러 트랜잭션 동시 수행시 스케줄은 더이상 순차적이지 않아도 되지만
'스케줄의 동등성을 보존(preserve)'하도록 이뤄져야 함.
-> `write의 위치와 commit을 잘 보라.`

### Serializability (직렬성)
💎동시 수행한 결과가 
트랜잭션을 하나씩 순차적으로 실행하는 스케줄 결과와 동일하게 할 수 있는 스케줄 의미.
= 트랜잭션을 동시 수행해도 일관성을 보장하는 스케줄

1. conflict serializability
Q가 같은 데이터 항목이라 할 때,
i 가 읽고 j가 쓰는 경우 순서는 중요하다. 순서가 바뀌면 읽는 값이 달라짐.

우선순위 그래프(precedence graph)가 acyclic(비순환)인 경우 
위상정렬(topological sorting)을 통해 스케줄 직렬화 할 수 있음.



---
### 트랜잭션 Isolation, Atomicity
트랜잭션 실패의 영향

복구 가능한 스케줄
- i -> j 순서의 경우,
- Ti, Tj에서 i가 실패할 경우 i 커밋 연산이 j 커밋보다 먼저 발생해야 두 개의 커밋을 모두 취소할 수 있다.

비연쇄적인 스케줄이 바람직
- j가 읽기 연산 실행 전 i 커밋이 먼저 실행.

Goal: 직렬성을 보장하는 동시성 제어 프로토콜(규약)을 만드는 것이나,
약한 수준의 일관성을 대신 사용하는 방법도.

### isolation level
weak level은 직렬 불가능한 방식으로 동시 트랜잭션 수행되는 것을 허용함.
- Read uncommitted: 커밋되지 않은 데이터 읽기
다른 트랜젝션이 기록한 항목을 커밋 전에 읽는 것을 허용
시간이 오래걸리고 결과가 정확하지 않아도 되면 가능.
직렬가능 방식으로 하게되면 교차수행 과정에서 지연되므로.
- Read committed: 커밋된 레코드만 읽을 수 있다.
연속된 읽기가 다른(but committed) 값을 리턴할 수도..
- Repeatable read: 커밋된 레코드만 읽을 수. 
한 트랜잭션이 한 레코드를 두 번 읽는 사이 다른 트랜잭션의 갱신을 막음.
그러나 트랜잭션은 not serializable 할 수도.


예시

```sql
트랜잭션 T1:
SELECT * FROM Students WHERE department = 'CS';  // 결과: 10명
// 다른 작업 수행
SELECT * FROM Students WHERE department = 'CS';  
// Repeatable read 결과: 여전히 10명 
// Read committed 결과: 12명 (커밋된 레코드는 읽음)

트랜잭션 T2 (T1 실행 중에 수행):
INSERT INTO Students VALUES ('John', 'CS');
INSERT INTO Students VALUES ('Jane', 'CS');
COMMIT;
```
💎 Read committed는 직렬 가능함. 동시 실행이 순차적 실행과 결과가 같으므로. 
```sql
T1: SELECT * FROM Students WHERE department = 'CS';  // 10명
T1: SELECT * FROM Students WHERE department = 'CS';  // 10명
T1: COMMIT
T2: INSERT INTO Students VALUES ('John', 'CS');
T2: INSERT INTO Students VALUES ('Jane', 'CS');
T2: COMMIT
```

기본 셋팅
- MySQL: Repeatable read (엄격)
    - 트랜잭션 시작 시점의 스냅샷을 사용한다고 볼 수?
- Postgres: Read committed- `내가 하던 중간에 남이 변경한 것도 읽겠다`
    - 각 쿼리마다 새로운 스냅샷을 생성, 쿼리가 시작할 때마다 그 시점에 커밋된 데이터를 보게 됨. 따라서 같은 트랜잭션 내에서도 다른 쿼리는 다른 결과를 볼 수 있음.


SQL에서의 트랜잭션
- 대부분 DB 시스템에서, 모든 SQL statement는 쿼리가 성공하면 암묵적으로 커밋함. 끌 수도 있음.


트랜잭션마다 isolation 레벨 설정할 수 있음.
In SQL set transaction isolation level serializable
-> **Serializable**은 트랜잭션이 동시에 실행되더라도 마치 순차적으로 실행된 것처럼 동작하도록 보장하는 가장 높은 격리 수준입니다. 즉, 트랜잭션은 자신이 시작되기 전에 커밋된 데이터만 볼 수 있으며, 트랜잭션 도중 다른 트랜잭션이 데이터를 변경하더라도 해당 변경 사항을 볼 수 없습니다.
- 결국 완벽한 isolation을 하려면 지연이 되는것 아닌가?

---
### Isolation level을 구현하는 방법
Locking
- 전체 vs 아이템
- 얼마동안?
- 공유 vs exclusive locks

타임스탬프
- out of order 접근을 탐지하기 위해 사용


2단계 잠금
트랜젝션 두 단계. 잠금을 획득하기만 하고 해제하지 않는 것, 잠금을 해제하기만 하고 획득하지는 않는 것. 
share lock: 읽을 때, exclusive: 쓸 때.

---
# 실전문제
17.1 
DBMS는 고장나지 않는다해도 트랜잭션 실패시 복구 필요.
애플리케이션 오류, 사용자의 취소로 트랜잭션이 중단될 수 있고
데드락으로 트랜잭션이 진행되지 못하는 경우도.

17.2 
파일을 생성/삭제/쓰는 세부단계?
1. 스토리지 공간이 파일시스템에 할당된다
2. i-number을 파일에 할당, i-node entry 삽입
삭제는 정확히 반대.
파일시스템이 트랜잭션을 지원하진 않지만
파일 생성/삭제 과정은 atomic 해야한다. 그렇지 않으면 unreferencable files, unusable areas가 생길테니.. 
공간을 만들고 + 연결하고. 

17.3
통화거래, 좌석 예약 등- 실물과 연동되므로 ACID 속성은 보장되었어야 했다.
반면 파일시스템 유저 대부분은 ACID를 지원하기 위해 돈을 지불할 의사가 없다. 

17.4
어떤 종류의 저장소를 사용해야 내구성을 보장할 수 있을까?
- Stable Storage는 데이터 손실 가능성을 거의 0에 가깝게 줄이기 위해 데이터를 여러 곳에 복제하고 물리적으로 분산하여 저장하는 추상적인 개념
-> 비휘발성 저장 장치를 대량으로 사용함으로써 구현 가능.

17.6
우선순위 그래프 트랜잭션 예시
💎 아래 예시에서 
```
"여러 트랜잭션이 동시 실행해도 순차 실행과 같은 결과를 얻을 수 있다"는 말은 
실제로는 동시에 처리되더라도, 데이터베이스 시스템이 트랜잭션의 연산 순서를 조정하거나 
잠금(Lock) 메커니즘을 사용하여
순차 실행과 같은 결과를 보장하도록 제어한다는 의미.
(내부적으로 연산 순서를 조정)
```

#### 계좌 이체와 우대 등급 조정 트랜잭션 예시

다음과 같은 상황과 이에 대한 트랜잭션들을 생각해 보겠습니다.

* 계좌 테이블 (Accounts): 계좌번호(account_id), 잔액(balance), 고객 ID(customer_id), 우대 등급(grade) 정보를 저장
* 고객 테이블 (Customers): 고객 ID(customer_id), 이름(name), 총 자산(total_assets) 정보를 저장

**트랜잭션:**

* **T1**: A 계좌에서 B 계좌로 100만원 이체
* **T2**: B 계좌에서 C 계좌로 50만원 이체
* **T3**: C 고객의 총 자산 정보 업데이트
* **T4**: A 계좌의 잔액을 기반으로 A 고객의 우대 등급 조정
* **T5**: B 고객의 이름 변경

**Precedence Graph:**

* T1 → T2: T1의 A 계좌에서 B 계좌로 이체하는 작업이 완료되어야 T2에서 B 계좌 잔액을 사용할 수 있습니다.
* T1 → T4: T1의 A 계좌 잔액 변경이 반영된 후 T4에서 A 고객의 우대 등급을 조정해야 합니다.
* T2 → T3: T2의 C 계좌로 이체하는 작업이 완료되어야 T3에서 C 고객의 총 자산 정보를 정확하게 업데이트할 수 있습니다.
* T4 → T5: T4에서 A 고객의 우대 등급을 조정하는 작업과 T5에서 B 고객의 이름을 변경하는 작업은 서로 의존성이 없으므로 순서가 바뀌어도 무방합니다. 

**실제 트랜잭션:**

```sql
-- T1: A 계좌에서 B 계좌로 100만원 이체
UPDATE Accounts SET balance = balance - 1000000 WHERE account_id = 'A';
UPDATE Accounts SET balance = balance + 1000000 WHERE account_id = 'B';

-- T2: B 계좌에서 C 계좌로 50만원 이체
UPDATE Accounts SET balance = balance - 500000 WHERE account_id = 'B';
UPDATE Accounts SET balance = balance + 500000 WHERE account_id = 'C';

-- T3: C 고객의 총 자산 정보 업데이트
UPDATE Customers SET total_assets = (SELECT SUM(balance) FROM Accounts WHERE customer_id = 'C') WHERE customer_id = 'C';

-- T4: A 계좌의 잔액을 기반으로 A 고객의 우대 등급 조정
-- (잔액이 1000만원 이상이면 'VIP', 미만이면 '일반' 등급으로 업데이트)
UPDATE Accounts SET grade = CASE WHEN balance >= 10000000 THEN 'VIP' ELSE '일반' END WHERE account_id = 'A';

-- T5: B 고객의 이름 변경
UPDATE Customers SET name = '새로운 이름' WHERE customer_id = 'B'; 
```

**결론:**

위 예시처럼 실제 트랜잭션들을 분석하여 precedence graph를 그리고, 이를 통해 트랜잭션 스케줄의 직렬 가능성을 판단할 수 있습니다. 

---
17.7
- 현재 server에 비추어 보면
단일 쿼리들을 각각 자동 커밋이 되는거고,
transaction을 명시적으로 시작한 경우 
연쇄적인 방식으로 수행한듯. 하나가 앞선 쿼리 결과에 의존하는 경우가 많은.

```
// Cascadeless schedule
T1: Read(A)  
T1: A = A - 50 
T1: Write(A)
T1: Commit
T2: Read(A)
T2: A = A - 20
T2: Write(A)
T2: Commit

// Non-Cascadeless (연쇄적)
T1: Read(A)  
T1: A = A - 50 
T1: Write(A) 
T2: Read(A)
T2: A = A - 20
T2: Write(A)
T2: Commit
T1: Abort // 문제 발생! T2는 이미 T1의 변경된 데이터를 읽었음
```
---
17.8
lost update

repeatable read level의 경우 lost update는 발생하지 않음.
T1 트랜잭션은 끝날때까지 데이터 항목 X에 대한 shared lock을 들고 있는다. 
이는 새로운 트랜잭션 T2가 X에 쓰려해도(exclusive lock 필요) T1이 끝날 때까지 쓸 수 없게 만든다. 
-> 이는 T1, T2의 직렬 순서를 강제하고, 따라서 T2에 의해 쓰여진 값은 손실되지 않는다. 

17.10

스냅샷 격리 수준으로 인해 두 트랜잭션 모두 성공적으로 커밋되어 실제 좌석 수보다 많은 예약이 발생하는 **초과 예약(Overbooking)** 문제가 발생합니다.

**항공사가 감수하는 이유:**

* **낮은 발생 확률:** 초과 예약은 동시에 정확히 같은 좌석에 대한 예약 요청이 발생해야 하므로 실제로는 드물게 발생합니다.
* **성능 향상:** 스냅샷 격리는 잠금 기반 방식보다 동시성이 높아 시스템 성능을 향상시킵니다.
* **해결 가능성:** 초과 예약이 발생하더라도, 대부분의 경우 취소 또는 대체 항공편 제공 등으로 해결 가능합니다.

**결론:**

항공사 예약 시스템과 같이 동시성이 중요하고 초과 예약 문제를 감수할 수 있는 경우, 스냅샷 격리 수준을 사용하여 성능 향상을 얻는 것이 유리할 수 있습니다. 

💎참고
- 스냅샷 격리 수준: 트랜잭션 시작 시점 스냅샷으로 일관된 보기를 제공하나 phantom read 문제
- Read committed: 쿼리 실행시점 최신 데이터, but Non-repeatable read 발생. 

Phantom Read (팬텀 읽기) 란 트랜잭션 실행 중 동일한 쿼리를 두 번 이상 실행했을 때, 첫 번째 쿼리에서는 없던 새로운 레코드가 두 번째 쿼리에서 나타나는 현상을 말합니다. 마치 유령처럼 나타났다 사라지는 것처럼 보이기 때문에 팬텀 읽기라고 부릅니다.

---
17.11
멀티 프로세서 환경에서 연산들의 순서를 정확히 알 수 없더라도, 충돌 직렬화 가능성을 판단하는 데 문제가 없는가?

**답변:**

문제없습니다. 왜냐하면 충돌 직렬화 가능성을 판단할 때 중요한 것은 **같은 데이터 항목에 대한 연산들의 순서**이지, **서로 다른 데이터 항목에 대한 연산들의 순서는 중요하지 않기 때문입니다.**

**예시:**

질문에 나온 스케줄에서 `read(A)`와 `read(B)`는 서로 다른 데이터 항목에 대한 연산이므로 순서가 바뀌어도 결과에 영향을 미치지 않습니다. 중요한 것은 `read(B)`가 `write(B)`보다 먼저 실행되어야 한다는 것입니다.

**결론:**

멀티 프로세서 환경에서 연산들의 순서를 정확히 알 수 없더라도, 각 데이터 항목에 대한 🔴연산들의 순서🔴를 보장할 수 있다면 충돌 직렬화 가능성을 만족할 수 있습니다. 

---

직렬화 가능성(Serializability)과 충돌 직렬화 가능성(Conflict Serializability)의 차이를 설명하겠습니다:

1. 직렬화 가능성(Serializability):
- 동시 실행의 결과가 어떤 직렬 실행의 결과와 동일한 경우
- View serializability를 포함하는 더 넓은 개념

2. 충돌 직렬화 가능성(Conflict Serializability):
- 충돌하는 연산들의 순서가 어떤 직렬 실행과 동일한 경우
- 더 제한적인 개념

예시:
```
초기값: x = 100

스케줄 1:
T1: r(x), x+1 계산, w(x)  // x를 1 증가
T2: r(x), x+1 계산, w(x)  // x를 1 증가

직렬 실행 1 (T1→T2): 
T1: r(x,100), w(x,101)
T2: r(x,101), w(x,102)
결과: x = 102

직렬 실행 2 (T2→T1):
T2: r(x,100), w(x,101)
T1: r(x,101), w(x,102)
결과: x = 102

동시 실행 A (충돌 직렬화 가능하지 않음):
T1: r(x,100)
T2: r(x,100)
T1: w(x,101)
T2: w(x,101)
결과: x = 101

동시 실행 B (충돌 직렬화 가능하지 않지만, 직렬화 가능함):
T1: r(x,100), x+1 계산
T2: r(x,100), x+1 계산
T1: w(x,101)
T2: w(x,101)
결과: x = 101
```

동시 실행 B는:
- 충돌 직렬화 가능하지 않음 (충돌하는 연산들의 순서가 어떤 직렬 실행과도 일치하지 않음)
- 하지만 직렬화 가능함 (결과값 101은 가능한 직렬 실행 결과임)

이처럼 충돌 직렬화 가능성은 직렬화 가능성의 부분집합이며, 더 엄격한 조건입니다.