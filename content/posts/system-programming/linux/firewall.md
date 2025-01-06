+++
title = 'Ubuntu 설정'
date = 2024-12-08T00:20:22+09:00
draft = true
+++
## iptables 지우기

💎 ufw로 평가하고 싶으면 iptables REJECT 라인 지워줘야함.
ufw 설정들이 하나도 안먹고 있었음.

sudo iptables -nL
sudo iptables -I INPUT -p tcp -m tcp --dport 8080 -j ACCEPT

- `-I INPUT`: INPUT 체인의 시작에 규칙 추가
- `-p tcp`: TCP 프로토콜
- `--dport 8080`: 목적지 포트 8080
- `-j ACCEPT`: 해당 트래픽 허용


위에서부터 평가.
기본적으로 인바운드 deny 하되 특정 포트는 조건부로 허용하도록 하려고 하면
INPUT을 해야함. APPEND로 하면 reject에서 걸려버림.. 

```
Chain INPUT (policy DROP)
target     prot opt source               destination
ACCEPT     6    --  0.0.0.0/0            0.0.0.0/0            tcp dpt:8080
ACCEPT     0    --  0.0.0.0/0            0.0.0.0/0            state RELATED,ESTABLISHED
ACCEPT     1    --  0.0.0.0/0            0.0.0.0/0
ACCEPT     0    --  0.0.0.0/0            0.0.0.0/0
ACCEPT     17   --  0.0.0.0/0            0.0.0.0/0            udp spt:123
ACCEPT     6    --  0.0.0.0/0            0.0.0.0/0            state NEW tcp dpt:22
REJECT     0    --  0.0.0.0/0            0.0.0.0/0            reject-with icmp-host-prohibited
```

---
