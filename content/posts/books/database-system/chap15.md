+++
categories = ['Database']
series = ['Database-System']
tags = ['데이터베이스']
date = 2024-08-04T22:50:44+09:00
draft = true
+++

# Chap 15. Query Processing

쿼리 실행 엔진은 query-evalution plan을 받아 실행하고 결과를 알려준다.

#### 쿼리 프로세싱
비용이 가장 적게 드는 방법으로.
statistical information 사용. 튜플 개수, 크기 등.
- 쿼리 비용 측정방법
- 관계대수 평가 알고리즘
- 각 operation을 어떻게 알고리즘을 조합해서 complete expressoin을 만드는지
그 다음에 쿼리 최적화를 배운다.

#### 쿼리 비용 측정
SSD로 인해 IO 비용의 지배력이 내려가고, 따라서 실제 시스템에선 CPU 비용을 반드시 포함하지만-
지금은 단순성을 위해 제외하고 보자.

##### 디스크 비용
- seeks
- blocks read
- blocks written

디스크로터 이동한 블록 개수(read + write), number of seeks를 사용하겠다.

어떤 알고리즘은 버퍼공간을 이용해 diskIO를 줄이기도 한다.
worst case는 버퍼에 어떤 데이터도 없고 최소한의 메모리만 가능한 경우다.

#### Selection operation
File scan
선형검색. 각 파일블록을 스캔하고 조건을 만족하는지 보는.

선형검색은 파일순서, 인덱스 여부, 선택 속성과 상관없어 어떤 파일에도 적용된다.
바이너리 서치(또는 훨씬 나은 index serach)는 인덱스 조건이 맞을때만 가능하다. 

#### 인덱스를 사용한 Selection
인덱스 스캔
(clustering index, equality on key): 하나의 레코드를 반환. 


---
## 가정

10억개의 row가 있는 테이블에는 몇 개의 블록이 있는가?
한 row당 100 byte라 치면
1페이지 4096/100 byte = 40row per page

10억개 row / 40 row = 2400만개 블록.

M은?
16GB 램에 몇 개의 블록이 올라가는가?
16 * 10^9 / 4KB = 4백만개 블록.

전체 br = 24 million
M = 4 million
1번의 pass에 6개의 run이 생김.
ㅋㅋ 뭔가 한번에 되는데?

      


---
# 실전문제

15.2
```sql
select branch_name 
from branch T
JOIN (
	select assets
	from branch
	where branch_city = "Brooklyn"
) S ON T.assets > S.assets;

// 응용
select user_id 
from (
	select user_id, COUNT(keyword_id) AS count from user_tracking_keywords
	group by user_id
) T
JOIN (
	select COUNT(keyword_id) AS count from user_tracking_keywords
	where user_id = 2
) S ON T.count > S.count;
```
```
IN (1,2) 으로 해도 둘 중 큰 count인 userId 2 기준으로 더 큰 튜플만 나옴.
확실히 T의 한 튜플당 S의 모든 튜플을 보는게 맞나봄.
💎💎그만큼 JOIN은 부담스러운 연산이기에 inner 테이블인 S의 크기를 최대한 줄여서 가져가야함.
```

이 관계대수 표현식은 주어진 SQL 쿼리를 효율적으로 수행하기 위한 것입니다. 각 기호의 의미는 다음과 같습니다:

- \(\Pi\): 프로젝션 연산자로, 필요한 속성만 선택합니다. (select)
- \(\sigma\): 선택 연산자로, 조건에 맞는 튜플만 선택합니다. (where)
- \(\rho\): 리네이밍 연산자로, 테이블이나 속성의 이름을 변경합니다.
- \(\Join\): 조인 연산자로, 두 테이블을 결합합니다. 여기서는 세타 조인(\(\Join_{\theta}\))을 사용하여 특정 조건에 맞는 튜플을 결합합니다.

이 표현식은 다음과 같은 단계로 이루어져 있습니다:

1. \(\rho_T(branch)\)와 \(\rho_S(branch)\): 두 테이블을 각각 T와 S로 리네밍합니다.
2. \(\Pi_{branch\_name, assets}(\rho_T(branch))\): T 테이블에서 branch_name과 assets 속성만 선택합니다.
3. \(\sigma_{(branch\_city = 'Brooklyn')}(\rho_S(branch))\): S 테이블에서 branch_city가 'Brooklyn'인 튜플만 선택합니다.
4. \(\Pi_{assets}(\ldots)\): S 테이블에서 assets 속성만 선택합니다.
5. \(\Join_{T.assets > S.assets}(\ldots)\): T와 S 테이블을 assets 속성을 기준으로 세타 조인합니다.
6. \(\Pi_{T.branch\_name}(\ldots)\): 최종 결과에서 T 테이블의 branch_name 속성만 선택합니다.

이 과정은 불필요한 데이터를 줄이고, 필요한 데이터만을 효율적으로 추출하기 위한 것입니다.

---
7.
```js
class IndexedNLJoin {
    Iterator outer;
    Iterator inner;
    Tuple t_r; // current outer tuple
    boolean done; // flag for end of outer scan

    public void open() {
        outer.open();
        inner.open();
        done = false;
        
        if (outer.next() != null) {
            t_r = outer.getCurrentTuple();
        } else {
            done = true;
        }
    }

    public boolean next() {
        while (!done) {
            if (inner.next(t_r.getJoinAttributes()) != null) {
                Tuple t_s = inner.getCurrentTuple();
                // Join t_r and t_s and place in output buffer
                outputBuffer.add(concatenate(t_r, t_s));
                return true;
            } else {
                if (outer.next() != null) {
                    t_r = outer.getCurrentTuple();
                    inner.rewind();
                } else {
                    done = true;
                }
            }
        }
        return false;
    }

    private Tuple concatenate(Tuple t_r, Tuple t_s) {
        // Implement tuple concatenation logic
        return new Tuple(t_r, t_s);
    }
}
```


실제 데이터베이스 예시로 Indexed Nested Loop Join을 설명하겠습니다:

```sql
-- 예시 테이블
Employees (emp_id, name, dept_id)  -- outer relation
Departments (dept_id, dept_name)   -- inner relation with index on dept_id
```

실행 과정:
```java
// 1. 초기화 (open())
IndexedNLJoin join = new IndexedNLJoin();
join.open();
    outer.open();  // Employees 테이블 스캔 시작
    inner.open();  // Departments 인덱스 준비
    // emp_id=1, name="John", dept_id=10 첫 번째 튜플 읽음

// 2. 조인 실행 (next())
while (join.next()) {
    // First iteration:
    // outer tuple: (1, "John", 10)
    inner.next(10)  // dept_id=10인 Department 검색
        // 인덱스 사용하여 dept_id=10인 튜플 찾음
        // (10, "IT") 발견
        // 결과: (1, "John", 10, "IT")
    
    // Second iteration:
    // outer tuple: (2, "Jane", 20)
    inner.next(20)  // dept_id=20인 Department 검색
        // (20, "HR") 발견
        // 결과: (2, "Jane", 20, "HR")
    
    // 계속...
}
```

실제 실행 결과:
```sql
-- 입력 데이터
Employees:
emp_id  name    dept_id
1       John    10
2       Jane    20
3       Bob     10

Departments:
dept_id dept_name
10      IT
20      HR

-- 조인 결과
emp_id  name    dept_id  dept_name
1       John    10       IT
2       Jane    20       HR
3       Bob     10       IT
```

주요 특징:
1. Departments 테이블의 dept_id에 인덱스가 있어 빠른 검색 가능
2. 각 Employee 레코드에 대해 인덱스를 통해 매칭되는 Department를 찾음
3. 인덱스 검색으로 전체 테이블 스캔 회피
4. 특히 inner relation이 큰 경우 효율적


---
8.

데이터베이스에서의 나눗셈 연산은 특정 조건을 만족하는 튜플을 찾기 위해 사용됩니다. 주로 두 개의 관계 \( r \)과 \( s \)가 있을 때, \( r \div s \)는 \( r \)의 \( T \) 속성 값 중 \( s \)의 모든 \( S \) 값을 포함하는 \( T \) 값을 찾는 연산입니다.

### 예시

#### 관계 \( r \)
| T | S |
|---|---|
| A | 1 |
| A | 2 |
| B | 1 |
| B | 2 |
| C | 1 |

#### 관계 \( s \)
| S |
|---|
| 1 |
| 2 |

### 나눗셈 연산 \( r \div s \)

- **목표**: \( s \)의 모든 \( S \) 값을 포함하는 \( T \) 값을 찾기.
- **결과**: \( T = A, B \)

#### 설명
- \( T = A \)는 \( S = 1, 2 \)를 모두 포함하므로 결과에 포함됩니다.
- \( T = B \)도 \( S = 1, 2 \)를 모두 포함하므로 결과에 포함됩니다.
- \( T = C \)는 \( S = 2 \)를 포함하지 않으므로 결과에 포함되지 않습니다.

이렇게 나눗셈 연산은 특정 조건을 만족하는 \( T \) 값을 찾는 데 유용합니다.

---
이 문제는 관계형 데이터베이스에서의 나눗셈 연산을 수행하기 위한 정렬 기반 및 해시 기반 알고리즘을 설계하는 것입니다. 

### 정렬 기반 알고리즘

1. **정렬**: 
   - 관계 \( r(T \cup S) \)와 \( s(S) \)를 각각 정렬합니다. \( r \)는 \( (T, S) \)로 정렬하고, \( s \)는 \( S \)로 정렬합니다.

2. **스캔 및 비교**:
   - \( r \)를 스캔하면서 첫 번째 튜플의 \( T \) 속성 값을 확인합니다. 동일한 \( T \) 값을 가진 튜플들을 모두 스캔합니다.
   - 동시에 \( s \)를 스캔하여 \( r \)의 \( S \) 속성 값과 일치하는지 확인합니다. 이는 병합 조인과 유사합니다.
   - 모든 \( s \)의 튜플이 \( r \)의 \( S \) 속성으로 존재하면 해당 \( T \) 값을 출력합니다.

3. **디스크 접근**:
   - 정렬 후 디스크 접근 횟수는 \( |r| + N \times |s| \)입니다. 여기서 \( N \)은 \( r \)의 서로 다른 \( T \) 값의 개수입니다.

### 해시 기반 알고리즘

1. **메모리 내 파티션**:
   - \( r \)를 \( T \) 속성으로 파티션하여 각 파티션이 메모리에 적합하도록 합니다.

2. **해시 테이블 생성**:
   - 각 파티션을 처리하면서 모든 서로 다른 \( T \) 값을 별도의 해시 테이블에 수집합니다.

3. **검사 및 출력**:
   - 각 \( T \) 값에 대해, \( s \)의 각 값과 해시 테이블을 비교합니다.
   - 값이 없으면 해당 \( T \) 값을 버리고, 있으면 출력합니다.

이 알고리즘들은 관계형 데이터베이스에서 나눗셈 연산을 효율적으로 수행하기 위한 방법을 제시합니다. 정렬 기반 알고리즘은 정렬과 병합을 통해, 해시 기반 알고리은 해시 테이블을 통해 연산을 수행합니다.

---