+++
categories = ['Database']
series = ['Database-System']
tags = ['데이터베이스']
title = "TypeOrm"
date = 2024-05-09T22:50:44+09:00
draft = true
+++
## 중급 SQL

#### 기타 연습
서브쿼리를 = 으로 연결하려면 서브쿼리의 결과가 반드시 하나의 row여야 함.
두번째 쿼리의 경우 결과가 여러행이기 때문에 =을 하면 에러가 남.
이러한 불확실성을 줄이고자 IN으로 하는 것.
```sql
select * from keyword_product_counts where id in (select id from keyword_product_counts where keyword_id = 2193505);

select * from keyword_product_counts where id = (select id from keyword_product_counts where keyword_id = 2193505);
```

#### 연습문제
4.1
```sql
-- 문제 원인: instructor 테이블과 course 테이블 모두 dept_name 속성을 가지고 있습니다. 따라서 NATURAL JOIN 수행 시 교수의 소속 학과와 강좌의 개설 학과가 동일한 경우에만 결과에 포함
select *
from instructor natural join teaches natural join section natural join course
where semester = 'Spring' and year = 2008;
```

4.2
```sql
-- a.
-- all instructors, each ID and number of sections taught
-- A에는 있는데 B에는 없고, 그것을 count 0으로 치려면
-- A LEFT JOIN B
-- This query should not be written using COUNT(*) since that would count null values also.
select instructor.ID, count(sec_id) as cnt
from instructor left join teaches on instructor.ID = teaches.ID
group by instructor.ID;

-- teaches 테이블에 없는 instructor들 -> count 0이 되려면 이어야
select instructor.*, teaches.*
from instructor left join teaches on instructor.ID = teaches.ID;


-- b.
-- count(ID)는 ID 열이 NULL이 아닌 행의 수를 세고, COUNT(*)는 행의 총 수를 셉니다. 여기서는 teaches 테이블의 ID 열이 NULL이 아니므로 두 가지 방식의 결과가 같습니다.(PK라서 *와 똑같음)
SELECT ID, (
	SELECT COUNT(*) 
	FROM teaches T
	WHERE T.ID = I.ID
) AS Number_of_sections
FROM instructor I;

select instructor.ID, (
	-- 사실상 teaches에 대한 LEFT JOIN.
    select count(ID)
    from teaches
    where teaches.ID = instructor.ID
)
from instructor;

-- c. 
-- section과 instructor의 공통칼럼이 없으므로, 두 번의 JOIN을 거쳐야.
-- I ^ T : ID
-- T ^ S : course_id, sec_id, semester, year
-- coalesce(name, '-')
select course_id, sec_id, teaches.ID, if(name IS NULL, '-', name) AS name
from (section natural left join teaches)
left join instructor on teaches.ID = instructor.ID
-- 이하 natural left join instructor과 동일
where semester = 'Spring' and year = 2008;

-- d. 없는 필드는 count 0으로 표시하려면 없는 칼럼 기준으로 count해야함.
-- 여기서 ID = NULL 인 행은 무시되어 0으로 카운트됨.
select D.dept_name, count(ID) as Instructor_count 
from department D
left join instructor I on D.dept_name = I.dept_name
group by D.dept_name;

-- 우선 이 쿼리를 해보면, instructor가 없는 D는 inst가 NULL로 채워져서 온다.
-- 그런데 count(*)이나 count(dept_name)은 LEFT JOIN 후에도 dept_name은 항상 NULL이 아니므로 count 1이 됨.
select * 
from department D
left join instructor I on D.dept_name = I.dept_name;
```

4.3 
outer join 없이 외부 조인을 평가할 수 있다.
이를 위해 outer join을 사용하지 않고 다음 sql 질의를 재작성해보자.
```sql
select * from student natural join takes
-- 그냥 행 결과를 합친다 (칼럼만 같으면 됨)
UNION
select ID, name, dept_name, tot_cred, NULL, NULL, NULL, NULL, NULL
from student
where not exists (
	select ID from takes where takes.ID = student.ID
);

/* ID, name, dept_name, tot_cred, course_id, sec_id, semester, year, grade */
select ID, name, dept_name, tot_cred, NULL, NULL, NULL, NULL, NULL
from student S
-- takes.ID가 존재하지 않는 student를 찾아라
where not exists (
	select ID from takes T where T.ID = S.ID
);
```

4.5

```sql


```

4.6
가중평균
```sql
-- credit_sum이 0이면 0으로 나누지 않도록 NULL로 처리
select ID, (credit_points_sum / IF(credit_sum = 0, null, credit_sum)) as GPA from (
  select 
    ID, 
    -- grade = NULL이면 points도 NULL인데 0으로 처리
    sum(credits * IFNULL(points, 0)) as credit_points_sum,
    sum(credits) as credit_sum
  from takes t
  -- grade = NULL이어도 join 결과에 있어야 함
  left join grade_points gp on t.grade = gp.grade
  join course c on t.course_id = c.course_id
  group by ID
  -- takes 수강한적 없는 학생은 credit * point, credit SUM 모두 NULL로 둔다
  union
  select ID, null, null
  from student
  where ID not in (
    select ID from takes
  )) as tmp;


-- 참고
IF(condition, true_result, false_result)
IFNULL(Column명, "Null일 경우 대체 값")

SELECT student_id,
       IF(grade IS NULL, 0, grade) AS adjusted_grade
FROM student_grades;

```

4.8
제약조건을 약하게 걸어보면 두 명의 교수 케이스가 나오긴 하는데
(동일한 학기에 동일한 강의실에서 두 개 이상의 수업이 열리는) -> year 조건을 안걸었음.
이미 duplicate key가 잘 걸려있어서 (courseId, SecId, Spring, 2008) 답안과 같은 케이스는 안나옴.

```sql
-- a.
select i.ID, name, s.semester, building, count(distinct concat(building, room_number)) as same_place_cnt
from instructor i
left join teaches t on i.ID = t.ID
join section s on 
	t.course_id = s.course_id and
	t.sec_id = s.sec_id and
	t.semester = s.semester and
	t.year = s.year
group by i.ID, name, semester, building
having same_place_cnt > 1;

-- 엉터리 케이스를 만들었음..
-- 아래는 빌딩,룸넘버 같은거 두개 해도 sec_id가 달라서 안걸릴수밖에
select * 
from instructor i
left join teaches t on i.ID = t.ID
join section s on t.course_id = s.course_id
where i.name = 'Heo';
```

4.9

```sql

```

```sql

```

```sql

```
---
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