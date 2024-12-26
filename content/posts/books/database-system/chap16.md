+++
categories = ['Database']
series = ['Database-System']
tags = ['데이터베이스']
date = 2024-11-10T22:50:44+09:00
draft = true
+++

# Chap 16. Query Optimization

## EXPLAIN

Key: index

Type: 
- const: 유일한 id
- ref:  equality operator(주로 non unique). 어쨌든 인덱스 일부 칼럼을 사용
- index: 인덱스 풀스캔(인덱스의 모든 엔트리, 더 느림). 정렬, group by  할 때도 나타날수.
- eq_ref: 첫번째 읽은 테이블 칼럼값을 두번째 테이블의 PK/UK 검색조건에 사용할 때. 

Ref: 인덱스와 비교되는 값
- const: 인덱스 검색시 사용되는 값이 상수임을 의미

---
### Statistics for operator optimization
연산비용이라는건 통계가 필요함.
기본적으로 input 릴레이션의 튜플수, 튜플 사이즈 등.
그러나 input이 sub expression 결과인 경우
그 결과에 대한 통계가 필요해짐. - 어떤 속성의 distinct value 수라던가.

어떤 evaluation plan을 택할 것인가?
당장은 비용이 더 많이 들 순 있지만 그 이후 연산을 거치면 더 효율적인 경우들 있음.



---
### Pipelining
Nested-loop join의 파이프라이닝 기회는 다음과 같은 상황을 의미합니다:

### 예시 상황:
```sql
SELECT *
FROM Orders o, OrderDetails d, Products p
WHERE o.order_id = d.order_id 
AND d.product_id = p.product_id
```

### 파이프라이닝 없는 실행:
1. Orders와 OrderDetails 조인 완료
2. 결과를 저장
3. 저장된 결과와 Products 조인

### 파이프라이닝 있는 실행:
1. Orders에서 한 튜플 읽음
2. 매칭되는 OrderDetails 튜플 찾음
3. 즉시 매칭되는 Products 튜플 찾음
4. 결과 즉시 출력
5. 다음 Orders 튜플로 진행

### 장점:
- 중간 결과를 저장할 필요가 없음
- 메모리 사용량 감소
- 첫 번째 결과를 빨리 얻을 수 있음

이처럼 nested-loop join은 각 단계의 결과를 기다리지 않고 순차적으로 처리할 수 있는 파이프라이닝 기회를 제공합니다.

---



---
### 실전문제
16.1

```sql
-- e. semijoin
-- 학생 테이블에서 각 학생에 대해 해당 학생이 과목ID '105'를 수강했는지 확인
explain
select distinct s.ID, s.name
from student s
where exists (
	select 1
	from takes t
	where t.ID = s.ID
	and t.course_id = '105'
);
```

이 실행 계획에서 쿼리의 실행 순서는 다음과 같습니다:
순서는 id의 역순. 가장 안쪽의 서브쿼리부터 실행되어 바깥쪽으로 나아감.
🍎-> 어쨌든 index(Key)가 의도한대로 걸렸는지만 잘 보면 됨.

1. **MATERIALIZED (t 테이블)**:
   - `takes` 테이블에서 `course_id = '105'`인 행들을 먼저 필터링합니다.
   - 이 결과는 물리적으로 저장됩니다(`MATERIALIZED`).

2. **SIMPLE (s 테이블)**:
   - `student` 테이블의 각 행에 대해 `EXISTS` 조건을 평가합니다.
   - `takes` 테이블의 저장된 결과와 비교하여 해당 학생이 과목 '105'를 수강했는지 확인합니다.

3. **SIMPLE (<subquery2>)**:
   - `EXISTS` 절의 서브쿼리를 실행하여 `student` 테이블의 각 행에 대해 조건을 평가합니다.

따라서, `takes` 테이블의 서브쿼리가 먼저 실행되어 결과가 저장되고, 그 후에 `student` 테이블의 각 행에 대해 이 저장된 결과를 사용하여 조건을 평가합니다.

```sql
-- f. not in clause
explain
select *
from student s
where s.ID not in (
	select t.ID
	from takes t
	group by t.ID
	having count(t.course_id) > (
		select avg(course_count)
		from (
			select ID, count(course_id) as course_count
			from takes
			group by ID
		) as student_course_counts
	)
);

```
g. 상관 서브쿼리(correlated subquery)는 메인 쿼리의 각 행에 대해 서브쿼리가 실행되는 것. 서브쿼리가 메인 쿼리의 특정 열 값을 참조할 때 발생.

```sql
SELECT i.*
FROM instructor i
WHERE i.salary > (
    SELECT AVG(i2.salary)
    FROM instructor i2
    WHERE i2.dept_name = i.dept_name
);
```
- 상관 서브쿼리: 각 교수의 학과별 평균 급여를 계산합니다.
- 비교: 교수의 급여가 "해당 학과의" 평균 급여보다 높은 경우만 선택합니다.


i.
1. 서브쿼리에서 각 학과의 평균 급여의 10%를 계산합니다.
2. 이 결과를 메인 instructor 테이블과 조인합니다.
3. 조인된 결과를 사용하여 각 교수의 급여를 업데이트합니다.
```sql
-- 각 교수의 급여를 그 교수의 학과 평균 급여의 1%만큼 인상
EXPLAIN
UPDATE instructor i
JOIN (
    SELECT dept_name, AVG(salary) * 0.1 as salary_increase
    FROM instructor
    GROUP BY dept_name
) AS dept_avg ON i.dept_name = dept_avg.dept_name
SET i.salary = i.salary + dept_avg.salary_increase
WHERE i.dept_name IN (
    SELECT dept_name
    FROM department
);

select *
from instructor i
join (
    SELECT dept_name, AVG(salary) * 0.1 as salary_increase
    FROM instructor
    GROUP BY dept_name
) AS dept_avg ON i.dept_name = dept_avg.dept_name;
```