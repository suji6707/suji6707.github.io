+++
title = 'Review Analysis'
date = 2024-12-09T00:20:22+09:00
draft = true
+++
## Review Analysis 구조


💎💎💎 TODO
깃 파기. 
ecr image에서 함수로 바꾸고 로컬에서 호출하자(별도 브랜치 ㄱ?). 
그전에 우선 깃에 push해두고. 


💎 DuckDB Result Conversion 형태 세 가지.
python object: fetchall
pandas - numpy 래퍼: df
numpy: fetchnumpy

💎 S3 접근 두 가지 방법
1. boto3 클라이언트로 접근
2. duckDB에서 바로 접근
- 지금은 후자이며, create secret으로 .

🍋 질문
- 어떻게 클라에서 query를 받아올 것인가?
    - 일단 사이트 없이
    - 로컬에서 8000 포트로 요청하면 
    - 람다환경에선 api gateway
- 어떻게 requestId를 기록하고 클라에서 폴링할 것인가
    - 다이나모DB 객체
    - 3초?마다 requestId로 검색
- */*/* 에서 타임시리즈 읽기
    - order by year,month,day 순으로 읽기
    - read parquet 파일을 split해야하는가?


🍎 람다1
1. 클라에서 custom 쿼리를 받음
2. 파싱
3. requestId 생성
4. SQS로 job 보냄
5. requsetId를 DynamoDB에 저장 및 반환
++ 메타데이터만 dynamo에 저장하고
result는 람다2에서 S3에.

* 필요한 필드
- requestId
- query
- result는 requestId 이름으로 💎S3 object로 저장해도 될듯?
    - json을 바로 저장하기보단 csv, pq 등
- status 
- timestamp (createdAt, updatedAt ?): 쿼리실행시간 있으면 좋을듯..

🍎 람다2
SQS 이벤트로 호출되어
1. requestId 에 대한 query 읽기
2. 쿼리 실행
3. 결과 S3에 저장

- 일단 requestId 생성후 query param이랑 dynamo에 저장하고 (실제 1에서 수행)
- query 실행 후 S3 에 저장
- requestId로 dynamo 키 찾아서 status 업데이트 (실제 3에서 수행)
- S3에서 가져와서 반환 (실제 3에서 수행)
-> 나중에 S3 저장 이후 바로 반환하도록 변경


https://www.serverless.com/framework/docs/providers/aws/events/sqs

🍎 람다3
1. client가 폴링해서 SQS 이벤트 완료를 인지하면 
2. 람다3로 requestId를 보낸다
3. 람다3는 dynamo에서 status를 완료로 바꾼 후
4. (S3에서) 결과를 찾아서 반환한다

🔺 람다3의 유스케이스는 두 가지다.
requestId만 있으면 
- dynamo의 status COMPLETED를 확인하고 s3에서 찾아오나 (new 쿼리 결과)
- dynamo 확인없이 바로 s3에서 찾아오나 (history 쿼리결과)
결과를 반환한다는 점은 같음.

🔺 데이터 반환 형태
- csv or dictionary
- 클라이언트는 squelAce처럼 테이블로 볼 수 있어야함.
- curl로 요청해서 데이터를 받았을때 클라이언트 서버에 sqlite 등으로 임시로 결과를 저장할 수도 있음. file로 받는 경우(-o)와 dict로 받는경우 각각 어떤식인지 궁금

- curl로 query params 보내는법?
curl -G -H "Authorization: Bearer access_token_goes_here" \
  https://api.provider.com/thing/you_want/index.json \
  -d "query=scope=1"
- G를 붙여야 d가 POST가 아닌 query param이라는 게 명시됨
- Note that if you use -d without -G, the request will be a POST request.


curl -o data.csv https://api.example.com/data (CSV)
curl https://api.example.com/data -H "Accept: application/json" (JSON)


테스트 환경
써볼 함수들
readString: header: False/True. => 리스트/객체


💎 리액트- 클라 구현
- 쿼리창은 VS코드처럼 tab이 되어야하고 syntax에러도 잡아줄 수 있어야할듯?
    - 아니면 에러시 무조건 null 이던가
- history는 클릭하면 기존 데이터 바로 가져오되(status 바로 떠야함), run 버튼 누르면 새로 실행하면서 history에 추가되게끔.



🔺 AWS API Gateway CORS
- 결국 서버에서 응답헤더에 Allow Access를 넣어줘야하는데(origin, credential)
OPTIONS를 허용시 
- OPTIONS 메서드의 200 응답은 preflight 핸드셰이크를 수행하기 위해 Access-Control-Allow-* 헤더 세 개를 반환하도록 자동으로 구성
- GET) 메서드는 기본적으로 200 응답에 Access-Control-Allow-Origin 헤더를 반환

🔺 serverless yml 옵션만으로는 cors가 작동안하는듯.
수기로 cors 활성화 후 API 배포 눌러줘야함.
응답에도 allow~ 넣어야함

