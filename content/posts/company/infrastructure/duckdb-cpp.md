+++
categories = ['Company']
series = ['DuckDB']
tags = ['DataWarehouse']
title = "DuckDB"
date = 2024-07-25T22:50:44+09:00
draft = true
+++
## 🔵 
### 현재 상황 파악

3초 중 1초는 duckdb 결과를 파이썬으로 fetch하는데 쓰임.
duckdb cpp에서 파이썬으로 바인딩하는 과정에 오버헤드가 있는듯.

언어단의 한계라면
cpp 또는 Go로 bullmq + duckdb 쿼리로 교체해보자.

bullmq는 Node/파이썬 전용이라
bullmq proxy 쓰면 될듯.

https://docs.bullmq.net/http-api/queues/adding-jobs

<img width="797" alt="image" src="https://github.com/user-attachments/assets/95decca4-dd50-4e2f-8af6-73a8af122ba4">


### 구조

<img width="924" alt="image" src="https://github.com/user-attachments/assets/385adc8d-dcd7-4e17-accc-989729a3d279">

쿼리서버 producer와 별개로
job을 만들어서 분배할 프록시 서버가 있어야하는데,
그걸 쿼리서버와 같은 테섭/실섭에서 하면 될듯.

참고로 
기존 bullmq 라이브러리는 worker(컨슈머)가 레디스 캐시를 바라보고
job이 들어오면 pull 가져가는 방식이었는데,

**bullmq proxy는 job이 생성되면 worker(여기선 query01~04) 엔드포인트로 POST 요청(push)하는 형태임.**

어쨌든 bullmq proxy 코드는 queryServer에 Node로 작성하면 되고(POST 요청하는 부분)
Worker만 DuckDB cpp API 참조해서 작성하면 되지않을까?
테섭에서 충분히 테스트 거치고.
