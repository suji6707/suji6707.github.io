+++
title = 'SSH'
date = 2025-01-01T00:20:22+09:00
draft = true
+++
# SSH remote 접속 vs Git Repo SSH

🟣🟣🟣 왜 아직도 모르니
처음 ssh는 로컬 PC 키로 도커 연결하는게 맞고,
git 연동할 ssh key는 새로 생성.
- .ssh/config는 remote PC 연결
- .ssh/**.pub 및 프라이빗 키들 중 일부가 remote PC 연결에 쓰일거고, 일부는 github 레포지토리 연결에 쓰임. 

* ~/.git/config
ssh 키로 remote 레포와 통신하려면 
remote origin을 HTTPS가 아닌 🔥🔥🔥SSH로 해둬야 함!!

🟣 문제가 생긴 이유를 알았다.
SSH 키 기반 인증 방식.
1. 로컬PC - 도커간 ssh 접속과
2. 도커내 git - remote git repo간 ssh 통신을 혼동했기 때문.

1의 경우,
키 페어: private key는 로컬 PC에만 있어야 함.
즉 `ssh-copy-id -p 16222 -i ~/.ssh/id_ed25519.pub workspace@127.0.0.1`
- ~/.ssh/id_ed25519.pub의 내용이 도커 컨테이너의 ~/.ssh/authorized_keys 파일에 추가됨.
- .pub 파일 자체가 복사되는 것이 아님
- 참고로 authorized_keys 는 여러 개의 퍼블릭키를 가질 수 있음. 여러 디바이스에서 해당 컨테이너로 SSH 접속할 시. 

💎로컬 PC(프라이빗 키) <----> 도커 컨테이너(퍼블릭 키)
- 도커는 퍼블릭 키로 암호화된 데이터를 보내고, 로컬PC는 프라이빗키로 해독함.

2의 경우 반대로,
git remote에는 .pub 키를,
🔥🔥🔥로컬 디바이스에선 private 키를 .ssh/에 들고있어야 한다.

💎로컬 PC or 도커 (프라이빗 키) <----> GitHub (저장된 퍼블릭 키)

---

* 트릭을 써서 .
그냥 git clone이 안된다면-
student0로 넣어보고

git remote add personal git@github.com:Berkeley-CS162/student0
git pull personal main 

ssh-keygen -t ed25519 -C "heojisu6707@gmail.com"
