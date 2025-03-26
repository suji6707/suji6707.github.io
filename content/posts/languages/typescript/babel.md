+++
title = 'Babel, tsc'
date = 2024-07-30T00:20:22+09:00
draft = true
+++

💎 
tsconfig: 타입체크만 수행하도록 설정
babel에서 실제 변환(transfile) 수행.

"noEmit": true
이 옵션은 TypeScript 컴파일러에게 코드를 트랜스파일하거나 타입 선언 파일을 생성하지 말라고 지시한다. 즉, tsc는 타입 체킹만을 수행하고, 실제 코드의 변환은 babel에게 맡기는 것이다.

"isolatedModules": true
이 옵션은 TypeScript 컴파일러에게 각 파일을 모듈로 취급하라고 지시한다. 즉, 각 파일을 모듈로 취급하면 파일 간의 의존성을 추적할 수 있고, 이를 통해 더 빠른 타입 체킹을 수행할 수 있다.


@babel/preset-typescript 은  @babel/plugin-transform-typescript 플러그인을 포함

---

## 💎💎💎 중요. babel vs ts eslint, 그리고 tsc 💎💎💎
바벨은 ts -> js로 변환은 가능하나 타입검사를 해주지 않음.
그래서 tsconfig로 js 파일은 생성하지 않되 타입 d.ts을 생성하도록 명시.

바벨- ts를 eslint 적용을 못하는구나-

build될 때 eslint를 적용시켜야한다.

* d.ts는 tsc가 만들어줌.
ts를 빌드하면 js가 되는 과정에서 d.ts가 생성되어 외부 js,ts에서 쓸때 
그 d.ts 파일을 보고 추론하게 함.

---
🔹 스텝을 밟을 때마다 항상 되는지 확인
안되었다면 방금 한 걸 되돌리거나, 아니면 더 나아가거나 둘 중 하나.

문제를 정확히 파악.
tsc 빌드가 안되는 거란걸.
js 파일이 생겨야하는데 gulp에 .pipe로 ts를 넣는것만으론 안생김. ts({})만 하면 되는데 config를 넣는순간 안됨..
-> gulp는 watch 모드이고 tsc는 빌드라서 뭔가 호환이 안되는.
-> commit시 검사를 하자!

husky. 
- git push 단계에서 스크립트 실행 .husky/sh
- sh 실행시 tsc를 하는데, noEmit 옵션으로.

* 트랜스파일링은 babel, 타입은 tsc
js는 바벨 preset-typescript가 만드는거고
typescript는 타입검사, .d.ts만 생성하면 됨. 따라서 noEmit.
🔺 어쨌든 일반 nestjs와 달리 server에서는
d.ts를 생성하지 않고 내부적으로 빌드/타입검사할때만 로직상 거치는듯.

* tsconfig
"compilerOptions": {
  // tsc를 사용해 .js 파일이 아닌 .d.ts 파일이 생성되었는지 확인합니다.
  "declaration": true,
  "emitDeclarationOnly": true,
  // Babel이 TypeScript 프로젝트의 파일을 안전하게 트랜스파일할 수 있는지 확인합니다.
  "isolatedModules": true
}


(참고)
tsconfig
"typeRoots": ["./node_modules/@types"],       
"types": ["./types", "jest"], 

{
  "compilerOptions": {
    // 사용자 정의 타입과 @types 모두 사용
    "typeRoots": [
      "./node_modules/@types",
      "./custom-types"
    ],
    // 특정 패키지만 포함
    "types": [
      "node",
      "jest",
      "custom-type-package"
    ]
  }
}