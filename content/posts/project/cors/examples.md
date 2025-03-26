+++
title = 'CORS Examples'
date = 2024-07-22T00:20:22+09:00
draft = true
+++


🟢 이미지검색 CORS 문제 🟢 
확장프로그램이 iframe으로 들어가고 - 스크립트
익스텐션 식별자의 어느 위치에 들어가는건데. 
이경우 /extension이라 통과가 되는데

네이버 이미지는 따로 iframe이 아니라서
origin이 네이버,1688 이렇게 됨.
-> 
이미지 키 받는게 content script에서 보내고있음.
반면 iframe은 크롬익스텐션 내부 파일을 여는거라 크게 문제 없는데.

이제 content script js에서 서비스워커에 요청하고 거기서 fetch로 보내면 대체로 해결될듯.
서비스워커는 origin을 딱히 갖고있지 않을거라.

윈도우에서 각자 나갈때 vs 백그라운드에서 나갈때 차이 있음.
백그라운드로 통일하면 어떨까.
전자에선 axios인데 후자이면 그게 안돼서 fetch로만 나가고.
같은 API 요청인데 두번 작성하곤 했었음.


---
이미지검색 CORS 뭐가 문제였을까?
- 익스텐션 -> 알리바바 -> itemscout 이미지 POST 요청인데
알리바바에서 아스로 요청하려면 origin을 아스로 바꿔야하는 상황.
그러려면 익스텐션 manifest permission에 알리바바를 추가해야함.

*
manifest.json의 permissions에 도메인을 명시적으로 선언하는 것은
- 익스텐션이 접근할 수 있는 도메인을 명확히 제한
- 사용자가 익스텐션 설치 시 어떤 도메인에 접근하는지 확인 가능
- 무분별한 외부 접근을 방지하는 것.

* origin 설정
CORS는 다른 도메인에서 오는 요청에 대해 적용됩니다. 만약 요청이 같은 도메인에서 발생한다면 CORS가 적용되지 않으므로 origin이 찍히지 않을 수 있습니다.

app.use((req, res, next) => {
    if (whitelistPaths.some(path => req.path.startsWith(path))) {
        whitelistCors(req, res, next);
    } else {
        defaultCors(req, res, next);
    }
});

💎 클라에서 해야함. 