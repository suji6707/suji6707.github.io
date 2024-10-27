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

## 중첩 루프 조인

