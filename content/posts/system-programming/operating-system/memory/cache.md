+++
title = 'Memory Cache'
date = 2025-01-05T00:20:22+09:00
draft = true
+++

#### 🚀🚀🚀https://youtu.be/Bz49xnKBH_0?si=Dd7WDWkPzQAvZCYT

# Memory Cache

메모리 bit는 tag, data로 나뉨.

메모리자체를 tag로 활용. 
block id ex) 1, 5 -> 2bit: 0, 1나눈 나머지로 생각. 

공간 찾아가는건 line ID.
캐시 히트/미스 판단은 tag.
tag의 last 3bit를 line ID로. 


RAM is actually an addressing mode where i give a pattern of 0 and 1, 
I have access to specific memory location within that full memory map .
캐시는 이와 달리 associative addressing을 사용한다. 
where it came from - tag

< associate a tag with a block in a cache 방법 > 
1. Fully associative mapping algo
캐시는 라인들로 나뉘고,
각 라인은 tag, block data로 나뉨.

라인 중에 일치하는 tag가 있으면 메인메모리까지 갈 필요 없다.

장점: threasing이 적다
Fully Associative 캐시는 모든 데이터가 캐시의 어떤 라인에든 저장 ->  특정 캐시 라인에서 데이터 간 충돌이 발생하지 않음 

2. Direct Mapping

11111
32: 100000
매 2^5 = 32 개 address 마다 loop
number of blocks that are sharing a line 
12 bit address space(4KB) 에서 
tag : 7bit -> 2^7=128개의 블록이 한 라인ID을 공유하게 됨.
=> cheap and easy search but lot of threasing.

3. Set-associative caches
캐시 라인들을 여러 개의 "set"으로 묶는다.
2 way: 각 set에 2개의 라인이 있음.
같은 tag에 두 개의 index


## 캐시 write 정책, flag bits, split caches

flags
- type: instruction은 decode 필요, data는 그렇지않음.
- valid: invalid로 바꾸기 전 main memory에 반영해야. 
- lock: 대체못하게 표시
- dirty***: written but not updated in main memory

Write policy
- write through: every write to cache also writes through main memory
- write back: update main memory only when a dirty line is replaced. (수정된 데이터가 캐시에서 제거될 때 메인메모리에 반영)
 단점) write back은 여러 프로세스가 한 데이터를 접근할 경우
최신 데이터를 읽기 위해 캐시를 거쳐야 할 수도?

* indstuction / data
L1 Cache: 보통 split cache, write-through: 워낙 빨라서 바로바로.
L2: Unified, write-back. 메모리 트래픽 최소화





---
캐시 메모리에서 Line의 역할
캐시는 제한된 크기의 메모리를 다루므로
주 메모리 주소를 캐시 라인 개수로 나눈 값에 따라 데이터를 배치해야.

Line ID는 캐시 내부에서 라인의 물리적 위치
-> 메모리 주소의 하위 비트가 캐시에서 저장 위치를 결정하는 셈

SetID는 검색
1) Set Index를 사용하여 Set을 찾습니다 (먼저)
2) 선택된 Set 내에서 Line ID를 암묵적으로 사용
이 Set 안에 있는 k개의 라인 중에서 어떤 라인에 데이터를 저장할 지.
-> lineID는  하드웨어가 Set 내의 라인들을 관리하기 위해 내부적으로 사용하는 개념

책꽂이: Set
책꽂이 칸: Set index
각 칸 안의 책 위치: lineID
책 제목: Tag 를 보고 찾음. 



