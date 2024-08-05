+++
title = '스터디 7회차'
date = 2024-07-30T00:20:22+09:00
draft = true
+++
## AWS 네트워크 총정리
실무위주.

* 라우팅테이블
디폴트 경로가 IG로 가는 애 외에는 전부 프라이빗 서브넷임.
- Public: 목적지는 0.0.0.0, Target이 IGW
- Private: Target이 NAT임. (사설IP->사설IP->공인IP라)

* Private 서브넷에서 밖으로 나가려 할 때-
NAT는 사설->사설로 먼저 바꿈.
그다음 Public 서브넷 RT를 보고 Dst가 IG.

NAT도 가용영역별로 만들어주는 게 좋음.
iG는 리전 범위인데
두 개의 가용영역이 하나의 IG를 바라보고 있는.
NAT는 각각 자기영역 바라보고. 
-> 사설은 RT를 따로 만들어줘야 함.
공인은 RT가 같은 IG라 같지만.

---

* 네트워크 보안
NACL 
조건에 맞았을 때 허용/거부
탑다운 방식이라 
특정한 것들 막아놓고 아래에 특정한 것들 허용하는 방식.

나가는 정책 따로?
Stateless
출발지는 랜덤인데-
목적지는 사용자 포트고
-> 레인지 너무 넓어서 잘 안씀.

보안그룹: 
- 기본적으로 deny
- Stateful이라 응답은 정책이 없어도 통과

---

* VPC간 연결(peering, transit gateway)


* 온프렘과 연결
💎 https로 데이터를 주고받을 때와 뭐가 다르지?

VPN: AWS와 온프렘간 연결.
BGP라는 라우팅을 써야함.

유동IP는 내부적으로 바뀌므로 DNS를 회사도메인 CNAME에 박아넣음.

---

* VPC에서 도메인 질의 
Route53 Resolver => DNS 서버라고도 홤. CIDR의 3번째 주소 연결
사설도메인 쿼리도 가능.

* Route53 
PUblic 호스팅
Private 호스팅: 내부용으로만.

---
* 중요) Endpoint, Private link
💎💎interface Endpoint💎💎
https://white-polarbear.tistory.com/136

AWS에서 퍼블릭 서비스 이용하기
- IGW 통해 인터넷으로 통신해야 하는데 경로를 뚫기는 싫다-
-> 프라이빗하게 연결하는게 Endpoint 방식임.

G.W Endpoint(이건 라우팅 처리해서 -), 
Interface Endpoint


next hop을 GW Endpoint로 넘기고
API Gate 레코드 공인 IP
엔드포인트 만들었을 때 
API GW 조회시 interface Endpoint IP로 바뀜
=> IGW 안쓰고도 S3 등 리소스와 통신 가능. 

피어링이랑 다름. 
내가 관리하는 VPC의 사설 통로가 만들어지는거라..
NLB가 일종의 프록시가 됨.

되게 많이 씀.


💎💎 NLB를 앞에 두고 ALB로 바로 붙여서 쓰는 경우들 실무에서 꽤 있음.
NLB는 IP 고정인데 ALB는 계속 바뀌어서.
- ALB는 스케일아웃 가능, NLB는 엄청 큰 장비임. 
- 웹방화벽 필요할 때 ALB 써야해서 같이쓰기도 함.

NLB와 ALB
L4 - L7
- L4는 하드웨어적으로 IP가 고정되어있고
- L7는 IP가 보존되지 않음. X Forwarded for HTTP 헤더 사용 가능

https://no-easy-dev.tistory.com/entry/AWS-ALB%EC%99%80-NLB-%EC%B0%A8%EC%9D%B4%EC%A0%90
<img width="796" alt="image" src="https://github.com/user-attachments/assets/e3e55691-b9c5-4b9d-ab7e-dc1eff41417f">

우리는 로드밸런서가 com-itemscout-elb 하나만 있다고 보면 되고
(proxy-nlb는 없앴음)
elb의 Target Group은 각각의 인스턴스 또는 ECS 컨테이너가 된다.
<img width="1274" alt="image" src="https://github.com/user-attachments/assets/8486a452-9113-4d47-8234-6e903063a258">

---

Q. 
LB에서 앞단에서 IP를 구분하는 방법 ? 
내부 IP들만 허용하고 싶으면 (자동로그인)
내부 도메인으로 분리하는 경우가 더 흔함.

DNS 단에서 IP 분기처리 가능함.
서버에서도 할수있을거고.

---


#### 로드밸런서 리스너
ban `000.00.00.0/32` -> . 당 1 byte(8bit) 이므로 32면 4개 전부 명시해서 단 하나의 IP만 ban한 것.
한 . 당 0~255의 값을 가질 수 있음.

#### 라우팅테이블
Dest 172.31.0.0/16, Target local의 의미? 
- AWS에서 기본 VPC의 IP 주소 범위는 대개 172.31.0.0/16
- 172.31.0.0/16: 이 CIDR 블록은 172.31.0.0부터 172.31.255.255까지의 IP 주소 범위를 포함합니다. 즉, 최대 65,536개의 IP 주소(호스트 주소)를 포함
	- Dest: 172.31.0.0/16에 속하는 모든 IP 주소 대상
	- Target: local은 이 트래픽이 VPC내 다른 서브넷이나 인스턴스로 라우팅됨을 의미. 즉 VPC 내부 통신을 의미함.

한편 
Dest 0.0.0.0, Target Eni(Elastic Network Interface) 는 IGW 또는 NAT일텐데 이 경우 프라이빗 서브넷이라 NAT에 해당하는 네트워크 인터페이스.
- VPC 외부로 나가는 모든 트래픽을 정의. 
- 0.0.0.0/0 범위 내의 IP 주소로 향하는 모든 트래픽은 지정된 네트워크 인터페이스를 통해 라우팅

전부 private RT인데 
딱 하나 com-pub-in-rtb 만 public RT임.
아니나 다를까 
Dest 0.0.0.0, Target igw-e322fd8b 외부로 향하는.
마찬가지로 VPC 로컬도 있고. (즉 아웃바운드, 인바운드 다 정의됨)
