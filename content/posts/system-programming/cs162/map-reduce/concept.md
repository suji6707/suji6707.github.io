+++
title = 'Map Reduce'
date = 2025-01-05T00:20:22+09:00
draft = true
+++
# Map Reduce

🌌🌌🌌🌌🌌🌌🌌🌌🌌 Map Reduce 🌌🌌🌌🌌🌌🌌🌌🌌🌌
맵, 리듀스 단계.

1. 입력 데이터를 작은 조각으로 나눈다
- 각 조각에서 key value pair를 생성
- 단어가 키, 출현빈도가 값
2. Shuffle, Sort
- 동일한 키를 가진 pairs를 그룹화
- 올바르게 분배/정렬되도록.
3. 리듀스
- 그룹화된 키-값 pair를 입력으로 받아 각 키에 대해 연산
- 출현빈도를 합산한다거나..

맵리듀스의 데이터 저장
- 하둡 분산파일시스템 등을 사용하여 데이터를 물리적으로 저장함.

A Serverless Query Engine from Spare Parts
- querying parquet files directly in S3 with SQL syntax.
- 

