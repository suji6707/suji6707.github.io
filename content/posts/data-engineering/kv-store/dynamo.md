+++
title = 'DynamoDB'
date = 2024-07-22T00:20:22+09:00
draft = true
+++


### 🟡 Dynamo Spec

꼭 핫파티션이 문제는 아님.
한 파티션당 1000WCU까지 max인데
그냥 total 1000WCU가 문제인것. (이걸 계속 늘렸음. 근데 어쨌거나 한 파티션에 1000 넘는건 시간문제라)
파티션 나눈다해도 나눠가지는 느낌?


# Read Capacity Unit
1 RCU는 최대 4KB의 데이터를 읽을 수. 
-> Provisioned capacity units이 77이라고 해서 초당 77개의 아이템을 읽을 수 있다는 게 아님. 
우리는 한 아이템에 50KB이고, 따라서 한 아이템당 12.5 RCU를 쓰게 됨.
-> 초당 약 6개만 읽을 수 있음.
쓰기는 90까지.

WCU (쓰기용량)을 더 크게 해둔 이유는
쓰기가 읽기보다 적지 않아서.
한번 write당 읽기는 한두번 뿐.

** provisioned지만 auto scaling을 켜놓으면
현재 최소용량에서 max 설정까지 늘어남.
ondemand랑 비슷해보이는데 더 저렴할 수 있다고.
-> ㅋㅋㅋ auto scaling 안키고 RCU, WCU만 고정했으면 큰일날뻔했음.. 바로 60만원 나갈뻔
단 auto scaling 기록에서 캐파가 줄어들지 않고 고정되는지는 지켜볼 필요.


💎 Hash
sha256은 항상 256비트 = 32바이트 해시값을 생성
- 각 바이트는 8비트 = 0~255 (11111111까지), 
16진수로는 00~FF 라서 두자리 필요.
- 32바이트 x 2자리(16진수) = 64자

---

### 키 밸류 설계
해시테이블은 오브젝트.
스토리지 사이드의 오브젝트인셈.

해시값이 키가 됨.
해시로 키를 찾은다음.
hash key, sort key가 있어서 
키 값 안에서 또 찾을 수.
user-timestamp라 한다면.

---
설계
hash key: reqId,
sort key: timestamp

- requestId
- query
- status 
- timestamp (createdAt, updatedAt ?): 쿼리실행시간 있으면 좋을듯..


* 실질적인 인덱스는 sortkey
sortkey로 .
범위검색 할수있도록 구성해보기?
- 최근 검색기록 requestId로 다시 한다던가 유스케이스가 있을듯.
💎 모든 기능을 다 쓸 필요는 없음.! 주어진 상황과 필요에 맞게.


