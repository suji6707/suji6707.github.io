+++
categories = ['KV store']
title = "DynamoDB"
date = 2024-10-06T22:50:44+09:00
draft = true
+++
## DynamoDB 기초

리소스 인터페이스를 사용한 동일한 쿼리 작업을 줄이고 단순화할 수 있습니다.

```python
import boto3
from boto3.dynamodb.conditions import Key, Attr

dynamodb = boto3.resource('dynamodb')
table = dynamodb.Table('YourTableName')

response = table.query(
    KeyConditionExpression=Key('pk').eq('id#1') & Key('sk').begins_with('cart#'),
    FilterExpression=Attr('name').eq('SomeName')
)
```

---
이미 `requestId`가 정렬키로 사용되고 있고, 파티션키로는 임의의 고정 문자열을 사용하고 있는 경우, 추가적인 인덱스(Global Secondary Index, GSI)를 사용하는 것이 효율적인 방법일 수 있습니다. GSI를 사용하면 기존 테이블의 파티션키와 정렬키 외에 다른 속성을 기준으로 데이터를 조회할 수 있습니다.

### Global Secondary Index (GSI) 설정

GSI를 사용하여 `type`과 `viewName`을 인덱스로 설정하면, 특정 타입의 모든 뷰 이름을 효율적으로 조회할 수 있습니다.

### 수정된 데이터 구조
기존 데이터 구조는 그대로 유지하고, GSI를 추가합니다:
```yaml
- id: 고유 식별자 (UUID 등)
- requestId: 요청 식별자 (정렬키)
- query: 쿼리 내용
- createdAt: 생성 시간
- type: 쿼리 타입 (일반 쿼리 또는 VIEW)
- viewName: 뷰 이름 (VIEW 타입인 경우에만 사용)
- dataSource: 데이터 소스 (MySQL / S3)
```

### GSI 설정
- **인덱스 이름**: `TypeViewNameIndex`
- **파티션키**: `type`
- **정렬키**: `viewName`

### DynamoDB 테이블 생성 예시 (AWS CLI)
```sh
aws dynamodb create-table \
    --table-name YourTableName \
    --attribute-definitions \
        AttributeName=requestId,AttributeType=S \
        AttributeName=type,AttributeType=S \
        AttributeName=viewName,AttributeType=S \
    --key-schema \
        AttributeName=fixed,KeyType=HASH \
        AttributeName=requestId,KeyType=RANGE \
    --global-secondary-indexes \
        "[
            {
                \"IndexName\": \"TypeViewNameIndex\",
                \"KeySchema\": [
                    {\"AttributeName\":\"type\",\"KeyType\":\"HASH\"},
                    {\"AttributeName\":\"viewName\",\"KeyType\":\"RANGE\"}
                ],
                \"Projection\":{
                    \"ProjectionType\":\"ALL\"
                }
            }
        ]" \
    --provisioned-throughput ReadCapacityUnits=5,WriteCapacityUnits=5
```

### Python 코드로 GSI를 통한 조회 예시
DynamoDB 테이블에 GSI를 추가한 후, 특정 타입의 모든 뷰 이름을 조회하는 파이썬 코드는 다음과 같습니다:

```python
import boto3
from boto3.dynamodb.conditions import Key

# DynamoDB 클라이언트 생성
dynamodb = boto3.resource('dynamodb')
table = dynamodb.Table('YourTableName')

def get_views_by_type(view_type):
    response = table.query(
        IndexName='TypeViewNameIndex',
        KeyConditionExpression=Key('type').eq(view_type)
    )
    views = response.get('Items', [])
    return views

# 특정 타입의 VIEW 이름들 가져오기
view_type = 'VIEW'
views = get_views_by_type(view_type)

for view in views:
    print(view['viewName'])
```

### 결론
- 기존 테이블 구조를 유지하면서도 효율적으로 조회할 수 있도록 GSI를 추가하는 방법을 사용합니다.
- GSI를 사용하여 `type`과 `viewName`을 인덱스로 설정하면 특정 타입의 모든 뷰 이름을 효율적으로 조회할 수 있습니다.
- AWS CLI를 통해 GSI를 설정하고, 파이썬 코드를 통해 GSI를 사용하여 데이터를 조회할 수 있습니다.