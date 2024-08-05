+++
categories = ['Company']
series = ['Proxy']
tags = ['Squid-Proxy']
title = "Squid-Proxy"
date = 2024-07-17T22:50:44+09:00
draft = true
+++
## Squid

### Connect to a Private network via proxy server from macOS

---

Homebrew를 통해 설치된 Squid의 설정 파일 경로가 다를 수 있습니다. Homebrew는 일반적으로 `/usr/local/etc/squid` 또는 `/opt/homebrew/etc/squid` 경로를 사용합니다. Squid 설정 파일의 위치를 확인하고 수정하는 방법을 안내해 드리겠습니다.

### Squid 설정 파일 위치 확인

Squid 설정 파일의 경로를 확인하려면, Squid의 기본 설정 파일을 찾아야 합니다. 일반적으로 Homebrew를 통해 설치된 경우는 다음과 같습니다:

```bash
find /opt/homebrew -name "squid.conf"
```

위 명령어를 실행하면 Squid 설정 파일의 정확한 경로를 확인할 수 있습니다. 일반적인 경로는 `/opt/homebrew/etc/squid/squid.conf`입니다.

### Squid 기본 설정 파일 복사

기본 설정 파일이 `/opt/homebrew/Cellar/squid/6.10/etc/squid.conf.default`에 있을 수 있습니다. 이를 복사하여 사용할 수 있습니다:

```bash
cp /opt/homebrew/Cellar/squid/6.10/etc/squid.conf.default /opt/homebrew/etc/squid/squid.conf
```

### Squid 설정 파일 수정

복사한 설정 파일을 편집합니다:

```bash
nano /opt/homebrew/etc/squid/squid.conf
```

### 설정 예시

Squid 설정 파일을 다음과 같이 수정합니다:

```plaintext
# HTTP 포트 설정
http_port 8000

# ACL (Access Control List) 설정
acl to_example_com dstdomain example.com
http_access allow to_example_com

# 원격 IDC 서버로 요청을 전달
cache_peer example.com parent 80 0 no-query originserver name=example_com
never_direct allow to_example_com
cache_peer_access example_com allow to_example_com
cache_peer_access example_com deny all

# 로그 파일 설정
access_log /opt/homebrew/var/logs/squid/access.log squid
```

### Squid 서비스 시작

설정을 완료한 후, Squid 서비스를 시작합니다:

```bash
brew services start squid
```

### Squid 로그 디렉토리 확인 및 생성

Squid 로그 디렉토리가 존재하지 않으면 생성합니다:

```bash
mkdir -p /opt/homebrew/var/logs/squid
```

### 프록시 설정 확인

Squid가 정상적으로 실행되고 있는지 확인합니다:

```bash
squid -k check
```

### 로컬 테스트

로컬에서 프록시 서버를 테스트합니다:

```bash
curl --proxy "http://localhost:8000" "http://example.com"
```

### Squid 로그 확인

Squid 로그 파일을 확인하여 요청이 제대로 처리되고 있는지 확인합니다:

```bash
tail -f /opt/homebrew/var/logs/squid/access.log
```

### 요약

1. **Squid 설정 파일 위치 확인**: `find /opt/homebrew -name "squid.conf"`
2. **기본 설정 파일 복사**: `cp /opt/homebrew/Cellar/squid/6.10/etc/squid.conf.default /opt/homebrew/etc/squid/squid.conf`
3. **Squid 설정 파일 수정**: `nano /opt/homebrew/etc/squid/squid.conf`
   - HTTP 포트를 8000으로 설정
   - 원격 IDC 서버로 요청을 라우팅하는 설정 추가
4. **Squid 서비스 시작**: `brew services start squid`
5. **Squid 로그 디렉토리 확인 및 생성**: `/opt/homebrew/var/logs/squid` 디렉토리 생성
6. **프록시 설정 확인**: `squid -k check`
7. **로컬 테스트**: `curl --proxy "http://localhost:8000" "http://example.com"`
8. **Squid 로그 확인**: `tail -f /opt/homebrew/var/logs/squid/access.log`

이 단계를 따라가면, 로컬에서 Squid 프록시 서버를 설정하고 특정 포트에서 들어오는 요청을 원격 IDC의 서버로 라우팅할 수 있습니다. 추가로 궁금한 점이 있으면 언제든지 말씀해 주세요!

---