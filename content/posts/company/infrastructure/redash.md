+++
categories = ['Company']
series = ['Redash']
tags = ['Data']
title = "Redash"
date = 2024-07-25T22:50:44+09:00
draft = true
+++
## 🔵 Redash 
### APM 서버 구조

원래는 Signoz docker-compose에 Nginx 있고
Redash 도커 이미지에도 Nginx가 있었는데
그냥 Nginx 설정 하나로 redash/signoz/grafana 다 서빙하려고 
Redash docker-compose.yaml에 Nginx 다운받는 curl 부분 없앰.

궁극적으로. ncs-rank-apm.itemscout.io 모니터링 데이터도 
https 암호화 통신이 필요하다고 생각했고,
오직 정해진 포트에서만, https 접근을 허용하는 게 원칙이다.

Signoz 백엔드 포트는 3301이고 이것들을 http로 통신하기 때문에
이걸 해커가 바로 접속하는 걸 막기 위해 3001로 https를 열어두고 백단에서는 막음.


💎 중요
접근제한은 
1. 방화벽 설정(sudo ufw) 
2. docker-compose.yaml
로 했음. 도커 IP tables는 차단이 안먹었음.

```yaml
# docker-compose.yaml
127.0.0.1:3301:3001
```
-> 내부에서 쓰는 포트(http)를 3001 포트만 접근을 허용한다. 
 

<img width="814" alt="image" src="https://github.com/user-attachments/assets/ad3cff9b-8d35-421d-9c05-dd8382671279">



---
#### apm 서버 Nginx 설정
```shell
upstream redash_server {
    server 127.0.0.1:5000;
}

upstream grafana_server {
    server 127.0.0.1:3030;
}

upstream signoz_server {
    server 127.0.0.1:3301;
}    


# Redash는 443으로 오면 5000 포트로 라우팅
server {
    server_name ncs-rank-apm.itemscout.io;

    listen [::]:443 ssl ipv6only=on; # managed by Certbot
    listen 443 ssl; # managed by Certbot
    ssl_certificate /etc/letsencrypt/live/ncs-rank-apm.itemscout.io/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/ncs-rank-apm.itemscout.io/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot
        
    location / {
        proxy_set_header X-Forwarded-Host $http_host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_pass       http://redash_server;
        proxy_redirect   off;
    }
}

# 그라파나는 3000으로 받아서 3030 포트로 라우팅
server {
    server_name ncs-rank-apm.itemscout.io;

    listen [::]:3000 ssl ipv6only=on;
    listen 3000 ssl;
    ssl_certificate /etc/letsencrypt/live/ncs-rank-apm.itemscout.io/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/ncs-rank-apm.itemscout.io/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot

    location / {
        proxy_set_header X-Forwarded-Host $http_host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_pass       http://grafana_server;
        proxy_redirect   off;
    }
}

# Signoz는 3001로 오면 3301 포트로 라우팅
server {
    server_name ncs-rank-apm.itemscout.io;

    listen [::]:3001 ssl ipv6only=on;
    listen 3001 ssl;
    ssl_certificate /etc/letsencrypt/live/ncs-rank-apm.itemscout.io/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/ncs-rank-apm.itemscout.io/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot

    location / {
        proxy_set_header X-Forwarded-Host $http_host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_pass       http://signoz_server;
        proxy_redirect   off;
    }
}

server {
    if ($host = ncs-rank-apm.itemscout.io) {
        return 301 https://$host$request_uri;
    } # managed by Certbot

    listen 80;
    listen [::]:80;

    server_name ncs-rank-apm.itemscout.io;
    return 404; # managed by Certbot
}
```


---
(참고)

즉, 외부에서 들어오는 요청이 컨테이너로 전달되기 전에 DNAT를 통해 외부 IP와 포트가 내부 컨테이너의 IP와 포트로 변환됩니다. 따라서 DOCKER-USER 체인에서 iptables 규칙을 작성할 때는 변환된 내부 주소를 기준으로 매칭해야 합니다.

