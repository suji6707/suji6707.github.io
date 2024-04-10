+++
categories = ['Company']
series = ['Database']
tags = ['TypeOrm']
title = "TypeOrm"
date = 2024-03-30T22:50:44+09:00
draft = true
+++
## Introduction

#### 키워드 및 카테고리 데이터 저장하기
1. 배치 파일 만들기
1) e2e 테스트를 배치처럼 vs Nest standalone application

크롤링은 15초 간격인데 jest는 5초 타임아웃 있어서 후자로 함
-  pnpm build; node dist/~ 로 실행
- 배치간격은 wait 유틸함수 만들어서. 밀리초 이후 () => resolve() 하는 프로미스를 리턴. `await wait(ms);`
참고로 wait 유틸함수 그 자체로는 이후 함수들의 실행을 지연시키지 않는다.
await을 앞에 걸어줬기 때문에 순차적 실행이 가능한 것.
2) 배치를 돌려서 필요한 샘플 데이터를 전부 저장한다
3) mock API 작성: DB 데이터를 가공해서 보내준다

---
2. TypeOrm 저장시 upsert
- save는 원래 select해서 있으면 update, 없으면 insert를 하는데
그 duplicate 판단 기준이 프라이머리 키인듯..
- 나의 경우 keywordId(TypeOrm 엔터티 네임 기준), year, month 복합 유니크키라 
save 외 upsert 메서드를 따로 만들어주었음(entity-domain 매핑 알아서 해주는 BaseTypeOrm 레포지토리에). 

upsert는 삽입 결과만 반환하며, identifier에서 id를 가져온 후 select를 해서 저장된 객체를 반환해야 한다.

```typescript
const a: number = 1;
console.log(a) // 1
```

`a` is number


