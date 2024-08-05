+++
categories = ['Database']
series = ['Database-System']
tags = ['데이터베이스']
title = "TypeOrm"
date = 2024-07-14T22:50:44+09:00
draft = true
+++
## E-R 모델을 사용한 데이터베이스 설계

### 6.1
"premium_payment"와 "policy" 간의 관계를 이해하기 위해서는 각 "policy"가 여러 개의 "premium_payment"를 가질 수 있는지, 아니면 각 "premium_payment"가 여러 개의 "policy"에 속할 수 있는지를 알아야 합니다.

일반적으로 보험 정책(policy)와 보험료 지불(premium_payment) 간의 관계는 다음과 같이 이해될 수 있습니다:

- **일대다(1:N) 관계**: 
  - 하나의 "policy"는 여러 개의 "premium_payment"를 가질 수 있습니다. 예를 들어, 한 보험 정책에 대해 매월 또는 매년 보험료를 지불할 수 있습니다. 따라서 하나의 "policy"에 대해 여러 번의 "premium_payment"가 발생할 수 있습니다.
  - 반대로, 하나의 "premium_payment"는 단 하나의 "policy"에 속합니다. 특정 보험료 지불은 특정 보험 정책과만 연관됩니다.

따라서, "premium_payment"와 "policy" 간의 관계는 일대다(1:N) 관계입니다. 이를 다이어그램으로 표현하면, "policy"가 1이고 "premium_payment"가 N입니다.

### 예시
#### `policy` 테이블
| policy_id | policy_holder |
|-----------|---------------|
| 101       | John Doe      |
| 102       | Jane Smith    |

#### `premium_payment` 테이블
| payment_no | policy_id | amount | payment_date |
|------------|-----------|--------|--------------|
| 1          | 101       | 100    | 2023-01-01   |
| 2          | 101       | 100    | 2023-02-01   |
| 3          | 102       | 200    | 2023-01-15   |
| 4          | 101       | 100    | 2023-03-01   |

위 예시에서 볼 수 있듯이, 하나의 "policy_id"가 여러 개의 "premium_payment"를 가질 수 있습니다. 예를 들어, "policy_id"가 101인 경우, 세 번의 "premium_payment"가 있습니다. 이는 "premium_payment"와 "policy" 간의 관계가 일대다(1:N) 관계임을 보여줍니다.

---
### 6.2
a. 
삼진관계(trinary relationship)는 세 개의 엔티티 간의 관계를 나타내는 것을 의미합니다. 여기서 "exam_marks" 관계는 "student", "exam", "section" 세 개의 엔티티 간의 관계를 나타내고 있습니다. 즉, 한 학생이 특정 시험을 특정 섹션에서 본다는 것을 의미합니다.

해설에서 말하는 대안은 "exam_marks" 관계를 이진관계(binary relationship)로 단순화하는 것입니다. 이 경우, "exam_marks" 관계는 "student"와 "exam" 간의 관계로만 모델링되고, "section"은 별도의 관계로 처리됩니다. 이를 위해 "exam"을 약한 엔티티(weak entity)로 모델링하고, "section"과의 관계를 통해 "exam"을 식별할 수 있도록 합니다.

예시를 들어 설명하겠습니다:

### 현재 모델 (삼진관계)
- **Entities**:
  - student(student_id, name, dept_name, tot_cred)
  - exam(exam_id, name, place, time)
  - section(sec_id, semester, year)
- **Relationship**:
  - exam_marks(student_id, exam_id, sec_id, marks)

### 대안 모델 (이진관계)
- **Entities**:
  - student(student_id, name, dept_name, tot_cred)
  - section(sec_id, semester, year)
  - exam(exam_id, name, place, time, sec_id)  // exam이 약한 엔티티로 section에 종속됨
- **Relationship**:
  - exam_marks(student_id, exam_id, marks)

이 대안 모델에서는 "exam"이 "section"에 종속된 약한 엔티티로 모델링되며, "exam_marks" 관계는 "student"와 "exam" 간의 이진 관계로 단순화됩니다. 이를 통해 데이터 중복을 줄이고, 각 시험이 어떤 섹션에서 이루어졌는지 명확히 할 수 있습니다.

---
a. 
대안 모델(이진관계)에서 하나의 `exam_id`가 여러 섹션에 걸쳐 진행될 수 있도록 모델링할 수 있습니다. 이를 위해 `exam` 엔티티가 `section` 엔티티와의 관계를 통해 종속되는 방식으로 설계합니다. 

### 대안 모델 구조
- **Entities**:
  - `student(student_id, name, dept_name, tot_cred)`
  - `section(sec_id, semester, year)`
  - `exam(exam_id, name, place, time, sec_id)`  // `exam`이 약한 엔티티로 `section`에 종속됨
- **Relationship**:
  - `exam_marks(student_id, exam_id, marks)`

### 예시 테이블

#### `student` 테이블
| student_id | name       | dept_name | tot_cred |
|------------|------------|-----------|----------|
| 1          | Alice      | CS        | 30       |
| 2          | Bob        | Math      | 45       |

#### `section` 테이블
| sec_id | semester | year |
|--------|----------|------|
| 101    | Fall     | 2023 |
| 102    | Spring   | 2023 |

#### `exam` 테이블
엄밀히 말하면 weak entity가 되려면 exam_id PK 없이 나머지 정보들만 있는 경우임. name, place만으로는 고유하게 식별할 수 없는 경우.
| exam_id | name         | place   | time       | sec_id |
|---------|--------------|---------|------------|--------|
| 201     | Midterm      | Room 1  | 2023-10-10 | 101    |
| 202     | Final        | Room 2  | 2023-12-15 | 101    |
| 203     | Midterm      | Room 3  | 2023-10-11 | 102    |
| 204     | Final        | Room 4  | 2023-12-16 | 102    |

#### `exam_marks` 테이블
| student_id | exam_id | marks |
|------------|---------|-------|
| 1          | 201     | 85    |
| 2          | 201     | 90    |
| 1          | 202     | 88    |
| 2          | 202     | 92    |
| 1          | 203     | 80    |
| 2          | 203     | 85    |
| 1          | 204     | 82    |
| 2          | 204     | 89    |

### 설명
- `exam` 테이블에서 `exam_id`는 특정 `section`에 종속됩니다. 예를 들어, `exam_id` 201과 202는 `sec_id` 101에 속하며, `exam_id` 203과 204는 `sec_id` 102에 속합니다.
- `exam_marks` 테이블은 `student_id`와 `exam_id`를 통해 학생이 특정 시험에서 받은 점수를 기록합니다.

이 대안 모델은 유효한 경우로, 동일한 시험 이름(`Midterm`, `Final`)이 여러 섹션에서 진행될 수 있음을 반영합니다. 이 모델은 시험이 특정 섹션에 종속되도록 하여, 시험과 섹션 간의 종속 관계를 명확히 합니다.

---
## 6.3
played: 
- matchId
- playerId
- score

season_score 구하기
```sql
update player p
join (
  select player_id, sum(score) as total_score
  from played
  group by player_id
) psum on p.player_id = psum.player_id
set p.session_score = psum.total_score;

```
SQL에서 `UPDATE` 문에 `JOIN`을 사���하는 것은 특정 조건에 맞는 행을 업데이트할 때 유용합니다. 일반적인 `UPDATE` 문법과는 다르게, `JOIN`을 사용하여 다른 테이블이나 서브쿼리의 결과와 결합하여 업데이트할 수 있습니다. 이 방법은 특히 집계 함수와 함께 사용할 때 유용합니다.

### SQL 쿼리 설명
아래의 SQL 쿼리는 `player` 테이블의 `season_score`를 `played` 테이블에서 각 `player`의 `score`를 합산하여 업데이트합니다.

```sql
UPDATE player p
JOIN (
  SELECT player_id, SUM(score) AS total_score
  FROM played
  GROUP BY player_id
) psum ON p.player_id = psum.player_id
SET p.season_score = psum.total_score;
```

### 쿼리의 작동 방식
1. **서브쿼리**: 
   - `played` 테이블에서 각 `player_id`별로 `score`를 합산합니다.
   - 결과는 `player_id`와 `total_score`를 포함하는 임시 테이블 `psum`입니다.

2. **JOIN**:
   - `player` 테이블과 서브쿼리 `psum`을 `player_id`를 기준으로 조인합니다.
   - `player` 테이블의 각 행은 `psum`의 해당 행과 결합됩니다.

3. **UPDATE**:
   - `player` 테이블의 `season_score`를 `psum.total_score` 값으로 업데이트합니다.

### 예시 데이터

#### `player` 테이블 (업데이트 전)
| player_id | name  | season_score |
|-----------|-------|--------------|
| 1         | Alice | 0            |
| 2         | Bob   | 0            |

#### `played` 테이블
| player_id | match_id | score |
|-----------|----------|-------|
| 1         | 101      | 10    |
| 1         | 102      | 15    |
| 2         | 101      | 20    |
| 2         | 102      | 25    |

### 실행 결과

#### `player` 테이블 (업데이트 후)
| player_id | name  | season_score |
|-----------|-------|--------------|
| 1         | Alice | 25           |
| 2         | Bob   | 45           |

### 설명
- **서브쿼리 결과**:
  - `player_id`가 1인 경우: `SUM(score)` = 10 + 15 = 25
  - `player_id`가 2인 경우: `SUM(score)` = 20 + 25 = 45

- **업데이트 결과**:
  - Alice의 `season_score`는 25로 업데이트됩니다.
  - Bob의 `season_score`는 45로 업데이트됩니다.

이 쿼리는 `JOIN`을 사용하여 `player` 테이블을 서브쿼리의 결과와 결합하고, `player` 테이블의 `season_score`를 서브쿼리에서 계산된 `total_score`로 업데이트합니다. 이 방법은 여러 테이블 간의 복잡한 업데이트를 수행할 때 유용합니다.

---
## 6.6

그림 6.29의 b에서 모든 튜플은 E를 통해 어떤식으로든 A,B,C간에 연결이 되어있어야 한다는 것인데,
이 제약조건을 주기 위해 
b의 경우 E의 모든 튜플이 A,B,C와 관계를 가지도록 제약한다.
```sql
-- E 엔터티 테이블
CREATE TABLE E (
    id INT PRIMARY KEY,
    attribute1 VARCHAR(100),
    attribute2 VARCHAR(100)
);

-- A 엔터티 테이블
CREATE TABLE A (
    id INT PRIMARY KEY,
    attribute1 VARCHAR(100)
);

-- B 엔터티 테이블
CREATE TABLE B (
    id INT PRIMARY KEY,
    attribute1 VARCHAR(100)
);

-- C 엔터티 테이블
CREATE TABLE C (
    id INT PRIMARY KEY,
    attribute1 VARCHAR(100)
);

-- R_A 관계 테이블
CREATE TABLE R_A (
    e_id INT,
    a_id INT,
    PRIMARY KEY (e_id, a_id),
    FOREIGN KEY (e_id) REFERENCES E(id),
    FOREIGN KEY (a_id) REFERENCES A(id)
);

-- R_B 관계 테이블
CREATE TABLE R_B (
    e_id INT,
    b_id INT,
    PRIMARY KEY (e_id, b_id),
    FOREIGN KEY (e_id) REFERENCES E(id),
    FOREIGN KEY (b_id) REFERENCES B(id)
);

-- R_C 관계 테이블
CREATE TABLE R_C (
    e_id INT,
    c_id INT,
    PRIMARY KEY (e_id, c_id),
    FOREIGN KEY (e_id) REFERENCES E(id),
    FOREIGN KEY (c_id) REFERENCES C(id)
);

-- E 엔터티의 모든 튜플이 R_A, R_B, R_C에 포함되도록 하는 제약사항
ALTER TABLE E ADD CONSTRAINT fk_e_to_ra
FOREIGN KEY (id) REFERENCES R_A(e_id);
ALTER TABLE E ADD CONSTRAINT fk_e_to_rb
FOREIGN KEY (id) REFERENCES R_B(e_id);
ALTER TABLE E ADD CONSTRAINT fk_e_to_rc
FOREIGN KEY (id) REFERENCES R_C(e_id);
```


c번의 경우 아예 R에 A-B-C-E 모든 관계가 포함되도록 한다.
```sql
-- 삼항 관계 테이블
CREATE TABLE R (
    e_id INT,
    a_id INT,
    b_id INT,
    c_id INT,
    PRIMARY KEY (e_id, a_id, b_id, c_id),
    FOREIGN KEY (e_id) REFERENCES E(id),
    FOREIGN KEY (a_id) REFERENCES A(id),
    FOREIGN KEY (b_id) REFERENCES B(id),
    FOREIGN KEY (c_id) REFERENCES C(id)
);

-- E 엔터티의 모든 튜플이 R에 포함되도록 하는 제약사항
ALTER TABLE E ADD CONSTRAINT fk_e_to_r
FOREIGN KEY (id) REFERENCES R(e_id);

-- A 엔터티의 모든 튜플이 R에 포함되도록 하는 제약사항
ALTER TABLE A ADD CONSTRAINT fk_a_to_r
FOREIGN KEY (id) REFERENCES R(a_id);

-- B 엔터티의 모든 튜플이 R에 포함되도록 하는 제약사항
ALTER TABLE B ADD CONSTRAINT fk_b_to_r
FOREIGN KEY (id) REFERENCES R(b_id);

-- C 엔터티의 모든 튜플이 R에 포함되도록 하는 제약사항
ALTER TABLE C ADD CONSTRAINT fk_c_to_r
FOREIGN KEY (id) REFERENCES R(c_id);
```


---
## 6.7
weak entity set에 PK 속성을 추가함으로써 strong entity set이 될 수 있는데, 여기서 발생하는 중복 문제는?
- strong entity set과의 관계에서 weak entity set의 PK를 추론할 수 있는데. PK를 실제 추가해버리게 되면 두 개의 enity set에 나타나게 됨.

(예시)
#### 강한 엔티티 집합: `부서(Department)`
- `부서ID(DepartmentID)` (기본 키)
- `부서이름(DepartmentName)`

#### 약한 엔티티 집합: `직원(Employee)`
- `직원ID(EmployeeID)` (부분 키)
- `직원이름(EmployeeName)`
- `부서ID(DepartmentID)` (외래 키, `부서`와의 관계를 나타냄)

#### 관계: `소속(BelongsTo)`
- `부서ID(DepartmentID)`
- `직원ID(EmployeeID)`


### 해결 방법

중복을 피하기 위해서는 약한 엔티티 집합에 강한 엔티티 집합의 기본 키 속성을 추가하지 않거나, 관계 집합을 제거하여 중복을 방지해야 합니다. 예를 들어, `직원` 엔티티 집합에 `부서ID`를 추가한 경우, `소속` 관계 집합을 제거하면 중복 문제가 해결됩니다.

---
## 6.8
이 문제는 데이터베이스 관계에서 일대다 (many-to-one) 관계를 설명하고, 기본 키와 외래 키 제약 조건이 이러한 일대다 관계 제약 조건을 어떻게 보장하는지 묻고 있습니다. 

### 문제 설명
`sec_course`라는 관계가 주어졌습니다. 이 관계는 일대다 관계에서 생성된 것입니다. 여기서 기본 키와 외래 키 제약 조건이 일대다 관계 제약 조건을 보장하는지 설명해야 합니다.

### 답변 설명
1. **기본 키와 외래 키 제약 조건**:
   - `section` 테이블의 기본 키는 `(course_id, sec_id, semester, year)`입니다.
   - 이 기본 키는 `sec_course` 테이블의 기본 키이기도 합니다.
   - `course_id`는 `sec_course`에서 `course`를 참조하는 외래 키입니다.

2. **일대다 관계 보장**:
   - 이러한 제약 조건은 특정 `section`이 하나의 `course`에만 대응될 수 있도록 보장합니다.
   - 따라서, 일대다 관계 제약 조건이 보장됩니다.

3. **전체 참여 제약 조건**:
   - 그러나 이러한 제약 조건은 전체 참여 제약 조건을 보장하지 않습니다.
   - 즉, 모든 `course`나 `section`이 반드시 `sec_course` 관계에 참여해야 한다는 것을 보장하지 않습니다.

### 요약
기본 키와 외래 키 제약 조건은 특정 `section`이 하나의 `course`에만 대응될 수 있도록 보장하여 일대다 관계 제약 조건을 보장합니다. 그러나 전체 참여 제약 조건은 보장하지 않습니다.

---
## 6.9
여기서 설명한 내용은 `advisor` 관계 집합이 일대일 관계를 가지도록 하기 위해 추가적인 제약 조건을 설정하는 방법입니다. 이를 위해 `s_ID`를 기본 키로 선언하고, `i_ID`를 고유 키로 선언해야 합니다. 이를 SQL로 구현한 예시와 테이블 구조를 보여드리겠습니다.

### 테이블 구조 예시

```sql
CREATE TABLE advisor (
    s_ID INT PRIMARY KEY,
    i_ID INT UNIQUE
);
```

### 예시 데이터 삽입

```sql
INSERT INTO advisor (s_ID, i_ID) VALUES (1, 101);
INSERT INTO advisor (s_ID, i_ID) VALUES (2, 102);
-- 다음과 같은 데이터 삽입은 고유 제약 조건에 의해 실패합니다.
-- INSERT INTO advisor (s_ID, i_ID) VALUES (3, 101); -- i_ID가 고유해야 하므로 실패
-- INSERT INTO advisor (s_ID, i_ID) VALUES (1, 103); -- s_ID가 기본 키이므로 실패
```

이렇게 하면 `advisor` 테이블에서 `s_ID`와 `i_ID`가 각각 고유하게 유지되며, 일대일 관계가 보장됩니다.

---
## 6.10
외래 키 속성은 기본적으로 `NULL` 값을 가질 수 있습니다. 따라서, 외래 키 제약 조건을 사용하여 전체 참여 제약 조건을 강제하려면 추가적인 `NOT NULL` 제약 조건을 명시적으로 설정해야 합니다.

### 문제와 답변의 예시

### 문제:
엔티티 집합 A와 B 사이의 다대일 관계 R을 고려하라. R에서 생성된 관계가 A에서 생성된 관계와 결합된다고 가정하라. SQL에서 외래 키 제약 조건에 참여하는 속성은 `NULL`일 수 있다. R에서 A의 전체 참여 제약 조건을 `NOT NULL` 제약 조건을 사용하여 어떻게 강제할 수 있는지 설명하라.

### 답변:
R에서 B의 기본 키에 해당하는 외래 키 속성은 `NOT NULL`로 설정되어야 한다. 이는 R에서 B의 항목과 관련이 없는 A의 튜플이 R에 들어올 수 없도록 보장한다. 예를 들어, a가 A의 튜플인데 R에서 해당 항목이 없다면, R이 A와 결합될 때 B에 해당하는 외래 키 속성이 `NULL`이 되는데, 이는 허용되지 않는다.

#### 다대일 관계 R (NOT NULL 제약 조건 추가)
```sql
CREATE TABLE R (
    a_id INT,
    b_id INT NOT NULL,  -- NOT NULL 제약 조건 추가
    PRIMARY KEY (a_id, b_id),
    FOREIGN KEY (a_id) REFERENCES A(a_id),
    FOREIGN KEY (b_id) REFERENCES B(b_id)
);
```

이렇게 하면 `R` 테이블에서 `b_id` 속성은 `NULL` 값을 가질 수 없게 되며, 이는 A의 모든 튜플이 반드시 B의 튜플과 연결되어야 함을 보장합니다.

---
## 6.11-a.

### 예시: 다대다 관계

**시나리오:**
학생(Student)과 강의(Course) 사이의 다대다 관계를 고려해보겠습니다. 여기서 한 학생은 여러 강의를 들을 수 있고, 한 강의는 여러 학생이 들을 수 있습니다.

**테이블 구조:**

1. **Student 테이블:**
   - `StudentID` (기본 키)
   - `StudentName`

2. **Course 테이블:**
   - `CourseID` (기본 키)
   - `CourseName`

3. **Enrollment 테이블:** (학생과 강의 사이의 다대다 관계를 나타냄)
   - `StudentID` (외래 키, Student 테이블 참조)
   - `CourseID` (외래 키, Course 테이블 참조)

**테이블 예시:**

```sql
CREATE TABLE Student (
    StudentID INT PRIMARY KEY,
    StudentName VARCHAR(100)
);

CREATE TABLE Course (
    CourseID INT PRIMARY KEY,
    CourseName VARCHAR(100)
);

CREATE TABLE Enrollment (
    StudentID INT,
    CourseID INT,
    FOREIGN KEY (StudentID) REFERENCES Student(StudentID),
    FOREIGN KEY (CourseID) REFERENCES Course(CourseID),
    PRIMARY KEY (StudentID, CourseID)
);
```

**설명:**

- `Enrollment` 테이블은 `StudentID`와 `CourseID`를 기본 키로 가지며, 각각 `Student` 테이블과 `Course` 테이블의 외래 키로 설정되어 있습니다.
- 다대다 관계에서는 특정 강의를 듣는 모든 학생을 `Enrollment` 테이블에서 확인할 수 있습니다.

**전체 참여 제약 조건을 강제할 수 없는 이유:**

- SQL에서는 `Student` 테이블에 있는 모든 학생이 반드시 `Enrollment` 테이블에 나타나야 한다는 제약 조건을 설정할 수 없습니다.
- 예를 들어, `Student` 테이블에 새로 추가된 학생이 `Enrollment` 테이블에 등록되지 않더라도 데이터베이스는 이를 허용합니다.
- 이는 `StudentID`가 `Enrollment` 테이블의 기본 키 부분이지만 고유하지 않기 때문입니다. SQL의 외래 키 제약 조건은 기본 키나 고유 키만 참조할 수 있습니다.

### 예시: 일대다 관계

**시나리오:**
부서(Department)와 직원(Employee) 사이의 일대다 관계를 고려해보겠습니다. 한 부서에는 여러 직원이 있을 수 있지만, 한 직원은 한 부서에만 속할 수 있습니다.

**테이블 구조:**

1. **Department 테이블:**
   - `DepartmentID` (기본 키)
   - `DepartmentName`

2. **Employee 테이블:**
   - `EmployeeID` (기본 키)
   - `EmployeeName`
   - `DepartmentID` (외래 키, Department 테이블 참조)

**테이블 예시:**

```sql
CREATE TABLE Department (
    DepartmentID INT PRIMARY KEY,
    DepartmentName VARCHAR(100)
);

CREATE TABLE Employee (
    EmployeeID INT PRIMARY KEY,
    EmployeeName VARCHAR(100),
    DepartmentID INT,
    FOREIGN KEY (DepartmentID) REFERENCES Department(DepartmentID)
);
```

**설명:**

- `Employee` 테이블은 `DepartmentID`를 외래 키로 가지며, 이는 `Department` 테이블의 `DepartmentID`를 참조합니다.
- 일대다 관계에서는 특정 부서의 모든 직원을 `Employee` 테이블에서 확인할 수 있습니다.

**전체 참여 제약 조건을 강제할 수 없는 이유:**

- SQL에서는 `Department` 테이블의 모든 부서가 반드시 `Employee` 테이블에 나타나야 한다는 제약 조건을 설정할 수 없습니다.
- 예를 들어, `Department` 테이블에 새로 추가된 부서가 `Employee` 테이블에 등록되지 않더라도 데이터베이스는 이를 허용합니다.
- 이는 `DepartmentID`가 `Employee` 테이블의 기본 키 부분이 아니기 때문입니다. SQL의 외래 키 제약 조건은 기본 키나 고유 키만 참조할 수 있습니다.

이와 같은 이유로, 다대다 관계나 일대다 관계에서 전체 참여 제약 조건을 SQL 외래 키 제약 조건을 통해 강제할 수 없습니다.

---
## 6.11-b.
**총 참여 제약 조건을 복잡한 체크 제약 조건이나 어설션을 사용하여 어떻게 강제할 수 있는지 설명하시오.**

### 답변:
관계 \( R \)이 엔티티 \( A \)에서 엔티티 \( B \)로의 다대일 관계라고 가정하고, \( a \)와 \( b \)가 각각의 기본 키라고 가정합니다. 우리는 "하나" 측 관계 \( B \)에 다음과 같은 체크 제약 조건을 추가할 수 있습니다:

```sql
constraint total_part check (b in (select b from A));
set constraints total_part deferred;
```

### 설명:
1. **체크 제약 조건**:
   - `constraint total_part check (b in (select b from A));`는 \( B \) 테이블의 각 \( b \) 값이 \( A \) 테이블의 \( b \) 값 중 하나여야 한다는 것을 의미합니다. 이는 \( B \) 테이블의 모든 \( b \) 값이 \( A \) 테이블에 존재해야 한다는 것을 보장합니다.

2. **제약 조건 지연**:
   - `set constraints total_part deferred;`는 이 제약 조건이 트랜잭션의 끝에서만 확인되도록 설정합니다. 이는 트랜잭션 중간에 \( B \) 테이블에 \( b \) 값을 삽입하고 나중에 \( A \) 테이블에 해당 \( b \) 값을 삽입하는 경우를 처리하기 위함입니다. 만약 이 설정이 없다면, \( A \) 테이블에 \( b \) 값을 삽입하기 전에 \( B \) 테이블에 \( b \) 값을 삽입하면 제약 조건 위반이 발생할 수 있습니다.

### 요약:
이 체크 제약 조건과 지연 설정을 통해, \( B \) 테이블의 모든 \( b \) 값이 \( A \) 테이블에 존재해야 한다는 총 참여 제약 조건을 강제할 수 있습니다. 이는 트랜잭션이 완료될 때까지 제약 조건을 확인하지 않도록 설정하여, 트랜잭션 중간에 발생할 수 있는 제약 조건 위반을 방지합니다.

-> B에 엔터티를 먼저 삽입 후 A에 해당 엔터티를 삽입해도 constraint 제약 위반이 발생하지 않도록 deferred 처리를 해준다.


---