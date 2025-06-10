+++
title = 'nginx'
date = 2025-06-10T00:20:22+09:00
draft = true
+++

### certbot 전 필수 설정

최소한 
- 어느 도메인에서 (server_name)
- 어느 포트를 듣고(보통 80 or 443 중 하나)
- 어떤 행동을 할 것인지 정의
	- proxy_pass: 어디로 프록시
	- if (\$host = ~) {
        return 301 https://\$host$request_uri;
    } // 80인데 도메인이 일치하면 443 https로 리다이렉트

아래 설정 해놓고 
symlink를 avail -> enabled로 해둔 후
certbot -d domain 하면 알아서 서버블록이 추가될 거임. (listen 443)

```nginx
server {
    listen 80;
    listen [::]:80;
    server_name ncs-rank-query.itemscout.io;

    location / {
        proxy_pass http://127.0.0.1:3001;
    }
}
```

오늘 발생한 문제 스크립트
- listen 포트 정의하지 않으면 기본 80으로 들음
- 같은 도메인+포트 조합이 두 번 선언되어 있어서 nginx가 "conflicting server name" 경고
- certbot은 nginx 설정을 자동으로 분석해서,
server_name과 listen(443/ssl) 블록이 명확하게 있는 곳에 SSL 설정을 추가하려고 시도하는데 그게 default였나 싶음.?

```nginx
server {
	server_name ncs-proxy.itemscout.io;

	location / {
		proxy_pass http://127.0.0.1:8888;
	}
}
server {
	listen 80;
	listen [::]:80;
	server_name ncs-proxy.itemscout.io;
}
```