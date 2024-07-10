+++
title = '스터디 3회차'
date = 2024-05-07T00:20:22+09:00
draft = true
+++
## 7.1 NAT와 DNS

---
## 관련 실습
네, Docker를 사용하여 배스천 서버와 실제 서버를 설정하고, 배스천 서버 포트만 외부에 노출하는 방법을 설명드리겠습니다. 이를 위해 Docker Compose를 사용하여 두 개의 컨테이너를 설정하고, 배스천 서버 컨테이너의 포트만 외부에 노출하도록 구성할 수 있습니다.

### 1. Dockerfile 작성

#### 배스천 서버용 Dockerfile
배스천 서버를 위한 Dockerfile을 작성합니다. 이 예제에서는 Ubuntu 이미지를 기반으로 OpenSSH 서버를 설치합니다.

**Dockerfile.bastion**
```Dockerfile
FROM ubuntu:20.04

RUN apt-get update && apt-get install -y openssh-server && \
    mkdir /var/run/sshd

# SSH 설정 파일 복사
COPY sshd_config.bastion /etc/ssh/sshd_config

# 사용자 추가
RUN useradd -ms /bin/bash bastion_user && \
    echo "bastion_user:password" | chpasswd

# SSH 포트 노출
EXPOSE 2222

CMD ["/usr/sbin/sshd", "-D", "-f", "/etc/ssh/sshd_config"]
```

**sshd_config.bastion**
```plaintext
Port 2222
PermitRootLogin no
AllowUsers bastion_user
```

#### 실제 서버용 Dockerfile
실제 서버를 위한 Dockerfile을 작성합니다.

**Dockerfile.actual**
```Dockerfile
FROM ubuntu:20.04

RUN apt-get update && apt-get install -y openssh-server && \
    mkdir /var/run/sshd

# SSH 설정 파일 복사
COPY sshd_config.actual /etc/ssh/sshd_config

# 사용자 추가
RUN useradd -ms /bin/bash actual_user && \
    echo "actual_user:password" | chpasswd

# SSH 포트 노출
EXPOSE 22

CMD ["/usr/sbin/sshd", "-D", "-f", "/etc/ssh/sshd_config"]
```

**sshd_config.actual**
```plaintext
Port 22
PermitRootLogin no
AllowUsers actual_user
```

### 2. Docker Compose 파일 작성
Docker Compose를 사용하여 두 개의 컨테이너를 설정합니다. 배스천 서버 컨테이너의 포트만 외부에 노출하고, 실제 서버 컨테이너는 내부 네트워크에서만 접근 가능하도록 설정합니다.

**docker-compose.yml**
```yaml
version: '3.8'

services:
  bastion:
    build:
      context: .
      dockerfile: Dockerfile.bastion
    ports:
      - "2222:2222"  # 배스천 서버 포트만 외부에 노출
    networks:
      - internal

  actual:
    build:
      context: .
      dockerfile: Dockerfile.actual
    networks:
      - internal

networks:
  internal:
    driver: bridge
```

### 3. Docker Compose로 컨테이너 실행
Docker Compose를 사용하여 컨테이너를 빌드하고 실행합니다.

```bash
docker-compose up --build -d
```

### 4. SSH 접속
- **배스천 서버 접속**: 외부에서 배스천 서버에 SSH로 접속합니다.
  ```bash
  ssh -p 2222 bastion_user@<외부 IP 주소>
  ```

- **실제 서버 접속**: 배스천 서버에 접속한 후, 실제 서버로 SSH 접속합니다.
  ```bash
  ssh actual_user@actual
  ```

### 요약
- **Dockerfile 작성**: 배스천 서버와 실제 서버를 위한 Dockerfile을 작성하여 각각의 SSH 설정을 정의합니다.
- **Docker Compose 파일 작성**: Docker Compose를 사용하여 두 개의 컨테이너를 설정하고, 배스천 서버 포트만 외부에 노출합니다.
- **컨테이너 실행**: Docker Compose를 사용하여 컨테이너를 빌드하고 실행합니다.
- **SSH 접속**: 외부에서 배스천 서버에 SSH로 접속한 후, 배스천 서버를 통해 실제 서버에 접속합니다.

이러한 방법을 통해 Docker를 사용하여 배스천 서버와 실제 서버를 설정하고, 배스천 서버 포트만 외부에 노출하여 보안을 강화할 수 있습니다.


---
### NAT/PAT
발표 초안
- 인바운드/아웃바운드 IP. 서비스 특성에 따른. (그림 첨부)
보안상 AWS 실서버는 프라이빗 IP만 가지고 있는게 좋으므로.
- NAT를 언제 어떻게 쓰는가?

#### 용도와 필요성
- 주소 고갈문제
- 보안 - 외부에 사내 IP를 숨길 수.
- 주소체계가 같은 두 네트워크간 통신 가능
	- 사설IP는 서로 다른 회사에서 중복해 사용가능.

#### 동작 방식

#### SNAT와 DNAT
SNAT (출발지 NAT)
- 사설->공인으로 통신할 때 많이 사용.
	- 공인 IP dest에서 출발지로 응답을 받으려면 src IP 주소가 필요
	- 공인 IP로 서비스 요청시 출발지에서는 사설 IP를 공인 IP로 NAT.
- 보안
	- 내부 실제 IP 주소를 숨기기 위해.
- 로드밸런서
	- 출발지와 목적기 서버가 동일 대역일 때는 트래픽이 로드밸런서를 거치지 않을수도 있기에, SNAT를 통해 로드밸런서를 거치도록 강제할 수 있음.

DNAT
- 로드밸런서
	- 사용자는 로드밸런서의 virtual IP로 서비스를 요청하고
	로드밸런서는 서비스 virtual IP를 서버의 실제 IP로 DNAT

#### 동적 NAT, 정적 NAT
- 동적 NAT: NAT를 수행할 때 IP를 동적으로 변경
	- 다수의 IP 풀에서 정해지므로 최소한 출발지나 목적지 중 한 곳이 IP pool 또는 range로 설정되어있음.
	- NAT가 필요할 때 IP 풀에서 어떤 IP로 매핑될 것인지 판단해 NAT를 수행하는 시점에서 NAT 테이블을 만들어서 관리.
- 정적 NAT: 출발지와 목적지 IP를 미리 매핑해 고정한 NAT
	- 서비스 방향을 고려하지 않고 NAT를 설정할 수 있음.

보통 정적 NAT를 설정하면 src, dest 방향성 없이 1:1로 매핑되어 통신이 어느 방향에서 시작되더라도 같은 IP로 변환됨.
- 외부->내부로 통신을 시작하면 DNAT (내부가 사설 IP니까)
- 내부->외부로 통신을 시작하면 SNAT가 적용됨.

---
## 7.2 DNS
몇가지 발표 안건들
- 클라우드플레어 네임서버 쓰기 or AWS Route 53
	- 클플/Route 53에서 발급된 ns를 도메인 구매업체 사이트에서 네임서버에 등록시켜 준다.
	- proxied records: 도메인은 네임칩인데 네임서버는 클플
		- 도메인에 ssh 접속하려면 왜 DNS only 서브도메인을 만들어야하는가?
- nslookup 은 name server 관련한 조회를 할 수 있는 명령어


하나의 서버에서 배스천 서버와 실제 서버를 운영하는 경우, 보안을 강화하기 위해 다음과 같은 추가 조치를 취할 수 있습니다.
 방화벽 설정
 •배스천 서버 포트만 외부에 노출: 배스천 서버 포트(예: 2222)만 외부에서 접근 가능하도록 방화벽 설정을 합니다.
 
 •실제 서버 포트는 내부에서만 접근 가능: 실제 서버 포트(예: 22)는 내부 네트워크에서만 접근 가능하도록 설정합니다.

 예를 들어, iptables를 사용한 설정:
```
외부에서 배스천 서버 포트 접근 허용
sudo iptables -A INPUT -p tcp --dport 2222 -j ACCEPT

외부에서 실제 서버 포트 접근 차단
sudo iptables -A INPUT -p tcp --dport 22 -j DROP

내부 네트워크에서 실제 서버 포트 접근 허용
sudo iptables -A INPUT -s 192.168.1.0/24 -p tcp --dport 22 -j ACCEPT
```
--
- AWS에서의 DNS 활용: ECR
서비스들간 상호작용.
	- Elastic Load Balancing (ELB):
 •DNS 이름 제공: ELB는 로드 밸런서에 대한 DNS 이름을 제공하여 트래픽을 자동으로 여러 인스턴스로 분산시킵니다. 예를 들어, my-load-balancer-1234567890.us-west-2.elb.amazonaws.com와 같은 형식의 DNS 이름을 사용합니다.
	- Amazon CloudFront:
 •콘텐츠 배포 네트워크(CDN): CloudFront는 전 세계에 분산된 엣지 로케이션을 통해 콘텐츠를 빠르게 배포합니다. CloudFront 배포에 대한 DNS 이름을 통해 사용자 요청을 가장 가까운 엣지 로케이션으로 라우팅
	- Amazon S3:
 •정적 웹사이트 호스팅: S3 버킷을 정적 웹사이트로 호스팅할 때, 버킷에 대한 DNS 이름을 사용하여 웹사이트에 접근할 수 있습니다. 예를 들어, my-bucket.s3-website-us-west-2.amazonaws.com와 같은 형식의 DNS 이름
	- Amazon ECR (Elastic Container Registry):
 •컨테이너 이미지 관리: 앞서 언급한 것처럼, ECR은 컨테이너 이미지를 저장하고 관리할 수 있는 레지스트리를 제공하며, 이를 위한 DNS 이름을 사용합니다.
 	- Amazon RDS (Relational Database Service):
 •데이터베이스 인스턴스 접근: RDS 인스턴스는 고유한 DNS 이름을 가지며, 이를 통해 데이터베이스에 접근할 수 있습니다. 예를 들어, mydbinstance.123456789012.us-west-2.rds.amazonaws.com와 같은 형식의 DNS 이름을 사용

DNS는 TCP/IP 프로토콜 체계를 유지하기 위한 컨트롤 프로토콜 중 하나.
서버 IP 변경에 대처할 수 있고,
클라우드 기반 MSA 아키텍처에서 서비스간 API 호출 시 필요.

##### MSA에서 DNS 사용
- 리소스를 논리적으로 분리한다는 것은 물리적으로는 동일한 클러스터나 인프라 내에 있지만, 논리적으로 서로 독립된 단위로 관리하고 운영할 수 있도록 분리하는 것을 의미합니다. 이는 주로 네임스페이스(namespace)를 사용하여 이루어짐
- 네임스페이스를 사용하면 서로 다른 환경(예: 개발, 테스트, 프로덕션)이나 팀 간의 리소스를 논리적으로 분리할 때 좋음.

```
BATCH_API_URL=https://batch-api.example.io/api/batch
TRACKING_API_URL=http://tracking-api.prd-example-ns:3000/api/trackings
PAYMENT_API_URL=http://payment-api.prd-example-ns:3000/api/payments
```

#### DNS 구조와 명명 규칙
과거에는 다른 호스트에 접속하여 정보를 이용하기 위해서는 그 호스트 정보(숫자 주소와 이름)를 내 호스트가 알고 있어야 했습니다.
호스트 수가 증가할수록 hosts.txt 파일의 크기가 커짐은 물론, 모든 호스트가 파일을 다운로드 하기 위해 NIC 한 곳으로 접속하게 되어 일종의 병목 현상
무엇보다 가장 중요한 문제는 개별 호스트의 변화된 정보를 신속하게 반영할 수 없다는 점과 특정 호스트에서 다른 호스트와 동일한 호스트 이름을 설정했을 경우 소위, ‘이름 충돌’
-> ‘호스트 정보의 분산 관리’ 및 ‘호스트 정보 검색의 자동화’라는 방향성

관리적 목표의 결정체가 바로, 개별 기관에서 책임을 가지고 관리하는 호스트들을 개별 영역(domain)으로 분류하는 것이었습니다. 예를 들어 가비아(gabia)는 여러 호스트를 관리하는 기관으로 하나의 영역이 됩니다. 그리고 이렇게 구분된 영역들은 그 기관의 특성에 따라 더 큰 영역으로 다시 분류됩니다. 우리에게 너무나 익숙한 .com, .org가 바로 기관의 특성에 따른 영역 분류입니다. 마지막에는 이 모든 영역을 관리하는 ICANN 산하의 IANA가 있습니다. 이들 각 영역은 점(.)으로 구분하여 표시합니다. 이를 문자로 표현하면 다음과 같습니다.
- 호스트 이름(WWW). 흐스트 관리 기관 영역(GABIA). 최상위 영역(COM)
→ WWW.GABIA.COM이 됩니다.
역사적으로 살펴 보면 ‘도메인(domain)’의 탄생은 이처럼 호스트 정보 관리 주체를 분류하는 과정으로 볼 수 있습니다. 

각 관리 기관이 책임지는 영역의 호스트 정보가 담긴 데이터를 흔히 ‘zone 파일’이라고 합니다. 가령 위에서 가비아(관리 기관)가 관리하는 영역(GABIA.COM)의 zone파일에는 www, domain, hosting등의 호스트 이름과 주소가 담긴 정보가 들어있습니다. 그리고 이러한 zone 파일을 보관하고 있다가, 다른 호스트로부터 가비아의 www호스트를 찾는 요청이 있으면 이를 찾아 주는 역할을 하는 것이 바로 ‘Domain Name Server’입니다. 즉 가비아 영역(GABIA.COM) 내에 있는 호스트 정보를 제공하는 서버가 바로 가비아 네임 서버(ns.gabia.com, ns1.gabia.com)입니다.

--

역트리 구조: third.second.top 형태. 맨 뒤의 루트 '.'는 생략.
도메인 계층은 최대 128계층까지 가능하며 계층별 길이는 최대 63 바이트. '.' 구분자 포함 최대 255 바이트.

루트 DNS는 전세계 13개. DNS 서버를 설치하면 루트 DNS IP 주소를 기록한 힌트 파일을 가지고 있어
루트 DNS 관련 정보를 별도로 설정할 필요 없음

DNS 질의 과정
- 사용자가 www.example.com에 접근하려고 할 때, 로컬 DNS 서버는 이 도메인의 IP 주소를 찾기 위해 루트 DNS 서버에 질의합니다.
- 루트 DNS 서버는 .com TLD 서버의 주소를 반환합니다. 
- 로컬 DNS 서버는 .com TLD 서버에 질의하여 example.com 도메인의 권한 있는 DNS 서버 주소를 얻습니다. (네임서버 주소를 얻고)
- 마지막으로, 로컬 DNS 서버는 example.com의 권한 있는 DNS 서버에 질의하여(네임서버에 질의하여) www.example.com의 IP 주소를 얻습니다.
-> 참고로 도메인의 권한 있는 DNS 서버(Authoritative DNS Server)는 도메인 네임 시스템에서 해당 도메인에 대한 최종적인 DNS 레코드를 제공하는 서버입니다. 이 서버는 도메인의 `네임서버`로 설정된 서버를 의미합니다.

💎 모든 것은 미니PC와 버셀로 축약할 수 있지 않을까..?

##### Vercel을 통한 배포

Vercel은 Next.js 개발 팀에서 만든 호스팅 사이트
- 정적 및 동적 콘텐츠 호스팅: 정적 사이트뿐만 아니라, Next.js와 같은 프레임워크를 사용한 동적 웹 애플리케이션도 호스팅

버셀은 도메인 네임 서버와 통합하여 도메인을 Vercel 프로젝트에 연결할 수 있도록 지원합니다. Vercel은 도메인 등록 기관이 제공하는 DNS 설정을 통해 Vercel의 서버로 트래픽을 유도하는 방식으로 작동합니다.
- DNS 설정 지원: Vercel은 도메인 등록 기관의 DNS 설정을 통해 🍊도메인 이름을 Vercel의 서버로 매핑할 수 있도록 지원합니다. 이를 위해 CNAME 레코드나 A 레코드를 설정합니다.
- 콘텐츠 제공: Vercel은 도메인 이름이 Vercel의 서버로 매핑된 후, (버셀 서버에서) 웹 애플리케이션의 콘텐츠를 제공


#### DNS 동작 방식
DNS 서버에 쿼리하기 전 DNS 캐시를 먼저 확인.
- 동적 DNS캐시:DNS 조회시 저장
- 정적 DNS캐시: 로컬 hosts파일

host: 로컬에서 도메인과 IP 주소를 관리하는 파일
/etc/hosts
localhost 뿐만 아니라 local.example.io -> 개발중 도메인에 local을 붙여서 쓰고있음.

도메인 이름 사용의 일관성:
- 개발, 테스트, 스테이징, 프로덕션 환경에서 동일한 도메인 이름 형식을 사용할 수 있습니다. 이는 환경 간의 일관성을 유지하고, 환경별 설정을 단순화하는 데 도움이 됩니다.
- 예를 들어, 개발 환경에서는 local.example.io를 사용하고, 프로덕션 환경에서는 example.io를 사용할 수 있습니다.


#### 마스터와 슬레이브

#### 주요 레코드
- A 레코드: 도메인 이름을 IPv4 주소로 매핑하는 레코드 유형.
	- 동일한 도메인을 여러 IP 주소에 매핑하는 경우: 주로 로드밸런싱이나 Failover 장애조치를 위해 사용
		- ex) www.example.com이라는 도메인에 대해 두 개의 서버를 운영할 경우, 두 서버의 IP 주소는 192.0.2.1, 192.0.2.2. 이를 각각 A레코드1, A레코드2와 매핑함.
		- DNS는 example.com에 대한 요청에 대해 두 IP 주소 중 하나를 반환. 
	- 한 서버에서 여러 웹서비스를 운영하는 경우
		- 여러 도메인을 동일한 IP에 매핑할 수도 있음. HTTP 요청이 오면 헤더의 도메인을 보고 그에 맞는 웹 페이지를 반환 -> 아마 미니PC에서..
- CNAME: 도메인 주소에 대한 별칭

#### DNS 설정
- 버셀 배포 방법 예시?

<TTL>
DNSEver에서는 레코드의 종류에 따라 기본 TTL을 다르게 설정하며, 기본적인 레코드의 경우 10분으로, 그리고 "다이나믹DNS"의 경우 5분(300초)을 기본 TTL로 사용하고 있습니다.

TTL 값이 작을 경우 cache DNS에서 자주 정보를 가져가게 되므로, DNS의 레코드값을 변경했을 때 변경된 내역이 cache DNS에 빠르게 전파된다는 장점이 있습니다.

그러나, TTL 값이 작으면 cache DNS에서 저장하고 있는 정보가 빨리 사라진다는 뜻이므로,
DNS 서버에 장애가 발생했을 때 해당 도메인에 관한 정보가 cache DNS에서 빨리 사라질 가능성이 높아집니다.


---
## 7.3 GSLB (Global Server Load Balancing)
전 세계에 분산된 여러 서버에 트래픽을 분산시키는 기술
- 동일한 DNS 레코드로 서로다른 IP 주소를 동시에 서빙할 때.
응답받을 IP 주소를 로드밸런싱.

- 도메인 위임(DNS Delegation)
	- DNS 서버는 GSLB로 해당 사이트의 DNS를 위임했으므로 GSLB가 실제 NS 서버라고 응답
	- GSLB는 헬스 체크를 통해 해당 IP 서비스가 정상적인지 확인 후 응답.

도메인 내 특정 레코드만 GSLB를 사용할 수도 있음.
1. CNAME
- 외부 CDN을 사용하는 경우
	-  www.example.com을 외부 CDN을 통해 제공하려면, CNAME 레코드를 사용하여 www.example.com을 CDN 제공자의 도메인으로 매핑할 수 있음.
	- www.example.com에 대한 요청이 cdn.example-cdn.com으로 리디렉션되며, CDN의 GSLB를 통해 최적의 서버로 트래픽이 분산됨
2. 도메인 위임(NS)
예시 구성

도메인 등록기관에서의 설정:
 •example.com의 네임서버를 GSLB 제공자의 네임서버로 설정합니다.
```
example.com.      IN NS   ns1.gslb-provider.com.
example.com.      IN NS   ns2.gslb-provider.com.
```

 •GSLB 제공자 네임서버에서의 설정:
 •GSLB 제공자의 네임서버에서 example.com과 그 하위 도메인에 대한 트래픽 분산 정책을 설정합니다.
```
www.example.com.  IN A    192.0.2.1
www.example.com.  IN A    198.51.100.1
api.example.com.  IN A    192.0.2.2
api.example.com.  IN A    198.51.100.2
mail.example.com. IN A    192.0.2.3
mail.example.com. IN A    198.51.100.3
```


---
## 7.4 DHCP
컴퓨터 부팅 과정에서 IP를 어떻게 받아오는가.
- 컴퓨터가 부팅되거나 네트워크에 연결될 때, DHCP 프로토콜을 통해 IP 주소 및 네트워크 구성 정보를 자동으로 할당
- 대부분의 가정용 네트워크에서는 인터넷 서비스 제공업체(ISP)에서 제공한 공유기/라우터가 DHCP 서버 역할을 합니다. 라우터는 네트워크에 연결된 장치(맥북 포함)에게 IP 주소를 할당
- 공유기는 보통 ISP로부터 하나의 퍼블릭 IP 주소를 할당받는데, 사설 IP 주소를 연결된 장치들에 할당. 

DHCP(Dynamic Host Configuration Protocol): IP를 동적으로 할당하는 데 사용되는 프로토콜.
- IP주소, 서브넷마스크, 게이트웨이, DNS 정보를 자동으로 할당. 
(프로토콜은 네트워크 통신을 위한 규칙과 절차)

호스트가 DHCP 서버로 IP를 할당받는 과정
- 클라이언트 / 서버구조는 마찬가지.
	- DHCP 서버에 요청(브로드캐스트)하면 IP pool 중 할당할 IP를 선택해서 보내줌.
	- DHCP 클라이언트는 제안받은 IP를 사용하기 위해 Request 메시지를 전송(브로드캐스트). offer 여러개를 동시에 받고 그 중 하나에 대해 Request 메시지를 전송하는 것. 

AWS VPC내 EC2 인스턴스에 IP 주소 할당
- 자동 IP 할당: 자동으로 서브넷의 DHCP 서버로부터 private IP 주소를 할당받음. 
- Elastic IP주소: 퍼블릭 IP를 EC2에 할당할 때도 DHCP가 사용됨.  
	- Elastic IP 주소는 AWS의 퍼블릭 IP 풀에서 동적으로 할당됨.
- DHCP 옵션 세트
	- EC2 인스턴스와 같은 VPC의 리소스가 가상 네트워크를 통해 통신하는 데 사용하는 네트워크 설정 그룹입니다.
	- 각 리전에 기본 DHCP 옵션 세트가 있습니다. 각 VPC는 해당 리전의 기본 DHCP 옵션 세트를 사용
	- DHCP 옵션 세트 구성: 도메인 네임서버. 도메인 이름, IPv6 임대기간 등.
	- VPC에서 인스턴스를 실행하면 1) DHCP 서버와 상호작용, 2) Amazon DNS 서버와 상호작용, 3) VPC 라우터를 통해 네트워크의 다른 장치와 연결됨.

