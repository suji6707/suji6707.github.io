+++
title = 'Query Explain'
date = 2024-07-22T00:20:22+09:00
draft = true
+++

# Explain
데이터에 도달하는 데 사용하는 액세스 유형, 사용된 인덱스, 액세스되는 예상 행 수

/* 
Key: index

Type: 
- const: 유일한 id
- ref:  equality operator(주로 non unique). 어쨌든 인덱스 일부 칼럼을 사용
- index: 인덱스 풀스캔(인덱스의 모든 엔트리, 더 느림). 정렬, group by  할 때도 나타날수.

Ref: 인덱스와 비교되는 값
- const: 인덱스 검색시 사용되는 값이 상수임을 의미



2.explain 4분~
*/


* bio는 fulltext로.

---