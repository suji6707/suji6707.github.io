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
-- 명시적으로 조건 지정. instructor.dept_name = course.dept_name을 지정하지 않는 한편
select i.*, c.*
from instructor i
join teaches t on i.ID = t.ID
join section s on t.course_id = s.course_id and t.sec_id = s.sec_id and t.semester = s.semester and t.year = s.year
join course c on s.course_id = c.course_id
where t.semester = 'Spring' and t.year = 2008;

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
-- count(ID)는 ID 열이 NULL이 아닌 행의 수를 세고, COUNT(*)는 행의 총 수를 셉니다. 여기서는 teaches 테이블의 ID 열이 NULL이 아니므로 두 가지 방식의 결과가 같습니다.
SELECT ID, (
	SELECT COUNT(*) 
	FROM teaches T
	WHERE T.id = I.id
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


```sql

```

```sql

```