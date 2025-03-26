+++
title = 'DBT'
date = 2024-07-22T00:20:22+09:00
draft = true
+++


📗 DBT 
🔺 Jinja 템플릿을 사용한 SQL 쿼리 - 동적인 쿼리. 
{% set lowmall = ['low[1].mallPid', 'low[2].mallPid', 'low[3].mallPid'] %}

{% for item in lowmall %}
    SELECT DISTINCT({{ item }}, id)::STRUCT(mallPid VARCHAR, nmid VARCHAR) AS a
    FROM '{{ parquetPath }}'
    WHERE {{ item }} IS NOT NULL
    {% if not loop.last %}
        UNION
    {% endif %}
{% endfor %}

🔺 materialized view
dbt가 해당 모델을 데이터베이스에서 뷰로 생성
-> 실제 데이터를 저장하지 않고 쿼리를 실행할 때마다 최신 데이터를 조회.
근데 이 뷰를 특정경로에 COPY 함으로써 실체화된 뷰를 물리적으로 저장할 수 있음. 


🔺 DuckDB struct


🔺 COPY는 파일시스템과 데이터베이스간 상호작용임.
COPY lineitem FROM 'lineitem.csv'; -> csv 파일을 읽어 lineitem 테이블로 로드

🔺 Dbt는 ELT - 변환만 한다.
Dbt only transforms data. It will not push data from S3 to Snowflake. It will not read from MongoDB and write to Postgres. That’s not its job.
