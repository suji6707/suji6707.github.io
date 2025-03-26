+++
title = '데이터분석'
date = 2024-12-27T00:20:22+09:00
draft = true
+++
# Duckdb + snowflake csv results

### 🔮🔮 DB에 있는 키워드와 JOIN 하는 법
string으로 바꾼 후 keyword_id IN () 에 해당하는 데이터로 테이블 만들어서 
그다음에 조인하기. 일단 결과테이블의 keywordIds 만 가져오면됨.

1. csv에서 모든 keyword_id를 읽어 또다른 csv에 쓴다
string으로 바꿈.
```sql
COPY (
select string_agg(keyword_id) as keyword_ids from read_csv('/Users/heojisu/Downloads/Snowflake_ITEMSCOUT_queryResult/backup/keywords_search_above_10k.csv')
) TO '/Users/heojisu/Downloads/Snowflake_ITEMSCOUT_queryResult/backup/keywords_above_10k.csv';
```
2. prd itemscout DB에 접속해서 해당 kid, keyword csv로 저장.
```sql
ATTACH 'host=prd-itemscout-db-cluster.cluster-ro-c8jjtbsbb1is.ap-northeast-2.rds.amazonaws.com user=view password=dkdlxpatmzkdnt11!! port=3306 database=item_scout' AS mysqldb (TYPE MYSQL, READ_ONLY);

COPY (
select id, decode(keyword) as keyword
from mysql_query('mysqldb',
    'select id, keyword from keywords
    where id in ()'
)) TO '/Users/heojisu/Downloads/Snowflake_ITEMSCOUT_queryResult/backup/keyword_ids_above_10k.csv';
```
-> 8만자는 길어서 python 스크립트로 ㄱ. 

3. csv끼리 조인해서 저장
```sql
COPY (
WITH keywords AS (
select * from read_csv('/Users/heojisu/Downloads/Snowflake_ITEMSCOUT_queryResult/backup/keyword_ids_above_10k.csv')
),
growth_ratio AS (
select * from read_csv('/Users/heojisu/Downloads/Snowflake_ITEMSCOUT_queryResult/backup/prd_and_search_yearly_with_ratio_only_valid_2024_search.csv')
)
select k.id, k.keyword, gr.search_cnt_2024, gr.recent_prd_cnt_avg, gr.avg_search_cnt_growth, gr.avg_prd_cnt_growth, gr.ratio 
from growth_ratio as gr
join keywords k ON gr.keyword_id = k.id
) TO
'/Users/heojisu/Downloads/Snowflake_ITEMSCOUT_queryResult/backup/final_ratio_only_valid_2024_search.csv';
```

ATTACH 'host=localhost database=mysql' AS mysqldb (TYPE MYSQL);

---
## 파이썬 Duckdb 쿼리 스크립트
수 만 개의 keywordId를 IN으로 넣어야 할 때.
```py
import duckdb
import pandas as pd
from typing import List


def export_keywords_to_csv():
    keyword_ids_str = (
        pd.read_csv(
            "/Users/heojisu/Downloads/Snowflake_ITEMSCOUT_queryResult/backup/keywords_above_10k.csv",
            nrows=1,
        )
        .iloc[0]
        .values[0]
    )
    # keyword_ids_str = keyword_ids_str[:100]
    # print(keyword_ids_str)

    # 문자열을 리스트로 변환
    keyword_ids = keyword_ids_str.split(",")
    batch_size = 10000  # 1만개씩 처리

    try:
        conn = duckdb.connect()
        conn.execute("INSTALL mysql")
        conn.execute("LOAD mysql")
        conn.execute(
            "ATTACH 'host=prd-itemscout-db-cluster.cluster-ro-c8jjtbsbb1is.ap-northeast-2.rds.amazonaws.com user=view password=dkdlxpatmzkdnt11!! port=3306 database=item_scout' AS mysqldb (TYPE MYSQL, READ_ONLY)"
        )
        for i in range(0, len(keyword_ids), batch_size):
            batch = keyword_ids[i : i + batch_size]
            str_join = ",".join(batch)

            if i == 0:
                conn.execute(
                    f"""
			COPY (
			select id, decode(keyword) as keyword
			from mysql_query('mysqldb',
				'select id, keyword from keywords
				where id in ({str_join})'
			)) TO '/Users/heojisu/Downloads/Snowflake_ITEMSCOUT_queryResult/backup/keyword_ids_above_10k.csv';
			"""
                )
            else:
                result = conn.execute(
                    f"""
				select id, decode(keyword) as keyword
				from mysql_query('mysqldb', 
					'select id, keyword from keywords
					where id in ({str_join})'
				)
				"""
                ).df()
                result.to_csv(
                    "/Users/heojisu/Downloads/Snowflake_ITEMSCOUT_queryResult/backup/keyword_ids_above_10k.csv",
                    mode="a",
                    header=False,
                    index=False,
                )

            print(f"Processed batch {i//batch_size + 1}: IDs {i} to {i + len(batch)}")
    except Exception as e:
        print(e)
    finally:
        conn.close()


export_keywords_to_csv()

```
