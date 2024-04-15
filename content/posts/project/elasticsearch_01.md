+++
categories = ['ETL']
series = ['Elastic Search']
tags = ['elastic search', 'ETL', 'logstash']
title = "Elastic Search Logstash"
date = 2024-03-30T22:50:44+09:00
draft = true
+++
## Introduction

로그스태시는 파이프라인 도구다.
파이프라인은 데이터를 입력받아 실시간으로 변경하고 이를 다른 시스템에 전달하는 역할을 한다.
파이프라인은 입력, 필터, 출력 세 가지 구성요소로 이뤄진다. (입력, 출력은 필수. 필터는 옵션)

## Logstash
🍎 sql_last_value는 로그스태시에서 관리하는. libexec/plugin/jdbc/last_run
에 저장된 시간을 last value 로, 이후의 변경된 것들만 가져올 때 쓸 수,
https://www.elastic.co/guide/en/logstash/current/plugins-inputs-jdbc.html#_predefined_parameters

<img width="741" alt="image" src="https://github.com/suji6707/insightLINK/assets/111227732/d9b1e69e-c709-4ecd-a580-5124310333cf">


-> 프라이머리 키인 id로 하고싶지만, update를 하면 id는 바뀌지 않으므로
변경내역을 조회하지 못하는 문제가 생김. 
그냥 updatedAt으로 조회하는수밖에. 
-> ORDER BY는 다 필터링 하고나서 하는거라 빨리하는거랑 상관이 없음. 인덱스뿐..


ISODate로 비교. 

(참고) mongoDB id도 앞 4 byte는 timestamp
Hex에서 4 byte는 앞의 8자리. 
1자리가 bit 4개 상태 표현. 2^4 = 16 진수라서

---
### Recipe Project
##### 구조
Next
- EnterURL : 레시피 영상 등록
- SearchResult : Elastic 검색결과
	- 검색창만 따로 컴포넌트로 만들기 
- RecipeText : 요약 텍스트
- Trendy: 인기 레시피



### Elysia
#### MVC pattern
현재 구조
RecipeService
- fetchAndSummarizeRecipes: fetchTranscript - YoutubeTranscript 클래스로부터 .
- url을 입력하는 컨트롤러. 


#### 기타 구현 사항들
- elysia - 타입체크는 옵셔널이라 뒤에 붙일 수밖에 없음
- NotFoundError 클래스처럼 YouTubeError 라는 에러 객체를 만들어봄. 속성은 code, error. 생성자는 message.





---
👗👗👗 현재 상황.👗👗👗 practice/23.puppeteer에 코드 다있음!!
A. POST /recipe - body: url
1. recipe 서비스(fetchAndSummarize 함수, 인자는 url) -> outport 호출
ouport1) outport/ scrap-browser -> puppetier로 subtitles 배열 반환
ouport2) recipe 서비스에서 outport/ claude 호출해 레시피 요약을 받음
2. recipe 서비스에서 DB에 url 정보와 함께!! 저장함. (스키마 수정수정🟠)
3. 로그스태시에서 몽고DB 업데이트된 doc을 Elasticsearch로 넘김
4. elastic에서 해당 doc을 검색해 가져와서 화면에 띄움(프론트)

B. GET /search - query: query
검색하면 화면에 띄움




```typescript
const a: number = 1;
console.log(a) // 1
```

`a` is number


