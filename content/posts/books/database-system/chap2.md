+++
categories = ['Database']
series = ['Database-System']
tags = ['데이터베이스']
title = "Chap2 연습문제"
date = 2024-05-09T22:50:44+09:00
draft = true
+++

#### 

2.14.a
두 가지 방법이 있지만
결과적으로는 works에서 필터링을 먼저 한 후 employee와 조인하게 된다.

```sql
-- 서브쿼리 사용
select person_name
from employee
where employee.ID in (
	-- find many
	select ID 
	from works
	where works.company_name = "BigBank";
);

-- 조인: works에서 필터링 후 그 결과를 ID로 조인
select person_name
from employee
join works on employee.ID = works.ID
where works.company_name = "BigBank";

```

2.14.d
```sql
-- 1. 
select ID, person_name
from company c
join works w on c.company_name = w.company_name
join employee e on c.city = e.city;

-- 2. 임시테이블 먼저 만들고 조인
-- 쿼리가 복잡한경우 더 나을 수 있음!
select S1.ID, S1.person_name
from (
	select *
	from company c
	join works w on c.company_name = w.company_name
) as S1
join employee e on S1.city = e.city;
```

```sql
select D1.ID
from (
	select *
	from depositor d
	join account a on d.account_number = a.account_number
) as D1
where D1.balance > 6000;

select D1.ID
from (
	select *
	from depositor d 
	join accout a on d.account_number = a.account_number
) as D1
where D1.balance > 6000 
	and D1.branch_name = 'Uptown';
```

2.18.b
```sql
select ID, name
from instructor i
where i.dept_name in (
	select dept_name
	from department d
	where d.building = 'Watson'
);

select ID, name
from instructor i
join department d on i.dept_name = d.dept_name
where d.building = 'Watson';


-- c.
select s.ID, s.name
from student s
join (
	select * 
	from takes t
	join course c on t.course_id = c.course_id
	where c.dept_name = 'Comp. Sci.' 
) as T1
on s.ID = T1.ID;


-- d.
select s.ID, name
from student s
join takes t on t.ID = s.ID
where t.year = 2018;

-- e.
-- from의 student 각 행에 대해 순회하고 있다는 점 잊지 말 것.
-- select 1은 행이 있으면 무조건 참을 반환
-- 존재여부를 파악할 대상은 'takes'임!
select ID, name
from student s
where not exists (
	select 1 
	from takes t
	where s.ID = t.ID
		and t.year = 2018
);
```

