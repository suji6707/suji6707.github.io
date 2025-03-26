+++
title = 'AI Agent'
date = 2025-03-26T22:50:44+09:00
draft = true
+++
### AI Agent

핵심은 어떻게 tool을 부여할 것인가.
전체적인 느낌과 해야할것은 한문장 프롬프트로 될텐데.
특정 기능이 특정 에이전트로, 그 에이전트는 특정 API를 써야한다는 것.
그게 의미가 있을까??

이미지 분석 Agent 호출 (객체 및 배경 영역 파악)
배경 제거 Agent 호출 (배경 제거)
배경 합성 Agent 호출 (자연 풀밭 배경 선택 및 합성)
텍스트 처리 Agent 호출 ( "인기상품" → "Popular Item" 번역, 이미지 오버레이)

1. fal.ai에서 배경제거 API를 호출한다
2.


planing, reflexion, 어떤 tool을 사용할건지.
이 세 가지는 계속 쓰임.

아래 각 기능들에 대한 API 및 호출 함수는 tools에 정의해두되,
어떤 프롬프트를 했을 때 제대로 선택해서 하는지 먼저 봐야.
🍀배경제거
🍀OCR 텍스트 문자 가져오기
🍀이미지 생성해서 넣기
🍀텍스트 넣기(문장생성도)

저장해둘것
{ file name, file path, description }


봐야할 파일 
mastra-image-editor: 에이전트
테스트: fal-ai-test. yolo 참조