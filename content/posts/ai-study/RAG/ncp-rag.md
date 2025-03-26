+++
title = 'NCP RAG'
date = 2025-03-26T22:50:44+09:00
draft = true
+++
### NCP 클라우드
유튜브에 다 올라옴

1차

데이터를 검색. chunk 단위로 분할. 
vector store에 저장.
vector로 변환하기 위해 임베딩 사용.

* text chunking.
문단 나누기 등.
표 구조 중요.
-> 문서 데이터 전처리 단계 중요!
데이터 유형별로 구조화하여 저장.

* 임베딩
bge-m3 모델 추천.

* vector DB를 이용한 인덱싱.
RAG 성능에 큰 영향.
정확한 값이아닌 빠른 유사도 검색.
postgreSQL, OpenSearch 무난함.
아주 대규모 아니면.

* 의도분류를 통한 detect, orchestration.
도메인을 추론하고 그 안에서 세부적으로.

ㅡㅡㅡㅡㅡㅡㅡㅡㅡ
2차
