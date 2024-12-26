
+++
title = 'Trench'
date = 2024-10-31T00:20:22+09:00
draft = true
+++
## Trench
https://docs.trench.dev/api-reference/events-get
나중에 내가 만드는 서비스에 이벤트 넣어야할 때 참조하기.
Node, Kafka, Clickhouse 나한테 가능한 모든 기술스택 활용하고 있어서 공유해봄.
It can handle large event volumes and provides real-time analytics

### vinceanalytics/vince 
Vince is a self hosted alternative to Google Analytics.

---
### 샘플
To track events using the Trench library in a Node.js application, you would typically set up an event tracking system that listens for specific user interactions, such as click events, and then logs these events using Trench's API. Below is a basic example of how you might implement this in a Node.js application:

1. **Install the Trench SDK**: First, ensure you have the Trench SDK installed in your Node.js project. You might need to check the Trench documentation for the exact package name and installation instructions.

2. **Set Up Event Tracking**: Create a simple server that listens for incoming event data and logs it using Trench.

Here's a basic example of how you might implement this:

```javascript
// Import necessary modules
const express = require('express');
const bodyParser = require('body-parser');
const trench = require('trench-sdk'); // Hypothetical SDK import, replace with actual package

// Initialize Express app
const app = express();
app.use(bodyParser.json());

// Initialize Trench client
const trenchClient = new trench.Client({
  apiKey: 'YOUR_TRENCH_API_KEY', // Replace with your actual API key
  endpoint: 'https://api.trench.dev', // Replace with actual endpoint if different
});

// Endpoint to receive click events
app.post('/track-click', (req, res) => {
  const { userId, eventType, eventData } = req.body;

  // Track the event using Trench
  trenchClient.track({
    userId: userId,
    event: eventType,
    properties: eventData,
  })
  .then(response => {
    console.log('Event tracked successfully:', response);
    res.status(200).send('Event tracked successfully');
  })
  .catch(error => {
    console.error('Error tracking event:', error);
    res.status(500).send('Error tracking event');
  });
});

// Start the server
const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
  console.log(`Server is running on port ${PORT}`);
});
```

### Explanation:

- **Express Server**: We use Express to create a simple server that listens for POST requests on the `/track-click` endpoint.
- **Trench Client**: We initialize a Trench client with an API key and endpoint. This is a hypothetical example, so replace `'trench-sdk'` and the initialization details with the actual SDK and configuration details from Trench.
- **Event Tracking**: When a POST request is received, we extract the `userId`, `eventType`, and `eventData` from the request body and use the Trench client to track the event.
- **Response Handling**: We send a response back to the client indicating whether the event was tracked successfully or if there was an error.

### Usage Scenario:

This setup could be used to track click events on a website. For example, you might have a button on your frontend that, when clicked, sends a POST request to the `/track-click` endpoint with details about the click event (e.g., button ID, timestamp, user ID).

Remember to replace placeholders like `'YOUR_TRENCH_API_KEY'` with actual values from your Trench account and adjust the code to fit the actual Trench SDK and API specifications.
---
Q. Trench에 저장된 이벤트 데이터를 검색한다고 했는데, 이벤트를 넣을때는 어떤식으로 하지? 예시

## Trench에 이벤트 데이터 전송하기: 이벤트 생성하기 (/events)

Trench에 데이터를 넣으려면 `/events` 엔드포인트에 `POST` 요청을 보내면 됩니다. 이벤트는 Segment API 스펙을 따르므로 Segment와 유사한 방식으로 이벤트를 구성할 수 있습니다. 

### 엔드포인트

`POST /events`

### 요청 헤더

* **Authorization**: `Bearer <token>` 형식의 Bearer 인증 헤더. `<token>`은 Trench에서 발급받은 **공개 API 키**입니다.
* **Content-Type**: `application/json`

### 요청 본문

요청 본문은 다음과 같은 필드를 포함하는 JSON 객체 배열입니다.

| 필드 | 타입 | 필수 여부 | 설명 |
|---|---|---|---|
| `type` | string | 필수 | 이벤트 유형입니다. (`track`, `identify`, `group` 중 하나) |
| `event` | string | `track` 유형에 필수 | 이벤트 이름입니다. (`Product Viewed`, `User Signed Up` 등) |
| `userId` | string | 선택 | 사용자를 식별하는 고유 ID입니다. |
| `anonymousId` | string | `userId`가 없는 경우 필수 | 익명 사용자를 식별하는 고유 ID입니다. |
| `groupId` | string | 선택 | `group` 유형에 필수, 사용자 그룹을 식별하는 고유 ID입니다. |
| `traits` | object | `identify` 유형에 선택 | 사용자 또는 그룹에 대한 추가 정보를 나타내는 객체입니다. |
| `properties` | object | `track` 유형에 선택 | 이벤트와 관련된 추가 정보를 나타내는 객체입니다. |
| `timestamp` | string | 선택 | 이벤트가 발생한 시간입니다. (ISO 8601 형식) |

### 예시

#### 1. 상품 조회 이벤트 (`track`)

```json
{
  "events": [
    {
      "type": "track",
      "event": "Product Viewed",
      "userId": "user123",
      "properties": {
        "productId": "product456",
        "productName": "Awesome T-Shirt",
        "category": "Apparel"
      }
    }
  ]
}
```

#### 2. 사용자 정보 업데이트 이벤트 (`identify`)

```json
{
  "events": [
    {
      "type": "identify",
      "userId": "user123",
      "traits": {
        "email": "user123@example.com",
        "firstName": "John",
        "lastName": "Doe",
        "plan": "premium"
      }
    }
  ]
}
```

#### 3. 그룹 이벤트 (`group`)

```json
{
  "events": [
    {
      "type": "group",
      "userId": "user123",
      "groupId": "group456",
      "traits": {
        "name": "Acme Corporation",
        "industry": "Software"
      }
    }
  ]
}
```

### 응답

#### 204 - 성공

이벤트가 성공적으로 수신되면 서버는 응답 본문 없이 `204 No Content` 상태 코드를 반환합니다.


---
### Clickhouse Kafka 연동

ClickHouse에서 Kafka를 사용하여 데이터를 관리하는 방법은 Kafka 엔진을 활용하는 것입니다. 이 엔진은 Apache Kafka와 통합되어 실시간 데이터 스트리밍을 처리할 수 있습니다. 아래는 ClickHouse에서 Kafka를 설정하고 관리하는 방법에 대한 설명입니다.

### Kafka 테이블 생성

Kafka 엔진을 사용하여 ClickHouse에서 테이블을 생성할 수 있습니다. 이 테이블은 Kafka 토픽으로부터 데이터를 소비합니다.

```sql
CREATE TABLE [IF NOT EXISTS] [db.]table_name [ON CLUSTER cluster](
    name1 [type1] [ALIAS expr1],
    name2 [type2] [ALIAS expr2],
    ...
) ENGINE = Kafka()
SETTINGS
    kafka_broker_list = 'host:port',
    kafka_topic_list = 'topic1,topic2,...',
    kafka_group_name = 'group_name',
    kafka_format = 'data_format'
    ...
```

#### 필수 설정

- **`kafka_broker_list`**: Kafka 브로커의 주소 목록 (예: `localhost:9092`).
- **`kafka_topic_list`**: Kafka 토픽 목록.
- **`kafka_group_name`**: Kafka 소비자 그룹 이름. 동일한 그룹 이름을 사용하면 메시지가 중복되지 않도록 관리됩니다.
- **`kafka_format`**: 메시지 형식 (예: `JSONEachRow`).

#### 선택적 설정

- **`kafka_num_consumers`**: 테이블당 소비자 수. 기본값은 1이며, 토픽의 파티션 수를 초과하지 않아야 합니다.
- **`kafka_max_block_size`**: 한 번의 poll에서 가져올 최대 메시지 수.
- **`kafka_skip_broken_messages`**: 파싱할 수 없는 메시지를 건너뛰는 수.
- **`kafka_commit_every_batch`**: 각 배치마다 커밋할지 여부.
- **기타 여러 설정**: 메시지 처리 및 소비자 관리에 관련된 다양한 설정이 있습니다.

### Kafka와 Materialized View

Kafka 테이블은 직접 `SELECT` 쿼리로 읽기보다는 Materialized View를 통해 데이터를 다른 테이블로 변환하여 저장하는 것이 일반적입니다.

```sql
CREATE TABLE queue (
    timestamp UInt64,
    level String,
    message String
) ENGINE = Kafka('localhost:9092', 'topic', 'group1', 'JSONEachRow');

CREATE TABLE daily (
    day Date,
    level String,
    total UInt64
) ENGINE = SummingMergeTree(day, (day, level), 8192);

CREATE MATERIALIZED VIEW consumer TO daily
AS SELECT toDate(toDateTime(timestamp)) AS day, level, count() as total
FROM queue GROUP BY day, level;
```

- **Materialized View**: Kafka 테이블로부터 데이터를 읽어 다른 테이블에 저장합니다. 실시간으로 데이터를 수집하고 변환할 수 있습니다.

### Kafka 설정

Kafka 엔진은 ClickHouse 설정 파일을 통해 전역 및 토픽 수준의 설정을 지원합니다. 예를 들어, `<kafka>` 섹션에서 전역 설정을 지정할 수 있습니다.

```xml
<kafka>
    <debug>cgrp</debug>
    <statistics_interval_ms>3000</statistics_interval_ms>
    <kafka_topic>
        <name>logs</name>
        <statistics_interval_ms>4000</statistics_interval_ms>
    </kafka_topic>
    ...
</kafka>
```

### Kerberos 지원

Kerberos를 사용하는 Kafka와 통합하려면 `security_protocol` 및 관련 설정을 추가해야 합니다.

```xml
<kafka>
    <security_protocol>SASL_PLAINTEXT</security_protocol>
    <sasl_kerberos_keytab>/path/to/keytab</sasl_kerberos_keytab>
    <sasl_kerberos_principal>principal_name</sasl_kerberos_principal>
</kafka>
```

### 요약

ClickHouse의 Kafka 엔진은 Kafka로부터 데이터를 실시간으로 소비하고, 이를 ClickHouse의 강력한 분석 기능과 결합할 수 있도록 해줍니다. Kafka 테이블은 Materialized View와 함께 사용하여 데이터를 다른 테이블로 변환하고 저장하는 방식으로 주로 사용됩니다. 이를 통해 실시간 데이터 스트리밍과 분석을 효율적으로 수행할 수 있습니다.

---
