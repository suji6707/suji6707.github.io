+++
title = 'Signoz'
date = 2025-03-26T22:50:44+09:00
draft = true
+++

🟠🟠🟠 Signoz 트레이싱 비우기 🟠🟠🟠

* Clickhouse에서 테이블 크기 조회하는 법

use information_schema;

SELECT TABLE_NAME
FROM TABLES;

SELECT
    table AS table,
    formatReadableSize(sum(bytes)) AS size,
    sum(rows) AS rows
FROM
    system.parts
WHERE
    table = 'signoz_index_v2' AND active
GROUP BY
    table;

---
ClickHouse에서 사용하는 데이터베이스 또는 스키마에 대해 문서를 찾고 싶다면, 데이터베이스의 스키마, 테이블, 컬럼에 대한 설명을 포함하는 문서를 검색해야 합니다. 일반적으로 이러한 정보를 찾기 위해 사용할 수 있는 용어는 다음과 같습니다:

1. **ClickHouse Schema Documentation**:
   - ClickHouse의 데이터베이스 및 스키마에 대한 구조와 설명을 찾기 위해 검색할 수 있는 용어입니다.

2. **ClickHouse Table Documentation**:
   - 각 테이블의 목적과 설명을 포함하는 문서를 찾기 위해 사용할 수 있는 용어입니다.

3. **ClickHouse Data Model**:
   - 데이터 모델 전반에 대한 설명, 테이블 간의 관계, 데이터 저장 방식 등에 대한 정보를 찾을 때 유용한 용어입니다.

4. **ClickHouse Data Dictionary**:
   - 데이터베이스나 테이블의 메타데이터와 설명을 포함하는 사전을 찾을 때 사용할 수 있는 용어입니다.

5. **ClickHouse System Tables**:
   - `system` 데이터베이스에 있는 시스템 테이블과 그 설명을 찾기 위해 사용할 수 있는 용어입니다. 이 테이블들은 ClickHouse 인스턴스의 메타정보를 제공합니다.

6. **ClickHouse Table Descriptions**:
   - 특정 테이블의 역할이나 목적에 대한 설명이 필요할 때 사용할 수 있는 용어입니다.

7. **ClickHouse Data Storage**:
   - 데이터 저장 방식 및 각 테이블의 역할에 대한 설명을 찾을 때 유용한 용어입니다.

### 예시 검색어
- "ClickHouse schema documentation for Signoz"
- "ClickHouse table descriptions for logs and traces"
- "Signoz ClickHouse data model"
- "ClickHouse system tables explanation"
- "ClickHouse data dictionary for Signoz schema"

### 추가적인 방법
1. **공식 문서**:
   - ClickHouse의 공식 문서를 참조하면, `information_schema` 및 `system` 데이터베이스에 대한 메타데이터와 설명을 찾을 수 있습니다.
   - [ClickHouse Documentation](https://clickhouse.com/docs/en/)

2. **프로젝트 문서**:
   - Signoz 같은 오픈소스 프로젝트의 공식 문서나 GitHub 리포지토리를 참조하는 것도 좋은 방법입니다.
   - [Signoz Documentation](https://signoz.io/docs/)

3. **데이터베이스 메타데이터 조회**:
   - ClickHouse 클라이언트에서 `DESCRIBE TABLE` 명령어를 사용하여 각 테이블의 구조를 확인할 수 있습니다. 예를 들어:
     ```sql
     DESCRIBE TABLE information_schema.tables;
     ```

4. **SQL 쿼리로 메타데이터 조회**:
   - `system.tables`나 `information_schema.columns` 테이블을 조회하여 각 테이블의 메타데이터와 설명을 확인할 수 있습니다.
     ```sql
     SELECT 
         table,
         column,
         type,
         comment
     FROM 
         information_schema.columns
     WHERE 
         table = 'your_table_name';
     ```

이러한 방법과 용어를 사용하여 필요한 정보를 더 쉽게 찾을 수 있을 것입니다.



(참고) mysql
SELECT 
    table_schema AS database_name,
    table_name,
    ROUND((data_length + index_length) / 1024 / 1024, 2) AS size_in_mb,
    table_rows AS rows
FROM 
    TABLES
ORDER BY 
    size_in_mb DESC;


select TABLE_NAME, ROUND((data_length + index_length) / 1024 / 1024 , 2) from TABLES where TABLE_NAME = 'signoz_index_v2';