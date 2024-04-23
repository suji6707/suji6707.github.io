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
### 🍊 전반적인 기능
홈, 검색, My페이지, 설정
- 홈: 추천 기반. 클릭시 원본 영상과 레시피 요약이 뜸
- 검색: 한글자씩 입력할때마다 관련 영상 썸네일과 타이틀 리스트가 뜸
	- 추후 벡터 데이터베이스를 활용해볼 수도 있는 부분
- My페이지: 좋아요/팔로우 기반

추가 구현
- MLOps: 데이터만 갈아끼우면 모델링부터 배포까지 자동화되도록.
처음에는 유저 데이터가 없어서 힘들고,
우선 유사 그룹 기반(+가입시 취향 선택하게)으로 한 후 데이터가 모이면 도전.

#### 🫐 TODO
- 1분요리 뚝딱이형, @1mincook 과 같은 채널명/채널ID를 주면
- 퍼페티어로 동영상 탭을 클릭해 스크롤을 내리면서 className `ytd-rich-grid-renderer` `<a>` 태그 url을 가져와 배열에 담도록 함
- DB 스키마는 채널id, url []

#### 백엔드 구조
-  RecipeService
	- 크롤링 + 요약 + 저장 : fetchAndSummarizeRecipes
- channel 입력시 해당 채널의 url들을 긁어오는 API
	- 관련 테이블 구조: One to Many

```
{
  _id: ObjectId("507f1f77bcf86cd799439011"),
  channelName: "채널명",
  channelID: "UCBR8-60-B28hp2BmDPdntcQ",  // 유튜브 채널 ID
  description: "채널 설명"
}

{
  _id: ObjectId("507f1f77bcf86cd799439011"),
  channelID: "UCBR8-60-B28hp2BmDPdntcQ",  // 유튜브 채널 ID
  urls: [
    "https://www.youtube.com/watch?v=xyz123",
    "https://www.youtube.com/watch?v=abc456",
    // 추가 URL들
  ]
}
```

하나의 영상 url에 대한 크롤링이 있다면
하나의 채널에 대한 url들을 스크래핑하는 기능.
-> 특정 채널에 대한 url들을 One to Many로 저장해두고


#### 프론트 컴포넌트 구조
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


