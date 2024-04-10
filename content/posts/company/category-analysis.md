+++
title = '카테고리 분석기'
date = 2024-02-25T00:20:22+09:00
draft = true
+++
## Introduction

This is **bold** text, and this is *emphasized* text.

#### 질문
- 인기 키워드가 없는 서브카테고리의 경우 결과에서 제외해야하는데
for문을 돌 때 continue로 스킵하려면 그 아랫단에서 계속 null을 리턴하면서 최종 맨 바깥 for문에서 if (! ) continue를 해야하는 번거로움.. 여러 레이어를 타야해서 어쩔수없는건가.
- DB에서 keywordIds를 봤을때 빈배열이면 throw Error를 하면 젤 편하겠지만



#### 에러 처리
- 오히려 컨트롤러 말고 서비스단에서 throw Error하면 될듯

#### DB / 캐싱
- 크롤링 전에, categoryId가 category_sales_aggregate 테이블에 있고 year, month가 같으면 해당 항목 보내주는걸로 바꿔야함.

---
#### Nest CLI and scripts
왜 nest start를 해야 빌드 후 watch가 제대로 작동할까?
- Nest는 타입스크립트 기반이라 실행 전 JS로 먼저 컴파일되어야하는데
- Nest CLI를 보면 nest build시 ts 컴파일러가 JS로 변환하며(이때 tsconfig.json 참조)
- nest start를 함으로써 빌드된 JS를 실행한다. 
- 여기서 nest start는 빌드가 되어있음을 ensure(빌드과정도 포함)하므로, 애초에 pnpm dev: nest start --watch 로 넣어버리면 젤 편하다.


```typescript
const a: number = 1;
console.log(a) // 1
```

`a` is number
