+++
title = '스터디 1회차 Q&A'
date = 2024-05-07T00:20:22+09:00
draft = true
+++
NAT, DNS Q&A

## 🏀 NAT, DNS Q&A

### NAT
NAT는 사설 -> 공인 만 있는게 아님
어떤 IP를 다른 IP로 바꿔주는 것. 
사설 to 사설, 공인 to 사설도.
⭐️(my) 나중에 보겠지만 AWS 내부는 사설 to 사설이 많음

NAT라는 건 그 내부에선 NAPT:
N:1 (from:to)
👗여러 (사설)IP 주소를 포트로 구분해서 하나의 IP로 변환(PAT)

아파트에서 NAT를 할 때.
포트가 부족해서 NAT가 안될 수 있으니
public IP pool로 만들어서 함.

** NAT는 라우터와 같은 L3 장비에서 사용할 수 있으며,

AWS NAT는 사설 to 사설임.
실제 퍼블릭 IP는 다른데서 받음.

👗사설에서 공인으로 바꾸는건 인터넷 게이트웨이가 함.
공인 IP 사면 그 자체로 들고있고. (elastic IP 구매한거 뜻하는건가?)
ec2 파면 무조건 사설 IP 갖고있고
vpc에는 공인 넣은 적 없는데
(NAT가 그 자체로 공인IP를 들고있는게 아닌거임
-> 프라이빗 아니더라도 그냥 퍼블릭 VPC 안에 EC2 생각??.)


---
NAT 장비는 실제로 단방향임.

주로 outbound이긴 한데
내 PC를 웹서버로 쓰고싶다 - NAT 동작시 inbound 써야함.
공유기에서 직접 매핑을 하면 됨
특정 포트로 받으면 특정 사설IP로 보낸다- 

캠을 포트단위로 다 매핑시켜놓는다던가.

"네트워크는 나가는 방향과 오는 방향을 잘 구분"
👗load balancer는 DNAT만 하고
👗NAT Gateway는 SNAT만 함.

NLB에서 응답받아서 실서버로 갈때는 또 DNAT임. ㅋㅋ
api.*-에 던져줘야하니까.

금융회사-카드사끼리 네트워크 내부 연결할 때 사설 to 사설도 함.

---
### DNS
도메인 서버가 어떻게 동작하는지 보자.

맨처음 DNS 서버로 질의해야 하는데 (가까운 KT) -> 터미널에 nslookup 치고 > server
DNS 서버에는 root 주소가 있어야 함.

서버  - add roles - DNS 체크 next next - DNS Manager가 설치됨.
properties -> root hints 파일이 있음.


'권한없는 응답': 내가 갖고있는 게 아니라 다른 서버에 질의해서 가져온 경우 뜸.

** 로컬에서도 DNS 캐시 저장함.
ipconfig /displaydns -> 모든 캐시를 보여줌.

ping naver.com
서비스 도메인 설정을 바꿨을 때 flush를 해서 바로 들어가야함.

TTL: 자동으로 비워지기도 함.

만약 내 로컬 PC가 아닌 가비아 DNS 서버라면??
👗-> 작업 전에 TTL을 미리 낮춰놓기.
몇시간 전에 낮춰놔야. 모든 DNS 서버에 한시간에 바뀌는 게 아니라 계층적으로 타고타고 가는거라서.
작업 후 다시 TTL 늘려놓기. (TTL 낮으면 부하가 걸려서)
-> 무조건 TTL 시간보다는 전에 TTL을 낮춰놔야 함. 하루전이나.

중간에 바뀌는게 보임. 아달이가 안맞음
8.8.8.8 이 서버 하나로만 되어있지 않으니까
수많은 서버에서 주는데.

CNAME의 가장 대표적인게 www임.
abc.zigi.store <-> www.zigi.store

abc.sigi.store 
www.zigi.store
-> www는 abc를 가리키고 있고, 그 abc에 대한 실제 IP는 10.10.10.10이다.
- Aliaced된 abc.sigi.store 주소만 바꾸면
www. 주소는 자동으로 바뀜. CNAME으로 매핑되어있어서.

동일한 걸 가리켜야 할 때 CNAME으로 관리하는 게 좋음.
클라우드에서는 거의다 CNAME임.
ABL을 붙이는 등 IP가 계속 바뀔 수 있기 때문에 DNS로 하는거임.
길다란 도메인을 CNAME으로 넣는.
💎 길다란 ALB 주소로 요청이 도달하고, 이 ALB 주소(CNAME)로서 DNS 쿼리하게됨. 

---
#### DNS 위임
온프렘 도메인이 있고, 그건 DNS 서버에서 관리할거고.
클라우드를 만들면 온프렘이 저쪽에 아직 남아있을거고.
chatbot.ORG만 ALB로 처리함.

각각 담당자가 다른데...
-> AWS에서 Route53. 회사 도메인(온프렘쪽)은 그대로 쓰면서 chatbot.ORG를 쓰도록.

온프렘에서 chatbot.ORG를 Rout53으로 위임.
앞에 프리픽스 붙인 chatbot.OOO.ORG를 클라우드 서비스에서 관리할 수 있도록.

chatbot.zigi.store - 10.2.2.2 인데
여기가 아니라 내가 다 관리하고싶다 ! => 클라우드에서.
AWS Route53에서
호스팅 영역 생성
- chatbot.zigi.store. (zigi.store는 회사인데 chatbot 붙은것만!)

이제 도메인이 만들어졌고.
여기에 레코드를 넣는다.
- img:2.2.2.2

권한을 넘겨줄 때 Delegation.
chat -> 네임서버에 route53의 DNS 정보 - Name server 도메인을 넣어줌.
👗"`chat.(서브도메인)`에 대한 DNS 쿼리는 내가 아니라 Route53 네임서버가 알려줄 것이다" 라고 알려주는. 

nslookup img.chat.zigi.store 해보면 바로 나옴.

💎 CDN을 쓸 때 이런 원리를 쓰는 경우들 있음.
- image만 S3 static 쓸때??


---
*** 로드밸런서가 사실상 NAT다?
면서 NAT와는 방향이 정반대였네 ㅋㅋㅋㅋ


---
실습해보려 했음. VM 10대..
도메인 10개가 있을필요 X.
도메인 위임을 하면 되니까
- 💎 레코드 찍어보면서 하는거랑 다름! (nslookup + DNS server 설치)

---
💎 ** NAT Gateway에 여러 IP 8개 붙일 수??
보안장비에서 IP 기반 rule 적용을 위해 큰 IP 대역 필요

온프렘에 방화벽을 잘 안열어주는데
NAT 게이트웨이를 만들어서
사설 to 사설
보안장비는 NAT 하나만 등록하면 되는거고.
서비스 포트로 NAT 테이블을 만드는데
6만5천개 중 내가 쓸수있는게 정해져있는데
번에 대한 커넥션이 남아있으면 포트가 꽉차서 더이상 NAT가 안됨. 포트를

NAT G에 8개 IP 주소연결을 지원ㅇ해서 44만개 가능
세션 끊길 때까지가 한 커넥션이라 충분.
1~8번까지 갖고있는데 
천대가 있으면 포트 다 쓰면 2번으로 매핑.
N:M
공인 IP도 pool이고 소스 IP 도 pool

->
Nat Gateway 들어가서 맨밑에 
Elastic IP allocation 란 보면 추가할 수 있음!



💎 Rout53은 DNS 서버라기보다는 GSLB임. = DNS + Load Balancer
로드밸런싱을 할 때 살아있는 애들한테만 하는거라.



