+++
title = 'DSA Patterns'
date = 2024-06-30T00:20:22+09:00
draft = true
+++

DSA 패턴- 
🛍️ Two Pointer
배열에서 특정 조건을 만족하는 요소 쌍을 찾으려면 O(n^2)
투포인터는 한번의 반복(O(n))으로 요소 쌍을 찾을 수 있음.
- 두 포인터가 가리키는 요소의 합이 목표 값보다 작으면 시작 포인터를 오른쪽으로 이동
- 두 포인터가 가리키는 요소의 합이 목표 값보다 크면 끝 포인터를 왼쪽으로 이동

while (left < right)


---
* Sliding window
nums - n elements. int k
length = k인 subarray - that has max avg value and return this value.

특정 조건을 만족하는 배열 서브셋을 구하는 것.

size.
초기화
slide
update
slide
return 

max value
i - window index
sum (evaluate)

---

< 재귀 >
더이상 나눌 수 없는 배열의 길이 1.
트리의 높이
합치면서 올라가는 과정 - 바닥부터 정렬하면서 합친다

---