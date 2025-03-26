+++
title = 'CS162 Setup'
date = 2025-01-05T00:20:22+09:00
draft = true
+++
# CS162 Setup
쉽지 않았음.
생각해볼 게 많음.
로컬내 remote로 통신하기, clangd 설치 등.

🔵 CS162  
Docker 셋업
실제 하드웨어 소프트웨어 버전이라 생각. 

chown -R workspace:workspace /home/workspace
ENTRYPOINT [ "/entrypoint.sh" ]

* 도커로 접속
ssh workspace@127.0.0.1 -p 16222

ssh로 Docker로 실행 중인 워크스페이스에 접근?
Remote SSH 기능을 사용하면, 로컬에서 실행되는 것처럼 컨테이너 내 파일을 편집할 수 있다. 



🟣 clangd
windserf는 에디터임. 그래픽 GUI일뿐
clangd 익스텐션 기저의 clangd 프로그램을 설치하려면
로컬PC가 아닌 도커 컨테이너내 Ubuntu 버전으로 깔아야 한다.

현재 C/C++을 실행 중인 서버는 도커라서.