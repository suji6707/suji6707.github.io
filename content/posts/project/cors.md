+++
categories = ['Web']
tags = ['CORS']
title = 'CORS 에러 원인'
date = 2024-04-06T22:50:44+09:00
draft = true
+++
## CORS
Cross Origin Resource Sharing이란 다른 출처간 요청을 허용하는 것.

Same Origin Policy(SOP): 원래 브라우저는 요청 주소가 origin과 다르면 이상하다고 판단해서 응답을 막음. (악의적인 사이트로 요청 등 보안)
그런데 서버로부터의 응답 헤더에 Access-control-allow-origin에 현재 요청 origin이 포함되어있으면 안전한 것으로 판단해 허용.
서버 입장에서 Access-control-allow-origin은 '아니야 괜찮아' 정도가 아니라 괜찮다고 도장까지 찍어주는 거라 보면 됨.
-> CORS는 응답 헤더에 Access-control-allow-origin: 'localhost:3000'을 넣어줌으로써 3000포트에서 8000포트 서버로 요청을 브라우저에서 허용하도록 해주는 것.
참고로 브라우저 Origin을 바꿀 수는 없으므로 서버에서 응답헤더에 Access Allow 도메인을 넣어줌으로써 CORS 에러를 해결해야 함.

중요한건 동일출처 정책은 브라우저에서 막는다는 것.
(그래서 curl 요청시 브라우저를 거치지 않으므로 CORS 에러 X)
서버 입장에선 브라우저를 거치지 않는 악의적인 요청은 다른 방법으로 막아야 함.

주소창에 바로 localhost:8000 서버 주소를 치면 
Origin이 8000인 것으로 간주되기 때문에 
8000 -> 8000 Same origin 요청이 되어 허용됨.

그리고 브라우저에서 현재 origin과 같은 출처로 요청이 가면 
자체적으로 판단해 Response header에 Access-control-allow-origin가 뜨지도 않는다.



![image](https://github.com/suji6707/suji6707.github.io/assets/111227732/042ed50c-8fa2-4a6f-aba2-1fc01698caea)
