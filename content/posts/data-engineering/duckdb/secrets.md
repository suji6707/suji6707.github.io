+++
title = 'DuckDB Secrets'
date = 2024-07-22T00:20:22+09:00
draft = true
+++

DuckDB Secrets

타입이 있음.
각 타입의 시크릿은 "시크릿 제공자(secret providers)"에 의해 생성됩니다. 시크릿은 또한 선택적으로 "스코프(scope)"를 가질 수 있는데, 이는 시크릿이 적용되는 파일 경로의 접두사를 의미합니다

Secret Providers
시트릿을 생성하는 메커니즘. S3, GCS 등 여러 시크릿 타입이 있고 
DuckDB는 두 프로바이더를 지원 - CONFIG, CREDENTIAL_CHAIN
- CONFIG: 모든 정보를 create secret으로 (temporary secrets)
- CREDENTIAL_CHAIN: 자동으로 크리덴셜을 가져온다.
시크릿 프로바이더가 명시되지 않으면 CONFIG가 사용됨.
(serverless의 경우 그렇지 않나봄!)

🍎 신기한건
전혀 시크릿이 설정되어있지 않아야한다는것.

FROM which_secret('s3://itemscout-data-lake/db/keyword_product_counts/*/*/*.parquet', 's3');


name=itemscout_secret;type=s3;provider=config;
serializable=true;
scope=s3://,s3n://,s3a://;endpoint=s3.amazonaws.com;
key_id=AKIAY44DNSVNJLQZ3JJU;region=ap-northeast-2;s3_url_c…


