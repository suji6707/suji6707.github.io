+++
categories = ['Database']
series = ['Database-System']
tags = ['데이터베이스']
date = 2024-12-28T22:50:44+09:00
draft = true
+++
# 동시성 제어


#### Lock을 사용하는 것만으로는 직렬화를 보장할 수 없다
처음 x=100, y=200
![image](https://github.com/user-attachments/assets/305f527b-8e5f-4386-a748-88efab4109b5)
<img width="1307" alt="image" src="https://github.com/user-attachments/assets/f587a311-7069-4d9e-b8b9-88ae57190224" />

왜 이런 현상?
T1은 x를 업데이트, T2는 y를 업데이트하는데 
T1은 업데이트된 y를 읽어야 serializable하게 동작을 할텐데 T1이 업뎃 전 y를 읽어버림.

즉 y가 W(y) 락을 획득하기 전 x가 R(y)를 획득해버린게 문제. 
<img width="536" alt="image" src="https://github.com/user-attachments/assets/50b6169c-5ecd-47a6-852c-0419b17ee87e" />
이렇게 2PL로 순서만 바꿔주면
T1이 업뎃된 y 값을 읽을 수 있음.

---
## 그래프 기반
2PL의 대안으로서 충돌직렬 가능성 보장 방법.

데이터 항목의 순서에 관한 사전 지식을 이용.
-> set D = {d1, d2 ,..., dh} 가 순서를 지닌다면
D는 directed acyclic 데이터베이스 그래프로 볼 수 있는 것. 

트랜잭션이 끝나기 전에도 락을 해제할 수 있다.
락 shrink 시점까지 기다려야하는 2PL과 달리..

---
락의 구현

락 매니저는 별개 프로세스로 구현됨
요청이 오면 lock grant 메시지를 보내는데
lock table이라는 인메모리 데이터구조를 갖고있음. 
granted locks와 pending requests를 기록하기 위해 

---
## 데드락
교착상태 예방하려면
각 트랜잭션이 실행 시작 전 필요한 모든 락을 획득하거나
partial/total order를 사용해 특정 항목에 락을 걸면 앞에 있는 항목에는 락을 걸 수 없도록 하는 방법.

preempt 선점 기법: 기존에 락을 갖고 있던걸 취소시켜 선점 후 주기.
타임스탬프 사용:
1) wait-die(non-preempt): 이전에 요청된 트랜잭션만 대기, 다음 순서로 요청된 트랜잭션의 락 요청이 충돌하면 롤백.
2) wound-wait(preempt): 1)과 반대. 더 이후에 요청된 트랜잭션만 기다리기, 이전에 요청된거면 롤백

---
### 데드락 탐지
wait-for graph

V, E
Ti -> Tj인 edge(간선)이 있으면 
Ti가 필요로 하는 데이터를 Tj가 내놓을 때까지 기다려야함.

이 wait-for graph에 사이클이 있다면 교착상태가 발생한 것. 

#### 교착상태로부터 복구

- victim 선택. 어떤 트랜잭션을 롤백할지.
- 전체 롤백보다는 데드락을 풀 정도로만 롤백하는 게 효율적. 
-> 갱신 내용들이 기록되어있어야. 
- starvation 기아 현상. 같은 트랜잭션이 계속 롤백되지 않도록 횟수를 제한.
-> 가장 오래된 트랜잭션을 절대 victim으로 선정하지 않는 방법도.

---
# 다중 세분도(Multiple granularity)
여러 데이터 항목을 그룹으로 묶은 후 그 그룹을 동기화의 단위로.
2PL의 일종이므로 데드락 발생할 수 있음.

어떤 노드에 락을 걸 수 있는지 확인하기 위해 루트부터 전부 탐색해야하는데 비효율적.
-> 락을 걸 때 모든 부모 노드에 intension lock을 걸면 전부 탐색할 필요 없음. (경로 위의 모든 노드에 intension lock)

IS -> 자식노드에 S lock
IX -> 자식노드에 S or X lcok
SIX -> 해당 노드는 S lock + 자식노드를 전부 X lock
: SIX 예시) Table은 공유하되 그 안의 특정 행은 독점적으로 수정하는 경우

lock을 걸 때는 위에서부터, 풀 때는 밑에서부터. 

### Phantom 현상
유령현상.
공통 튜플이 아닌 것 같으나 WHERE절(predicate read 술어 읽기)에서 충돌이 존재해 직렬 불가능한 경우가 되는 현상.

해결책
- where - 읽기 트랜잭션은 해당 데이터 항목에 Shared lock
- 갱신하는 트랜잭션: Exclusive lock.
- 인덱스를 사용하는 트랜잭션은 인덱스에 lock.

#### Index locking
테이블에 접근하는 것을 막는 것은 비효율적이므로 인덱스 잠금이 대안.
but 인덱스의 모든 리프노드를 스캔해야함.
삽입/삭제시 영향을 받는 모든 리프노드에 exclusive lock.
-> 해당 튜플 검색 키값을 포함하는 노드

예시

* **트랜잭션 T1:** 급여가 50,000 이상인 직원 수를 쿼리
* **트랜잭션 T2:** 새로운 직원을 급여 60,000으로 추가

**인덱스 잠금을 사용한 해결:**

인덱스 잠금을 사용하면 위의 팬텀 현상을 방지할 수 있습니다.

1. **T1의 동작:**
    - 급여 50,000 이상인 직원을 찾기 위해 인덱스를 탐색합니다.
    - 탐색하는 과정에서 **관련된 모든 리프 노드에 공유 락(S-mode)을 획득**합니다.  예를 들어 50,000 이상의 값을 가진 모든 리프 노드에 락을 겁니다.
2. **T2의 동작:**
    - 새로운 직원을 추가하려면 인덱스를 업데이트해야 합니다.
    - 급여 60,000은 T1이 락을 걸어둔 범위에 속하므로, T2는 **배타적 락(X-mode)을 획득**해야 합니다.
    - 하지만 T1이 이미 해당 리프 노드에 공유 락을 걸어두었기 때문에, T2는 T1의 락이 해제될 때까지 대기해야 합니다.

이처럼 인덱스 잠금을 사용하면 T1이 쿼리하는 동안 T2가 새로운 직원을 추가하는 영향을 받지 않도록 보장할 수 있습니다.
핵심은 T1이 **조건을 만족하는 범위내 리프노드 전체에 락을 획득**하여, T2가 그 범위에 새로운 값을 삽입하는 것을 막는다는 것입니다.

#### Next-key locking
Next-key는 조건을 만족하는 값들과 그 다음 값만 잠금.

<img width="1130" alt="image" src="https://github.com/user-attachments/assets/6c2697e3-1c5e-4f85-9d14-03c4fc8b3cf9" />

🔴 1. 일반 인덱스 잠금(index-locking)
7 ≤ X ≤ 16 쿼리 시 잠금을 거는 범위는 **검색 조건에 맞는 레코드가 있는 리프 노드 전체**입니다.  
주어진 예시처럼 B+ 트리 리프 노드가 `(3, 5, 8, 11, 14)` 와 `(18, 24, 38, 55)`로 나뉘어 있다면, 7 ≤ X ≤ 16 쿼리는 첫 번째 리프 노드 (3, 5, 8, 11, 14)에만 락을 겁니다. 두 번째 리프 노드는 쿼리와 관련이 없으므로 잠그지 않습니다.

- 즉, 쿼리 범위에 부합하는 레코드가 있는 리프 노드는 *전체를 잠금*하지만, 범위 밖의 리프 노드는 잠그지 않습니다.  
다만, 해당 리프 노드 *전체*를 잠그기 때문에 next-key locking에 비해 동시성이 떨어지는 것은 맞습니다.  예를 들어 다른 트랜잭션이 2를 삽입하려고 할 때, 쿼리 범위(7~16) 밖임에도 불구하고 첫 번째 리프 노드가 잠겨있어 삽입이 블록됩니다.  이러한 문제를 해결하기 위해 next-key locking이 등장했습니다.

🔴 2. next-key locking
7 ≤ X ≤ 16 쿼리에 대해 5, 8, 11, 14 뿐만 아니라 18까지 잠가서 팬텀 현상을 방지.

- 💎만약 5, 8, 11, 14만 잠그고 18을 잠그지 않는다면, 다른 트랜잭션이 15를 삽입할 수 있고, 이는 7 ≤ X ≤ 16 쿼리 결과에 영향을 미치는 팬텀 레코드가 됩니다.

---
# 타임스탬프 기반 규약

트랜잭션들 사이에 미리 순서를 정하는 방법.
1. system clock
2. logical counter

트랜잭션 타임스탬프가 직렬가능성 순서를 결정.
ts(Ti) < ts(Tj) 경우 Ti->Tj 순차 실행과 동등한 결과인 스케줄이 생성되도록 보장함. 

W-ts(Q): W(Q)를 성공적으로 수행한 트랜잭션 타임스탬프 중 가장 큰 것(최근)

ts-ordering protocol
Ti가 read(Q)인 경우
- ts(Ti) < W-ts(Q): Ti는 Q에 덮어쓴 새로운 값을 읽어야 하므로 Ti는 롤백됨.
- ts(Ti) >= W-ts(Q): Ti 수행됨.

Ti가 write(Q)인 경우
- ts(Ti) < R-ts(Q): Ti가 쓰려는 값은 나중에 실행된 트랜잭션이 읽은 값이므로 Ti는 롤백됨.
- ts(Ti) < W-ts(Q): Ti가 쓰려는 값은 나중에 실행된 트랜잭션이 이미 Q를 갱신했으므로 쓸모없는 값이므로 Ti 롤백. 
- ts(Ti) >= R/W-ts(Q)인 경우에만 Ti 수행됨.
-> 쓰기 연산의 경우 더 최근에 수행되어야 accept

롤백되면 새로운 타임스탬프를 Ti에 할당해 재시작.


ts-ordering-protocol(규약)은 충돌 직렬 가능성을 보장함.
(non-conflict 순서 조정시 직렬화 가능)
이미 명확한 순서가 있으므로..

#### Thomas' Write Rule
토마스 쓰기 규칙은 오래된 쓰기 작업을 무시함으로써 불필요한 롤백을 방지하고, 시스템의 동시성을 높임.
쓸모없는 쓰기 연산을 무시함에 따라 충돌 직렬 가능성을 보장하지 못하지만 결과는 올바른 스케줄임.
=> '뷰 직렬 가능'
blind write는 충돌 직렬 가능하지 않은 뷰 직렬 스케줄에서 나타날 수 있다.

원래는 serial 순차 실행이 best지만 느려서
동시 실행하되 lock을 통해 충돌 직렬 가능한 스케줄을 만들었고
(물론 lock도 데드락과 같은 한계가 있음)
그 조차도 예외상황을 두자는.
타임스탬프 시스템이 맞다는 판단하
쓸모없는 쓰기 연산을 무시할 수 있다는 '뷰 직렬 가능' 개념을 만든.
(이전 트랜잭션의 쓰기 연산이 무시되면 read view가 동등)

100% 정합성을 희생하더라도 최종 상태가 크게 다르지 않다면.

#### Optimistic Concurrency control
검증기반 프로토콜.

낙관적 동시성 제어는 트랜잭션들이 서로 충돌할 가능성이 낮다고 가정하고, 트랜잭션 실행 중에는 락을 걸지 않고 실행.
대신 트랜잭션이 완료될 때, 다른 트랜잭션과 충돌이 발생했는지 검증.
-> 락을 거는 동시성 제어보다 동시성을 높일 수 있으나 충돌시 오버헤드 있음. 

검증과정:
Tj가 커밋되기 전 Tj와 동시 실행될 가능성이 있는 ts(Ti) < ts(Tj)인 모든 트랜잭션과 충돌여부 검사.
Tj가 앞선 Ti과 충돌하지 않았음을 보장하기 위한 조건
1) finishTS(Ti) < startTS(Tj): 겹치지 않았으므로 충돌 X
2) startTS(Tj) < finishTS(Ti) < validationTS(Tj): 시간적으로는 겹쳐도 실제 데이터 접근에 있어서는 충돌이 발생하지 않음.
검증 후에 커밋/롤백이라. 

---
### 멀티버전 기법
multiversion concurrency-control

write는 여러 버전을 만드는데 버전 label에 타임스탬프를 붙임.
read(Q) 연산이 들어오면 타입스탬프 기준으로 적절한 버전을 선택. 
이 read는 절대 기다릴 필요없이 즉시 반환할 수 있다.

Qk 버전은 값, W-TS, R-TS를 갖는다. 
Ti가 read(Q), write(Q) 연산을 하려는 경우-
Qk를 TS(Ti)보다 같거나 작은, 가장 큰 write TS 버전이라 해보자.
Ti가 read(Q)할 때:
R-TS(Qk) < TS(Ti)이면 R-TS(Qk) = TS(Ti)

<예시>-------------------------------------------
데이터 항목 Q가 있고, 초기값은 10이라고 가정해보겠습니다.

**예시 1: read 연산**
```
초기상태: Q = 10, W-timestamp(Q) = 0, R-timestamp(Q) = 0

T1(TS=5)이 Q를 읽으려고 함
- Q의 가장 최신 버전(Qk)을 읽음 (값: 10)
- R-timestamp(Q)=0 < TS(T1)=5 이므로
  R-timestamp(Q)를 5로 업데이트

If R-timestamp(Qk) < TS(Ti):
	set R-timestamp(Qk) = TS(Ti)
가장 최근에 읽은 타임스탬프로 업데이트.
```


**예시 2: write 연산의 세 가지 경우**

1. **롤백되는 경우**
```
Q의 상태: 값=10, R-timestamp(Q)=20, W-timestamp(Q)=15

T2(TS=10)이 Q에 20을 쓰려고 함
- TS(T2)=10 < R-timestamp(Q)=20 이므로
- T2는 롤백됨 (이미 더 나중 시점의 트랜잭션이 읽었으므로)
```

2. **덮어쓰는 경우**
가끔 동일 타임스탬프를 재사용하는 경우 타임스탬프가 같을 수는 있지만 흔치않을듯. 어쨌든 실행 타임스탬프가 같으면 버전도 같다. 
```
Q의 상태: 값=10, W-timestamp(Q)=15

T3(TS=15)이 Q에 30을 쓰려고 함
- TS(T3) = W-timestamp(Q)=15 이므로
- Q의 현재 버전을 30으로 덮어씀
```

3. **새 버전을 생성하는 경우**
더 최근 읽기 연산이면 당연히 새 버전.
어쨌든 '새 버전 생성'이란, 나중에 더 작은 타임스탬프를 가진 트랜잭션이 이전 버전을 읽을 수 있게 하기 위함🔴
```
Q의 상태: 값=10, W-timestamp(Q)=15

T4(TS=20)이 Q에 40을 쓰려고 함
- TS(T4)=20 > W-timestamp(Q)=15 이므로
- Q의 새 버전(Qi)을 생성
  * 값 = 40
  * W-timestamp(Qi) = 20
  * R-timestamp(Qi) = 20
```

이렇게 멀티버전을 사용하면:
- 읽기 작업은 자신의 타임스탬프보다 작거나 같은 가장 큰 타임스탬프를 가진 버전을 읽습니다
- 쓰기 작업은 상황에 따라 롤백되거나, 덮어쓰거나, 새 버전을 생성합니다
- 이를 통해 읽기 작업이 쓰기 작업을 기다릴 필요가 없어져 동시성이 향상됩니다

#### 멀티버전 2단계 잠금

read-only와 update 트랜잭션에 차별성을 둔다.
update는 read/write 락을 획득하고, 트랜잭션 마지막까지 모든 락을 들고있는다.(rigorouse 2PL)

예시
```
1. T1 시작: 읽기/쓰기 락 획득
   - Q 읽기: 현재 값 = 10

2. T1이 Q에 20을 쓰려고 함
   - 새 버전 Q1 생성
   - Q1의 값 = 20
   - W-timestamp(Q1) = ∞ (임시로 무한대 설정)
=> 🔴새 버전 생성 시 임시로 W-timestamp를 ∞로 설정하여 
다른 트랜잭션이 커밋 전의 데이터를 읽는 것을 방지

3. T1 커밋 시작 (커밋 처리 과정)
   - ts-counter 값이 100이라고 가정
   - ts-counter에 대해 2단계 락 획득
   - TS(T1) = ts-counter + 1 = 101 설정
   - W-timestamp(Q1) = TS(T1) = 101로 설정
   - ts-counter = 101로 증가
   - 모든 락 해제

결과:
- 원본 Q(값=10)
- 새 버전 Q1(값=20, W-timestamp=101)
```
**"락을 전혀 사용하지 않고도 일관된 데이터를 읽는다"**
🔮 어떻게보면 Lock이라는 제약조건 대신 Timestamp 기준으로 커밋/롤백을 판단함으로써 다른 제약조건을 둔 것이라 볼 수.
Lock은 연산을 하기 전부터 Lock을 획득할 수 있는지 따져보고 바로 롤백하지만
Timestamp protocol에서는 일단 수행하고 따진다. 버전이 기록되어있으면 그걸 바로 읽어오면 되고.. 

읽기 연산은 전혀 기다릴 필요가 없고,
🔴다만 쓰기 연산은 여전히 read/write Lock을 모두 획득해야함.

**예시:**

데이터 항목 Q에 대해 다음과 같은 버전들이 있다고 가정해 봅시다.

* Q0 (값: 10, W-timestamp: 5)
* Q1 (값: 20, W-timestamp: 10)
* Q2 (값: 30, W-timestamp: 15)


만약 읽기 전용 트랜잭션 T(시작 시간: 12)가 Q를 읽으려고 한다면, T는 Q1(값: 20, W-timestamp: 10)을 읽게 됩니다.  T보다 늦게 커밋된 Q2는 읽지 않습니다.  
이때 T는 어떤 락도 획득하지 않으며, 다른 트랜잭션과의 충돌 없이 데이터를 읽을 수 있습니다.

---
# 스냅샷 고립 (= Read committed)

읽기는 절대 block되지 않음.
단 쓰기는 commit을 먼저 한 트랜잭션의 값으로 써짐.

lost update는 없지만 롤백으로 해결하는거라..
트랜잭션이 커밋할 때 다른 트랜잭션이 이미 해당 데이터를 수정한 경우 커밋이 실패하고 롤백.

쓰기 이상현상이 있을수 있으나 보통 Snaption Isol에선 abort를 할 것이므로 팬텀현상은 드물다.

읽기 이상현상도.
시간순서:
T1: 계좌A에서 100원 출금 (잔액 900원)
T2: 계좌B에 100원 입금 (잔액 1100원)
T3(읽기전용): 두 계좌의 잔액 확인
-> T3가 1,2 중간에 일어나면 비일관적.


### 💎 Serializable 스냅샷 고립

🔴 스냅샷 고립도 일단 실행하고 보는. (lock과 달리)
우선순위 그래프를 따져서 롤백. 
-> 이때 read/write 충돌만 보면 됨. write/write는 first committer가 쓰는거라 해결되지만.

개발자는 쿼리시 `select for update`로 스냅샷 고립의 한계점을 보완할 수 있음. 
읽기 시작할 때부터 애초에 쓰기 lock을 걸어버리는 것. 
-> 이런 문제가 발생할 수 있음을 인지.

동시성 제어를 위해 락, 타임스탬프, 스냅샷 고립 등이 쓰이는 것.

Postgres는 인덱스 잠금으로 팬텀현상 방지.

---
# Weak Levels of concurrency

SQL에서의 약한 일관성
- read lock이 유지되나
팬텀현상은 막을 수 없다. 
- read committed: 2 degree consistency. 커밋된거면 읽음(도중에 바뀌더라도)
🔴 대부분 시스템은 cursor-stability로 구현됨. 락을 즉시 푸는.
-> 대부분이 read committed.


##### 사용자 상호작용 동시성 제어
튜플마다 버전이 있는데
update문은 읽은 버전이 같을 때만 업데이트 할 수 있음. 



---
# ADVANCED TOPICS IN CONCURRENCY CONTROL

온라인 인덱스 구축.

업데이트가 진행중인 릴레이션에
2PL을 걸면 모든 업데이트를 막아버리므로 
**스냅샷에 인덱스를 구축하되, 업데이트분에 대한 인덱스는 그다음 캐치업.**
스냅샷 인덱스 구축이 완료됐을 때 락을 획득. (구축하기전에 획득하는게 아니라) 
그동안 업데이트도 또 팔로업 해주고.
-> ALTER TALBLE도 이런식으로 동작할듯






---

시간 순서대로 ABA 문제를 도식화하겠습니다:

1. 초기 상태:
```
head -> N1 -> N2 -> N3
```

2. P1이 삭제 연산을 시작:
```
head -> N1 -> N2 -> N3
P1: oldhead = N1
P1: newhead = N2
(P1이 CAS 실행 전 일시 중단)
```

3. P2가 N1, N2 삭제:
```
head -> N3
(N1, N2는 삭제됨)
```

4. P2가 N1을 새로운 head로 재삽입:
```
head -> N1 -> N3
(N2는 여전히 삭제된 상태)
```

5. P1이 다시 실행되어 CAS 수행:
```
CAS(head, N1, N2) 실행
(head가 여전히 N1이므로 CAS 성공)
```

6. 최종 결과 (문제 상황):
```
head -> N2 (삭제된 노드) -> ???
(잘못된 상태가 됨)
```

올바른 결과였어야 할 상태:
```
head -> N1 -> N3
```

이 도식화는 ABA 문제가 어떻게 발생하는지, 그리고 왜 위험한지를 보여줍니다. P1은 자신이 수정하려는 노드가 그동안 변경되었다는 것을 알지 못하고 작업을 수행하여 데이터 구조가 손상되는 결과를 초래합니다.


---

특수목적의 Lock.
Increment는 리턴값을 반환하지 않으므로
**증분은 교환법칙이 성립함.**
S, I끼리만 락을 공유할 수.
Increment는 write임에도 공유 가능. 

티켓이 많을때는 순서 재정렬이 일어나도 괜찮지만 
마지막 한 티켓만 남았을 때는 안됨.
조건부로 허용한다는 말.

---

실시간 트랜잭션 시스템의 세 가지 마감시간 유형을 실제 예시로 설명하겠습니다:

1. Hard Deadline (엄격한 마감시간):
- 예시: 항공기 관제 시스템
- 비행기의 위치 데이터는 정확한 시간에 처리되어야 함
- 마감시간을 놓치면 심각한 안전 문제 발생
- 마감시간 초과 = 시스템 실패

2. Firm Deadline (확고한 마감시간):
- 예시: 주식 거래 시스템
- 특정 가격에 주문을 넣을 때 마감시간이 있음
- 마감시간 이후에는 주문이 무효화됨 (가치=0)
- 늦은 주문은 아예 실행되지 않음

3. Soft Deadline (유연한 마감시간):
- 예시: 온라인 쇼핑몰의 할인 쿠폰
- 블랙프라이데이 특별 할인 기간
- 마감시간 후에도 주문은 가능하나 할인율이 감소
- 정상가로 구매 가능 (일부 가치 유지)

이러한 마감시간은 시스템의 특성과 요구사항에 따라 적절히 선택되어 적용됩니다.





---
## 실전문제

1. 
2PL이 직렬 가능함을 증명
귀류법
- 2PL이지만 직렬화 불가능한 스케줄이 존재한다고 가정
- 직렬화 불가능한 스케줄: precedence 그래프에 사이클이 존재

2PL의 특성:
-  αi는 Ti가 마지막 락을 획득하는 시점
Ti -> Tj에서는 항상 αi < αj

모순 도출
- 사이클이 존재한다면 α0 < α1 < α2 < ... < αn-1 < α0
하지만 α0 < α0는 모순
따라서 사이클은 존재할 수 없음

결론
- 2PL은 사이클을 만들 수 없음. 즉 비직렬화 스케줄을 만들 수 없음. 

##### 락 포인트 기준으로 트랜잭션 정렬 예시
α3(2) < α1(3) < α2(4)
따라서 이 트랜잭션들의 직렬화 가능한 실행 순서는: T3 → T1 → T2
```
T1: 
- t=1: x에 대한 shared lock 획득
- t=3: y에 대한 exclusive lock 획득
- t=5: 모든 락 해제
락 포인트 α1 = 3 (마지막 락 획득 시점)

T2:
- t=2: z에 대한 shared lock 획득
- t=4: x에 대한 exclusive lock 획득
- t=6: 모든 락 해제
락 포인트 α2 = 4

T3:
- t=1: y에 대한 shared lock 획득
- t=2: z에 대한 exclusive lock 획득
- t=7: 모든 락 해제
락 포인트 α3 = 2
```
실제 실행에선 동시성이 유지됨.
아래 예시에서는
락 포인트 순서: T3(2) → T1(3) → T2(4) 

2PL은 직렬가능성을 보장하지만, 동시성을 최대한 유지하는 것이 목표가 아닙니다.2PL에서 락 포인트 정렬은 직렬화 가능한 순서를 찾는 방법 중 하나일 뿐

```c
시간    T1              T2              T3
1       SL(x)                          SL(y)
2                       SL(z)          XL(z)
3       XL(y)          
4                       XL(x)          
5       unlock(x,y)
6                       unlock(x,z)
7                                      unlock(y,z)
```

- 직렬 가능하다 = 우선순위 그래프에서 사이클이 없다. (우선순위 그래프: 충돌하는 연산 기준)
- 2PL = 트랜잭션들이 모두 증가 또는 감소를 유지하기 때문에 
트랜잭션들간 락포인트 기준으로 정렬 가능하다

위 예시를 엄밀히 동시 실행되는 형태로 다시 보여주자면
```c
실제 동시 실행 (2PL 적용):
시간    T1              T2              T3
1       SL(x)                          SL(y)
2                                      XL(z)
3       wait_XL(y)          
4                       wait_SL(z)     
5                                      unlock(y,z)
6       XL(y)
7                       SL(z)
8                       XL(x)          
9       unlock(x,y)
10                      unlock(x,z)
```

