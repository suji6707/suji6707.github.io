+++
title = 'Review Analysis'
date = 2024-12-09T00:20:22+09:00
draft = true
+++
## Review Analysis


keyword_types 테이블이랑 조인
- 마찬가지로 파이썬으로 duckdb 쿼리, keyword_id IN에 1만개 넣어서. 
(keyword_ids_only_valid_2024_search.csv 첫번째 row string 전체 1.8만개임)
- pc_shopping_tab, shopping_search_order 💎3 이하💎, (0은 아니어야)이면 충분히 '쇼핑성'으로 구분할만.
얘네들만 keyword랑 나머지 항목들 뽑아내기. (쇼핑탭오더, ms, prdcnt, 매년 growth)

### 분석툴 TODO
🔮🔮🔮🔮🔮
분석이 아니라 분석이 가능한 구조만 만들어본다.

오늘 목표
keyword 테이블 JOIN 문 실행해보기(sls deploy s3 anal)
🍎 create view - dynamo.
-> 사이드바 뷰테이블을 클릭하면 기본적으로 쿼리결과가 나온다.
즉 view는 실행시 requestId를 갖게되고 해당 type=VIEW&vieName=name인 dynamo에 requestId에 넣어주게 됨.
- 새로 RUN을 하면 신규 reqId가 생기고 해당 뷰의 reqId를 대체. ㅊ
🍎 11번가 with절 쿼리 실행해보기
🍎 download csv
🍎 Fail시 state 업데이트와 동시에 Error 칼럼에 에러메시지 json.dumps로 넣기
---
바로 해볼 것.
1. keywords JOIN.


클라이언트로 로직 이동
1. 타임라인을 프론트에서 설정하도록. 커서 부분에 삽입?
2. 왼쪽 테이블 클릭시 데이터소스 복사할 수 있음.

--- 
목표
최근 11번가 리퀘스트를 뷰에서 바로 처리할 수 있게함.
kid, keyword, (ms, sales) by month

--


1. 클라: create view 버튼을 누르면 즉시
2. producer에서 해당 쿼리에 create view as [뷰이름]을 붙이고
dynamo에 requestId = 뷰이름, type= VIEW, dataSource= MySQL / S3
query:에는 temp.main. 붙여서
응답은 뷰 이름.
sqs에는 반드시 RUN 버튼인 경우에만 들어가는데
3. consumer: 
쿼리명에 temp.main이 있는경우 해당 뷰이름을 dynamo - requestId, type = VIEW로 찾고
해당 쿼리를 가져온다.


---

🔺🔺🔺
create view
1. 클라이언트: 쿼리와 join 쿼리를 동시에 보낸다.
    create view 버튼을 누르면 로컬 메모리/ref에 view[].push 되고
    ( code: MySQL 또는 S3 )
    side bar의 views.push로 나타나게 한다.
    사용자는 그 view 이름을 가지로 최종 쿼리를 작성해 run 한다.
=> view를 저장되지 않는, 한번의 세션에서만 사용할 수 있는 존재로 우선.
Run을 하기전까지 view는 사이드바에 쌓이고,
Run을 하면 없어진다.
    
2. producer: 
- view 쿼리와 query를 ; ; ; 로 이어서 하나의 스트링으로 만들어 다이나모 쿼리에 저장.
- *** 뷰 이름엔 temp.main을 붙여야함.
view: [, , , ],
query: " "
로 받는다. 
view 가 있는 경우 Dynamo - view 필드에 true를 넣는다.

3. consumer:
dynamo.view 필드가 true인 경우
람다는 attach mysql을 하고 쿼리스트링을 그대로 실행하면 된다. 
결과를 S3에 저장한다.

---
무엇을 해야하나? 
이제부터가 진짜다.

지금은 테이블 다듬을 때가 아님.

Data Source 
- MySQL: item_scout 추가

Tables:
- show all tables -> temp가 뜸 (아이콘표시)

join 쿼리시:
- temp.main.A 형식으로 조인하면 됨.

🔺 문제는 
temp가 하나의 커넥션에서만 유지된다는 것.

사용자 세션을 유지

메모리 캐시 유지, 뷰 ID도 반환
- 고유 아이디로?

view가 맞음.

FROM duckdb_views
SELECT * EXCLUDE(sql);

create view를 할 때는 mysql 연결 어케하지?


CREATE PERSISTENT SECRET mysql_itemscout (
    TYPE MYSQL,
    HOST '127.0.0.1',
    PORT 3306,
    DATABASE mysql,
    USER 'root',
    PASSWORD 'root'
);


🍎 반드시 실제 db는 readonly!!
ATTACH 'host=localhost user=root password=root port=3306 database=item_scout' AS mysqldb (TYPE MYSQL, READ_ONLY);

USE mysqldb;
duckdb가 접근할 수 있게하려면 attach 커맨드

ATTACH '' AS mysql_itemscout (TYPE MYSQL);

🍎 조인시 temp table vs view
1. temp table
한 커넥션 세션 동안에만 메모리에 유지.
그러나 temp_directory 설정시 디스크로 감.

2. view
뷰는 실제 데이터를 저장하지 않고, 쿼리의 정의만을 저장하므로
매번 쿼리를 실행됨. -> 메모리 사용량은 적으나 ..
장점은 데이터 변경되어도 자동으로 최신데이터 반영한다는점. 

---

## serverless.yml

```yaml
resources:
  Resources:
    myCustRole:
      Type: AWS::IAM::Role
      Properties:
        RoleName: myCustRole
        AssumeRolePolicyDocument:
          Version: '2012-10-17'
          Statement:
            - Effect: Allow
              Principal:
                Service: lambda.amazonaws.com
              Action: sts:AssumeRole
        Policies:
          - PolicyName: myCustPolicy
            PolicyDocument:
              Version: '2012-10-17'
              Statement:
                - Effect: Allow
                  Action:
                    - ecr:DescribeImages
                    - ecr:DescribeRepositories
                    - ecr:CreateRepository
                    - ecr:GetAuthorizationToken
                    - ecr:BatchCheckLayerAvailability
                    - ecr:BatchGetImage
                    - ecr:CompleteLayerUpload
                    - ecr:GetDownloadUrlForLayer
                    - ecr:InitiateLayerUpload
                    - ecr:PutImage
                    - ecr:UploadLayerPart
                    - ecr:DeleteRepository
                  Resource: 
                    - arn:aws:ecr:${AWS::Region}:${AWS::AccountId}:repository/serverless-lambda-duckdb-dev

```