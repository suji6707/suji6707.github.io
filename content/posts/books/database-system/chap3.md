+++
categories = ['Database']
series = ['Database-System']
tags = ['데이터베이스']
title = "Chap3 연습문제"
date = 2024-05-09T22:50:44+09:00
draft = true
+++
## Intro

#### 

3.1.b
```sql
-- 튜플로 여러조건 만족
select distinct(ID)
from takes
where (takes.course_id, takes.sec_id, takes.semester, takes.year ) IN (
	select t.course_id, t.sec_id, t.semester, t.year
	from teaches as t
	where t.ID = (
		select i.ID
		from instructor as i
		where i.name = 'Morris'
	)
);

-- join은 나중에 키를 사용해서 최적화할 수도 있음.
SELECT DISTINCT t.ID
FROM takes t
JOIN teaches ts ON t.course_id = ts.course_id
                AND t.sec_id = ts.sec_id
                AND t.semester = ts.semester
                AND t.year = ts.year
JOIN instructor i ON ts.id = i.id
WHERE i.name = 'Morris';

-- 카티션곱
select distinct takes.ID
from takes, instructor, teaches
where 
	takes.course_id = teaches.course_id and
	takes.sec_id = teaches.sec_id and
	takes.semester = teaches.semester and
	takes.year = teaches.year and
	teaches.id = instructor.id and
instructor.name = 'Morris';
```

3.1.e
```sql
-- 1단계: section left join
select *
from section 
left join takes on takes.course_id = section.course_id
	and takes.sec_id = section.sec_id
	and takes.semester = section.semester
	and takes.year = section.year
where section.year = 2008 and section.semester = 'Fall';

-- 2단계: count
select section.course_id, section.sec_id, section.year, section.semester, count(takes.ID) as enrollment 
from section 
left join takes on takes.course_id = section.course_id
	and takes.sec_id = section.sec_id
	and takes.semester = section.semester
	and takes.year = section.year
where section.year = 2008 and section.semester = 'Fall'
group by section.course_id, section.sec_id, section.year, section.semester;

-- where 서브쿼리
select course_id, sec_id, count(ID)
from takes 
where (takes.course_id, takes.sec_id, takes.semester, takes.year) IN (
	select s.course_id, s.sec_id, s.semester, s.year
	from section as s
	where s.year = 2008 and s.semester = 'Fall'
)
group by course_id, sec_id;
```
3.1.f
```sql
WITH cte_name AS (
    -- Subquery
    SELECT ...
    FROM ...
    WHERE ...
)
SELECT ...
FROM cte_name

with counted_section as (
	select s.course_id, s.sec_id, s.semester, count(t.ID) as count 
	from section s
	join takes t on t.course_id = s.course_id
		and t.sec_id = s.sec_id
		and t.year = s.year
		and t.semester = s.semester
	where s.year= 2008
	group by s.course_id, s.sec_id, s.semester
	)
select *
from counted_section cs
where count = (
	select max(count)
	from counted_section
);
```

---
3.2.a
```sql
select tmp.ID, sum(credits)
from (
	select * 
	from course natural join takes t
	where t.ID = '10663'
) as tmp;

-- join 두 번보다는 카티션&where 두 번이 더 빠름
select sum(credits * points)
	from takes t
	join grade_points gp on t.grade = gp.grade
	join course c on t.course_id = c.course_id
where ID = '10663';

select sum(credits * points)
from takes, grade_points, course
where takes.grade = grade_points.grade
 	and takes.course_id = course.course_id
	and ID = '10663'; 
```


Exists 예시
1. 특정 과목을 수강한 학생들 목록
```sql
-- EXISTS 절은 student 테이블의 각 행에 대해 takes 테이블의 모든 행을 순회하여 조건을 만족하는지 확인
select *
from student s
where exists (
	select 1 from takes t
	where t.ID = s.ID
	and t.course_id = '239'
);

-- 아직 과목을 수강하지 않은 학생 목록
select *
from student s
where not exists (
	select 1
	from takes t
	where s.ID = t.ID // 🟡 위의 s와 연결되어있어야 가능
);

-- find ID and g_point avg of each student
select ID, sum(c.credits * gp.points) / sum(credits) as GPA
from takes t, grade_points gp, course c
where t.grade = gp.grade
	and t.course_id = c.course_id
group by t.ID
union
select ID, null as GPA
from student s
where not exists (
	select 1 from takes t
	where t.ID = s.ID
);	
```
3.3
```sql
-- b.
-- 아래 두 쿼리는 같은 결과이나 두번째가 나음
select *
from course c
where not exists (
	select 1
	from section s
	where c.course_id = s.course_id
);

select * from course
where course_id not in (
	select course_id from section
);

-- 삭제 버전
delete from course
where course_id not in (
	select course_id from section
);

-- c.
insert into instructor (name, dept_name, salary)
select name, dept_name, 10000
from student
where tot_cred > 129;
```

3.4
```sql
-- a번. 사실 차량정보는 필요없음. participated에서 바로 owns와 연관시키면 됨
select count(distinct(person.driver_id))
from accident, participated, person, owns
-- 사고가 난 차량정보
where accident.report_number = participated.report_number
-- 사고 차량의 실제 소유자
and owns.license_plate = participated.license_plate
-- 그 소유자의 person 정보
and owns.driver_id = person.driver_id
and year = '2017';

-- 3.5
select grade, count(ID)
from (
	select ID,
		case 
			when score < 40 then 'F'
			when score < 60 then 'C'
			when score < 80 then 'B'
			else 'A'
		end as grade
	from marks
) as tmp
group by grade;

-- 3.8
-- a.
select d.ID
from account a
join depositor d on a.account_number = d.account_number
-- 일단 depositor, borrower가 있는지는 신경쓰지 않고 - 그다음 left join
🟡left join borrower b on d.ID = b.ID
🟡where b.ID is null; 

-- b.
select ID
from customer c
where (c.customer_street, c.customer_city) = (
	select customer_street, customer_city
	from customer
	where ID = '12345'
);

-- 🟡 조인이 서브쿼리보다 빠를 수 있음. 자기자신을 join. 타겟은 F
select F.ID
from customer F, customer S
where F.customer_street = S.customer_street
	and F.customer_city = S.customer_city
	and S.ID = '12345';


-- c. 🟡 <>는 대소비교가 아니라 '같지 않음'임.
select ID
from works
where company_name <> 'First Bank Corp';

-- f.
with count_by_company as (
	select company_name, count(ID) as emp_cnt
	from works
	group by company_name
)
select company_name
from count_by_company
where emp_cnt = (
	select max(emp_cnt)
	from count_by_company
);

-- g.
-- where는 select 전에, hanving은 select 후에(group by 등 agg 적용후 - 다 가져와서 row가 결정된 후에 )
select company_name, avg(salary) as avg_salary
from works
group by company_name
having avg_salary > (
	select avg(salray)
	from works
	where works.company_name = 'First Bank'
);

-- 3.10.
-- b.
update works
set salary = (
	case
		when salary * 1.1 < 100000 then salary * 1.1
		else salary * 1.03
	end
)
where works.ID in (
	select ID 
	from manages
) and works.company_name = 'First Bank';

```


