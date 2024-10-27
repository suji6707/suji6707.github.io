+++
categories = ['MySQL']
series = ['Database']
title = "MySQL"
date = 2024-09-27T22:50:44+09:00
draft = true
+++
## MySQL config

도커로 구동할 때 발생한 문제. 
`sqlMessage: 'Client does not support authentication protocol requested by server; consider upgrading MySQL client',`

MySQL 8부터 `default_authentication_plugin(기본 인증 플러그인)`이 보안이 강화된 caching_sha2_password로 변경되면서 발생한 문제.

케이스는 두 가지.
- MySQL 클라이언트가 아직 caching_sha2_password 방식을 지원하지 못하는 경우.
- Nodejs의 mysql 모듈이 지원하지 못하는 경우.

후자는 mysql2로 해결되고.
전자는 default_authentication_plugin을 mysql_native_password으로 다운그레이드.

컨테이너 접속해서
`ALTER USER root@% IDENTIFIED WITH mysql_native_password BY 'root';`

`select user, host, plugin from user;` 로 확인.


 사실 mysql2를 사용하면 일반적으로 MySQL 8.0의 새로운 인증 방식인 'caching_sha2_password'를 지원해야 합니다. 그래서 이론적으로는 MySQL의 기본 설정을 변경할 필요가 없어야 하는데..