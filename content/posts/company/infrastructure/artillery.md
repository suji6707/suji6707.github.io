+++
title = 'Artillery 부하테스트'
date = 2024-07-09T00:20:22+09:00
draft = true
+++

## local.yaml 
target 서버.

flow 안에 loop.
`-`는 객체의 시작을 나타내는데 
loop 안에 3개의 `-` 메서드가 있어서 
그 배열을 순회하며 반복요청하는 플로우임.

봐야할 것은 method, url, json.
네이버에서 오는 웹훅은
사용자의 구독 시작/종료 등 이벤트가 발생했을 때 
그 정보를 우리에게 알려주는 역할.

초당 30회까지 테스트 해봐야한다고 했음.

json이 우리가 받을 데이터이고
누가(accountUid) 어떤 서비스를(plan) 어떻게 변경했는지(changeType).
우리 DB에 반영할 이벤트 타입이 changeType이라고 보면 됨.

지금은 START, END, UPGRADE만 지원하고 있고,
추후 DOWNGRADE 등 넣는다고 함.