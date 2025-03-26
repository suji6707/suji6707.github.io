+++
title = 'PubSub'
date = 2024-10-26T00:20:22+09:00
draft = true
+++

### 큐 실서비스 활용 예시

🍀 큐 두개.
하나: 스케줄링

둘: 완료 구분.
if 
🍀 stalled: 작업이 진행 중인데 워커가 응답하지 않는 경우
Failed랑 다름!. 
Rate limit으로 fail 한거면 큐에 다시 넣고- 


🍀 attempts: number; // The total number of attempts to try the job until it completes.

---

EC2 인스턴스 하나만 띄우고(1cpu)
pm2 instance:4로 켜면 프로세스 4개가 parallel 하게 돌아감.

별도 실행파일을 돌리는거고
이는 잡큐를 따로 관리함을 의미.

subscriber는 컨트롤러 포트인 + 유스케이스.

서버 프로젝트 ----->  subscriber
    포트아웃 addJob  포트인


---
### 문제상황

MaxListenersExceededWarning: Possible EventEmitter memory leak detected. 11 error listeners added to [Redis]. Use emitter.setMaxListeners() to increase limit


기존 경우엔 complete 되면 그때 다음 Job을 받게 되어있는데
지금은 Job 받고 1초면 계속 다음 Job을 받음. -> active가 10개 이상이 되면 문제.

Bull은 각 job의 상태를 추적하기 위해 Redis 연결에 이벤트 리스너를 추가
근본적인 원인은 Bull이 각 job마다 새로운 Redis 이벤트 리스너를 생성하기 때문
active 상태의 job이 많아질수록 이벤트 리스너가 누적되는
Node.js의 기본 최대 리스너 수는 10개

**
rate limiter 쓸 경우:
- bounceBack: true: 정상적으로 5개씩 들어옴. - 6개째부터는 retried
- false: 5개가 전부 완료되고나서 5개가 들어옴.  - not retried, 6개부터는 fail이고 다음 턴에 


** 
🔴 메모리누수: 다시 발생안하긴하는데..
사실 애초에 rate limit max 가 무한대로 설정 가능한거면 . add job마다 문제될리가??

job id는 내부 누적 id임.

---
### 🔴해결

클라에서 S3로 바로 업로드, 
1688 API 요청할 때는 url만 서버에서 가져와서 쓰도록 .

jobId만 던져주고 계속 조회하게 하는게 핵심.
이전 문제는 job.finished()에서 이벤트 핸들러를 달아서 
계속 지켜보게 하는 형식이었는데 (busy waiting같기도 하고. wait 1000 동안 계속-)
MaxEventListener 에러..
이젠 클라에서 직접 .

업로드하는 url만 별도로