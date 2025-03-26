+++
title = '고급 함수들'
date = 2024-10-21T00:20:22+09:00
draft = true
+++


---
### Function 일급 객체

👗👗👗 고차함수. 👗👗👗
Request.get 안에 Request.proxyVerbFunc가 리턴할 함수를 숨겨놨던 것. 
그래서 이미 Reuest.get 필드를 정의할 때 proxyVerbFunc 인자를 넣어놨고,
Request.get를 부를 때 proxyVerbFunc가 리턴한 함수에 넣을 인자를 넣으면 됨.

💎💎💎💎💎💎💎💎💎💎 
doSequential : 프로미스를 리턴할 함수.
- realMain()을 doSequential을 써서 해보기
main()을 year, quarter를 인자로 받도록 해서 
funcs 배열에 넣어서 - 
+ main을 가상함수로 감싸줘야. 
💎💎💎💎💎💎💎💎💎💎 

cur - 현재 실행할 함수
cur()
acc - 초기값은 그냥 프로미스. 
현재 cur()의 실행 결과가 acc

acc 프로미스가 실행된 후 then cur() 실행,
curl()이 실행되면 acc가 됨
새로운 함수의 실행결과로 고쳐줌.

curl()은 최소 프로미스.


---
### 직렬화

왜이렇게 시간이 오래걸리니?

data를 보낼 때 string으로 보낼 순 없다.

* string and Uint8Array 의 차이
- string은 연속된 char. encoded in UTF-16
- Uint8Array: 8-bit unsigned integers. 바이너리 데이터 다룰 때 .
files or blobs. Uint8Arr 
> expects its data in a binary format (Uint8Array), not in a string format. 
🔹 "파일 업로드할 때 대부분 바이너리로 핸들링된다"
- Convert Obj to Uint8Array
1. serialize : JSON.stringify()
2. Encode the string : TextEncoder를 사용하여 Uint8Array로 변환


참고)  UTF-8은 문자 인코딩 체계로, 특히 텍스트 데이터를 위한 것
정수는 일반 8bit, 16bit로 표현됨

JavaScript에서 문자열은 UTF-16 인코딩을 사용

* 직렬화란?
객체 상태를 전송가능 형식으로 변환하는 것.
객체 -> JSON.
JSON은 가장 보편적인 직렬화 형식. 

* Buffer 객체는 이진데이터 다룰 때 사용.
Buffer.from() 

