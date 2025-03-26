+++
title = 'LangChain Basic Concepts'
date = 2025-03-26T22:50:44+09:00
draft = true
+++
### LangChain Basic Concepts

랭체인 메서드가 많은 이유
LangChain의 핵심 목적은 다양한 데이터 소스와 NLP 도구를 유연하게 연결하고 통합하는 것입니다. PDF 파일, 데이터베이스, API, 웹 페이지, 텍스트 파일 등을 다룰 수 있어야 하기 때문에 그에 맞는 로더(loader)와 처리 유틸리티를 많이 제공합니다.

---
from langchain.prompts import ChatPromptTemplate

Thought: 문제를 해결하기 위해 어떤 작업을 수행해야 할지
Action: 수행할 작업(도구 사용)을 결정
Action Input: Action에 필요한 입력값을 제공. 검색 쿼리 또는 API 요청 등
Observation: Action의 결과로 반환된 데이터를 관찰하여 기록. 에이전트가 다음 단계의 Thought를 결정하는 데 필요한 정보

ResponseSchema
- 특정 데이터를 어떻게 추출할지와 반환 형식
템플릿: 작업 가이드라인. 

TokenTextSplitter: 토큰 단위. 언어모델에 더 적합.

Vector Store Retriver
- 쿼리할 수 있는.

LangChain Expression Language (LCEL) 


RetrieverQA
젤 흔한건 모든 문서 조각을 하나의 컨텍스트로 합치기(Stuffing).
그다음은 맵리듀스. 주로 요약할 때. 각 data chunk에 대해 LLM 결과물 뽑아서 최종.

combine_chain = CombineDocumentsChain.from_chain_type(
    llm=llm,
    chain_type="map_reduce",
)
💊 문서 관련 체인 설명: 
https://teddylee777.github.io/langchain/summarize-chain/


* evaluation
QAEvalChain: grading. 예측은 물론 스코어까지. 
examples를 llm으로 생성한 후 각각에 대해 evaluate_string 해도 되고,
허깅페이스에서 데이터셋 다운받아서 해도 좋음. llm_math 등 많음.

langchain.debug