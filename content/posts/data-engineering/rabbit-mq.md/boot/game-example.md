+++
title = 'RabbitMQ'
date = 2024-10-26T00:20:22+09:00
draft = true
+++


실시간 이벤트를 분산된 시스템에서 듣고 반응해야 하는경우 유용.
유저는 []piece라는 army를 갖는다.
다른 플레이어 위치로 이동하면 배틀.
move하면 메시지를 publish: 'move가 발생했다'

publishCh 채널로부터 메시지를 기다림.
publishCh <-chan move
채널이 move 타입 데이터를 수신할 수 있는 read-only.

move: 누가 어떤 piece를 움직였나.

publisher = producer = sender
sub = 

distribution 로직으로부터 subscribe 로직을 분리.

* publish
u.march: publish 채널에 move를 보냄.
누가, 어떤 piece를

* subscribe
subCh 채널에서 모든 move를 읽어 
user의 모든 pieces loc에 접근한 것들을 배틀에 넣음.

*distribute
publish 채널의 모든 move를 
모든 sub 채널에 보낸다. 

* 이벤트 드리븐 디자인
디커플링된 시스템 사이에서 이벤트를 트리거.

---
# RabbitMQ
- rabbitMQ 서버: 메시지 브로커 그 자체. 메시지 send, receive에 사용
- 클라이언트 라이브러리를 통해 MQ 서버와 상호작용
- 관리 UI: 서버 모니터링

complex routing, filtering, and delivery requirements 가능. 

5672: 기본 AMQP 프로토콜 포트. 클라이언트 애플리케이션이 이 포트를 통해 메시지를 보내고 받음
15672: 웹 UI 포트
포트 25672 (Clustering): 여러 RabbitMQ 노드를 클러스터로 구성할 때, 노드 간의 통신은 이 포트를 통해 이루어짐.
포트 15692 (HTTP/Prometheus):

curl -u guest:guest http://localhost:15672/api/overview


* Exchange and Queues
exchange는 퍼블리셔가 라우팅 키를 가지고 메시지를 보내는 곳.
라우팅 키를 필터로, 그 키를 듣고 있는(binding) 모든 큐에 메시지를 보냄.
큐는 메시지가 consume될때까지 들고있고
채널은 큐, exchange, 메시지를 만들게 해주는 가상 커넥션.
커넥션: RabbitMQ 서버에 대한 TCP 커넥션.


(참고)
go context 패키지
새로운 고루틴 작업시 일정 시간동안만 지시하거나 외부에서 취소할 때 사용.
timeout, deadline, channel을 통해 실행을 멈추게.

Go encoding/json 패키지
메모리 바이트 변환. 

