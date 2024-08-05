+++
categories = ['Database']
series = ['Database-System']
tags = ['데이터베이스']
date = 2024-07-30T22:50:44+09:00
draft = true
+++
## 빅데이터

🏞️ 빅데이터 저장
1. 분산파일 시스템
싼 컴퓨터를 수평적으로 확장. 일관성을 보장하지 않으므로 데이터를 복제 저장한다.(보통 3 machine)

뒷단에선 분산인데
앞단에선 single application 처럼 동작.

HDFS 파일시스템 API가 뒷단 분배를 해주는거고,
클라이언트는 메타데이터를 읽어서 특정 블록을 찾아감.

한 블록에 64KB인데 이것보다 작은 데이터가 무수히 많다면 오히려 비용 비싸질 수.

2. 샤딩
3. 키밸류 스토어
다이나모, S3는 byte를 그대로 저장하고,
반면 카산드라 등은 테이블이 있음.

확장성과 MySQL은 상충?
대규모 클러스터에서 MySQL 기능들을 제공하기가 쉽지 않음


🏞️ 복제 vs 일관성도 상충.
Write는 복제된 3군데를 고쳐야하는데 3개중 2개만 업데이트 성공해도 넘어간다.



```sql
CREATE VIEW mallpid_nvmid_map AS (
(
SELECT a.mallPid, a.nvMid, 1 AS GROUPED FROM (
   SELECT  DISTINCT(lowPriceByMallNo[1].mallPid, id)
   ::STRUCT(mallPid VARCHAR, nvMid VARCHAR) AS a FROM '${parquetPath}' 
      WHERE lowPriceByMallNo[1].mallPid IS NOT NULL))
   UNION (SELECT a.mallPid, a.nvMid, 1 AS GROUPED FROM  (
   SELECT  DISTINCT(lowPriceByMallNo[2].mallPid, id)
   ::STRUCT(mallPid VARCHAR, nvMid VARCHAR) AS a FROM '${parquetPath}' 
      WHERE lowPriceByMallNo[2].mallPid IS NOT NULL))
```