+++
categories = ['Javascript']
series = ['Javascript']
date = 2024-07-30T22:50:44+09:00
draft = true
+++

### package.json
ts-node : 컴파일할 필요 없이 바로.
원래는 이게 정석
"compile": "tsc && node ./build/src/app.js",
"start": "nodemon -e ts --exec \"npm run compile\""

// "start": "nodemon -e ts --watch 'src/**/*.ts' --exec 'ts-node' src/app.ts", 

---
## SWC

💎 SWC
Node에서 타입스크립트 실행 가능 옵션: --experimental-strip-types
 
타입스크립트 트랜스파일에 사용하기 위해
Rust로 작성된 SWC 컴파일러를 
언어 중립적인 WASM 모듈로 컴파일해 
Javascript 환경에서 사용할 수 있게 한다.

*Rust 코드를 WASM으로 컴파일하는 명령어
wasm-pack build --target web

-> WSAM 모듈을 Javscript 환경에서 로드
이로써 브라우저나 Node.js 환경에서도 SWC 고성능 컴파일을 사용할 수 있다.

// WASM 모듈 로드
import init, { transformSync } from './path_to_wasm_module/pkg/swc_wasm.js';

const output = transformSync(code, { jsc: { target: "es5" } });


---
원래도 npx add @swc/cli @swc/core 로 SWC를 쓸 수 있었음.
$ npx swc index.ts => ts에서 js로 변환된 코드 출력됨
$ npx swc index.ts -o output.js => 파일에 쓰기


---
- 소스 맵 파일: 
트랜스파일된 JavaScript 코드와 원본 TypeScript 코드 간의 매핑 정보를 포함한 JSON 형식의 파일입니다.
- 매핑 정보: 각 트랜스파일된 코드의 위치가 원본 코드의 어느 위치와 대응되는지에 대한 정보를 포함합니다.