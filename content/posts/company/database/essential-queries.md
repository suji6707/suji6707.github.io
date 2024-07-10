+++
categories = ['Company']
series = ['Database']
tags = ['Essential-Queries']
title = "Essential-Queries"
date = 2024-03-30T22:50:44+09:00
draft = true
+++
## Essential-Queries

#### 랭킹
- 랭킹정보 필요. 유저-상품-키워드 조합을 랭킹 테이블에서 찾아서 반환한다. 특정 날짜 기준.
```sql  
select rkp.* from user_tracking_keywords utk
join ranking_keywords_products rkp
on utk.tracking_product_id = rkp.tracking_product_id
and utk.keyword_id = rkp.keyword_id
where utk.user_id = 2
and rkp.created_at >= '2024-05-16 00:00:00'
order by rkp.created_at desc
```

#### LEFT JOIN
tag_id = 0이어도 되는 이유.
- 사실상 user_trackings에 전부 맞추겠다는 거라서.
user_trackings이 user_product_tags를 포함하는 개념이고
(집합 개념. right join은 항상 ut의 모든 칼럼이 존재)
- 그룹이 있으면 user_product_tags의 칼럼들이 채워지고
- 없으면(upt에 해당 tpId가 없으면) upt 칼럼들은 전부 NULL값으로 뜸.

```sql
  SELECT
      ut.*,
      upt.*
  FROM
      user_trackings ut
          LEFT JOIN user_product_tags upt
              ON ut.tracking_product_id = upt.tracking_product_id
              AND upt.tag_id = 0  // 79
  WHERE ut.user_id = 2
```