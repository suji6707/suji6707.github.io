+++
title = 'RAG Concept'
date = 2025-03-26T22:50:44+09:00
draft = true
+++
### RAG Concept

RAG
한번에 데이터를 다룰 수 없다.
큰 데이터를 질문할 때마다 업로드하는 건 비효율적.

쿼리 엔진
- retriever: 인덱스에서 맥락을 가져온다.
- postProcessing: LLM에 보내기 전에 노드를 처리
- synthesizer: 처리된 노드들을 결합. 하나의 프롬프트를 LLM에 전달한다.

"쿼리 엔진"
문서를 벡터화 해서 저장한다음 
원하는 부분을 찾는 것.

문서가 3만자이면, 그걸 프롬프트에 다 넣을수가 없기 때문에.

* 순서: 
- documents를 가져와서 vector index를 만든다.
    - 문서들을 임베딩 모델에 통과시켜 각 문서를 벡터로 변환
    - 벡터화된 문서들로 인덱스 구축
- 그 인덱스로 query engine을 만든다.
- 이 쿼리 엔진에 query 한다.

* 과정:
- 문서 내용이 index에 있고 이걸로 retriever를 만든다
- 이 retriever와 synthesizer(응답을 만드는)를 통해 Query engine을 만들고
여기에 쿼리한다.
- 이게 내가 저장해둔 문서에서, 쿼리를 해서 LLM을 통해 답변을 가져오는 것이다.



Advanced queries with Agents
실제 웹 서비스는 여러 데이터 소스를 가지고 있다.
you can create each data source seperately,
LLM에 각 데이터소스가 뭘 하는지 알려주고,
LLM은 어떤 데이터소스에 쿼리를 해야할지 결정할 수 있음.

queryEngine의 description: LLM이 쿼리시 어떤 Tool을 사용할 수 있을지 정의.
LLM이 알아서 pick the tool 해서 답변함.

보통 베이스 LLM 모델은 수학 못하는데
sumFuncionTool 만들어서 그 안에 properties 넣어주면
잘 작동함..
-> name: "sumNumbers"라는 내가 정의한 함수를 실행하게 됨. 
new FunctionTool(함수 및 함수에 대한 설명을 인자로 넘김)

🍊 다른 하나는 쿼리 엔진들을 묶었음.
LLM은 description, name을 보고 어떤 tool을 쓰는게 더 적절한지 판단해서 감.
React ? 혹은 덧셈 ? 
어떤 함수를 써야겠다- 혹은 어떤 쿼리엔진을 써야겠다-
