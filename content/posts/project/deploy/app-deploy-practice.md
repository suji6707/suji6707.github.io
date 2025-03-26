+++
title = '배포 연습'
date = 2024-02-25T00:20:22+09:00
draft = true
+++


# 🔮🔮🔮 서버/웹 배포 🔮🔮🔮
🔺 nginx 두 가지
1. static file serving
2. reverse proxy
보통 프록시는 forword 지만(클플 cdn 같은)
nginx는 외부 인터넷 요청을 받아서 서버들로 뿌려주는 리버스 프록시임.

# proxy_set_header
nginx는 자동으로 Host를 $proxy_host로 바꿔버리는데
자기자신이 궁금한게 아니므로
proxy_set_header 지시자에서 $host로 바꿔줘야함. 
---

http 상태에선 클플 프록싱 꺼놓고.
클플에 의존하지 않는 상태에서 - 

# config 파일 수정
서버 블록이 두 개임.
하나는 api, 다른하나는 클라 정적파일.
각각 server_name이 서브도메인인데, 클플에서 둘다 A 레코드로 IP가 suzee.xyz와 같으면 알아서 서브도메인으로 설정됨.

1. api 서버블록
# default_server
- 원치않는 요청으로부터 방어.
suzee.xyz에 맞지않는 요청이 IP로 들어온다면 
그걸 이 server 블록이 듣고있을 게 아니라 다른곳으로 몰아줘야함.
-> sites-enabled에 있는 default가 `listen 80 default_server;`로 들어주고 있을거임.
- It catch all requests that don't match other servers. It will return a 403 Forbidden (deny all)

2. 클라 
static serving: 클라측 block에서 location에 /index.html을 넣어줘야 작동함.

3. 그외 
nginx.conf.d -> user: root로 변경
- 젤 중요한 파일. 

let's encrypt랑 certbot이랑 같은건데
인증서 해당 도메인으로 받고나서 클플 프록시 켜기

🟧 TODO
구글클라우드 앱 하나더 추가해서 리다이렉트 도메인으로 수정.
기존꺼는 놔두기.



/usr/bin은 Linux 시스템에서 매우 중요한 디렉토리로, 일반 사용자가 사용할 수 있는 실행 파일(바이너리)들이 저장되는 표준 위치

---

암페어 
Canonical Ubuntu 24.04
4코어 24GB
부트볼륨 200GB VPU 20 (high perform)


ssh -i ~/auth/ ubuntu@~~

🔴 TODO 
postgres 도커로 설치
nginx 도커로 설치
suzee.xyz 연결
- 앱 만들때마다 서브도메인으로 파서 ㄱ. 이번엔 linkhub
🔴 클라이언트 배포
🔴 server .env의 클라 URL, 서버 리다이렉트 URL 변경

3306은 외부에 열지말고
서버로 22 ssh로 접속한 후 3306으로 ㄱ. 

```
module.exports = {
  apps : [{
    name         : "main",
    script       : "./dist/src/main.js",
    cwd          : "/home/ubuntu/apps/linkhub/server/",
    watch        : false,
    autorestart  : true,
    env          : {
        NODE_ENV : "productioin"
    },
    exec_mode    : "cluster"
  }]
}
```

# <클라 배포>
pnpm build
dist 폴더에 html css js로 압축되고, 이걸 서버로 올림

/var/www/linkhub/dist 로 배포

location / {
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_pass http://localhost:8080;
}
/home/ubuntu/apps/linkhub/web/dist



build 또 하시고 그 파일 그대로 다시 업로드하시면 됩니다.
build 할 때 마다 CSS, JS 파일 명이 dist 폴더 내에 무작위로 다시 생성됩니다.
그래서 새로 배포할 때마다 사이트 방문자들은 새로운 CSS, JS 파일을 이용할 수 있습니다. 


---
ts-node : 컴파일할 필요 없이 바로.
원래는 이게 정석
"compile": "tsc && node ./build/src/app.js",
"start": "nodemon -e ts --exec \"npm run compile\""

// "start": "nodemon -e ts --watch 'src/**/*.ts' --exec 'ts-node' src/app.ts", 


---
## 오라클

코어1개짜리 서버 4대, 고정IP 하나씩 각각 할당하고
80, 443 포트 열기.

유료계정은 꼭 OTP 사용하시고, 웹서버는 캐시서버를 거치도록 해두세요!
전 cloudflare 캐시를 이용하도록 해둬서 정적 파일 전송에 따른 네트워크 사용량을 많이 감소시켰습니다.

1달러만 되어도 결제알람 오도록 설정!

1. ARM 서버 만들기
- ssh 키
2. 고정IP 만들기
3. SSH로 연결하기
4. 포트포워딩, 포트열기


S1 = ARM 2CPU 12G 50G - MariaDB 전용 (DB분리, S1 = S2, S3)
S2 = ARM 2CPU 12G 50G - NGINX 웹서버 (그누,영카트,기타)
S3 = AMD 1CPU 1G 50G (항상무료) - NGINX 웹서버 (그누,영카트,기타)
S4 = AMD 1CPU 1G 50G (항상무료) - 대용량 파일전용 (htpasswd 사용, 대용량 파일  다운로드에만 사용)

클라우드 플레어 무료티어는 프록시 켜면 정말 느리지더군요
근데 희안한게 위에 처럼 DB를 분리하니 프록시 켠 상태 인데도 엄청 빨라 집니다.

200GB 한도에서 볼륨 2개 까지
