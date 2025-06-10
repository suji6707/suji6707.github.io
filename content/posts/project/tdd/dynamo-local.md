+++
title = 'dynamo local'
date = 2025-06-10T00:20:22+09:00
draft = true
+++

### 왜 dynamo 로컬 개발시 NoSQL Workbanch를 써야할까?
운영 하나로 로컬에서 테스트하게되면 PK가 충돌함. 특히 db pk를 dynamo pk를 쓰면 무조건.

```yaml
services:
 dynamodb-local:
   command: "-jar DynamoDBLocal.jar -sharedDb -dbPath ./data"
   image: "amazon/dynamodb-local:latest"
   container_name: dynamodb-local
   ports:
     - "12345:8000"
   volumes:
     - "{원하는 위치}:/home/dynamodblocal/data"
   working_dir: /home/dynamodblocal
```

dynamo는 8000을 듣게되어있고,
이게 로컬 dynamo 엔진으로 돌아감.

ouports에 공통 DynamoClient 만들고 'local', 'endpoint'로 로컬 dynamo 연결.

로컬 Dynamo 연결. 
- Operation builder
- add new connection: DynamoDB local에 도커 설정한 포트로 연결.
- 테이블 볼 때도 여기서 보면 됨.

Data modeler에서 export하면 json 파일 만들어주고, 이걸 workbanch에서 처음에 import로 넣어주면 됨.
Visualizer는 commit to DynamoDB로 원하는 테이블 각각 Local PC로 연결해야함.
