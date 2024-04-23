+++
title = '랭킹추적 리팩토링'
date = 2024-02-25T00:20:22+09:00
draft = true
+++
## 랭킹추적 리팩토링
### 규칙
파서는 어댑터에 둔다. 
어댑터에서 cheerio 파서를 주입한다

실제 스크래핑 하는 부분은 맨 마지막에 작성.
파싱까지 다 되어 왔다는 가정하에 도메인 작성하기.

1) 상품등록
2) 키워드를 등록한다
3) 랭킹을 추적한다
4) 그룹에 넣는다

---
#### 도메인 객체가 만들어지는 흐름

---
### 1. 유스케이스 정리


#### 랭킹추적 화면.
1. 상품등록을 했을 때
- 누가 트래킹하는지 저장한다 (user_trackings)
- 상품 메타데이터를 크롤링한다
    - 스마트스토어 url의 doc 페이지에서 아래 두 가지를 파싱한다
        - 기본 정보(tracking_products: nvmid, mallpid 등)
        - 디테일 정보(tracking_product_info: reviewAmount, delivery fee 등)
- 파싱된 정보로 도메인을 생성한다
2. 사용자가 키워드를 등록하면
- 네이버 쇼핑검색 페이지에서 해당 상품의 순위를 찾아 반환한다 

⁃⁃⁃⁃⁃⁃⁃⁃⁃⁃⁃⁃⁃⁃⁃⁃⁃
#### 코드레벨
유스케이스와 관련 함수 정리 
-> 도메인을 묶는 것이 목적. 대단한 것 없다. 창의성 발휘 X.

#### 1. 추적상품을 등록하는 서비스
A. 사용자가 url을 등록하면 
- 유효한 url인지 확인하고 파라미터를 제거 후 반환한다
- 해당 url로 tracking_product가 존재하는지 확인
- 있으면 반환, 없을때만 크롤링을 진행한다
    - 강제 업데이트 여부 체크: force = true면 덮어쓰기, 기본 false면 아래 프로세스 진행
- 크롤링 후 파싱하여 객체를 반환한다. 
    - a. 기본 정보 -> finalURL(리다이렉트 최종 엔드포인트)을 포함한 상품 정보들
    - b. 디테일 정보 -> 🍊 파싱로직은 getProductDetailInfoViaWeb 참조
- 파싱한 정보로 해야 할 것:
    - finalURL로 tracking_product가 존재하는지 한번 더 확인
    - 존재하면 (기존에는 conversion map에 추가했는데) tracking_product의 url을 업데이트한다
    - 존재하지 않으면 새로 도메인 객체를 생성하여 DB에 저장한다. DB 테이블은:
        - a. tracking_products
        - b. tracking_product_info
 
 ---
#### 2. 키워드 랭킹을 추적하는 서비스
키워드를 네이버 쇼핑검색 했을때 해당 상품 url이 몇번째에 있는지 체크한다.
##### 흐름
- ✓ addable: 해당 유저가 키워드 등록 가능한지 체크한다 (멤버십)
- ✓ 누가 어떤 키워드를 추적하는지 저장한다(keywords, user_tracking_keyword)
- 실시간 랭킹 추적
    - 네이버 쇼핑검색 API를 호출해서 해당 키워드 검색결과 상품들의 productIds 배열을 가져온다 -> 아웃포트 & 파싱(어댑터)
    - ✓ 유저의 추적상품이 몇번째인지 순위를 매겨서 반환한다


Q. 어떤 tpList에 대해 수행하는가? (실시간에서)
Q. 

##### 서버 파일
keyword_shopping_page.js
- crawlRankViaAPI:
    - 1. crawlTrackingProductsWithNvmid
        - 충분히 도메인 로직으로 내려올 수 있음.
        - 그 역할은:
            - Q. `nvCatalogId` 하나를 위해 `getProductDetailInfo`- 스마트스토어 크롤링까지 한다.
            묶음상품은 - 오픈API든 크롤링이든 묶음상품에 속하면 그 순위를 알기위해 필수다.
            ```
            result.push({
                id: tp.id,
                nvmid: tp.nvmid,
                url: tp.url,
                nvCatalogId: lastStat?.catalog_id,
            });
            ```
    - 2. requestAndCrawlShopPageViaAPI
    - 3. checkNvmids
        - keyword_shopping_detail_page.js 참조.
        - checkNvmids만 우선 구현하면 됨
        - checkMallPids: 배치에 쓰이는 crawlRank에서만 씀.

🍋 크롤링 및 파싱: 
- requestAndCrawlShopPage: 비공식 API -> Q. 지홍님이랑 나랑 각각 구현해야하나??
- requestAndCrawlShopPageViaAPI: 오픈 API(네이버 개발자센터)

일단 실제 스크래핑하는 부분은 제외하고 보라고 하셨다.
그걸 제외하면 무슨 도메인 로직을 보라는 말인가? 파싱도 빼면.




##### 컨트롤러
🍎 실제 랭킹 추적하는 API
@api {post} /v2/tracking/keyword 
-> body에 tracking_id, keywords: [] 추적키워드 배열
https://local.itemscout.io:3001/api/v2/tracking/keyword

🍎 랭킹데이터 조회
1-1. 🍑 랭킹: v2/tracking/product/166/keywords/ranks?
startDate=2024-04-16&endDate=2024-04-17&keywords=35207%2C35514
- rank, adRank, 
"rank": {
    "166": [
        {
            "rank": 101,
            "tie": 0,
            "tieRank": 0

1-2. v2/tracking/product/166/keywords/ranks?
startDate=2024-04-23&endDate=2024-04-23&keywords=389713
- "data":{"389713":[{"rank":101,"date":"2024-04-23"}]}}

2. 그래프: v2/tracking/product/graph/166?startDate=2024-04-16&endDate=2024-04-23&isExample=fal

3. 일반: v2/tracking/product/general/166?isExample=false
- ProductDetailInfo


4. /v2/tracking/keyword/stat/198
검색수, 상품수

5. v2/tracking/product/daily/198?startDate=2024-04-23&endDate=2024-04-23
- tags, price, cate_id, overseas, 잡다.

- addable
    - permission 체크. -> user.checkLimit이 랭킹추적 서비스 
- POST tracking/keyword/:trackingId


---
#### 3. 시계열로 데이터를 저장하는 서비스
server에선 crawler/procs/product_tracking.js
crawlProduct 함수. 

이 두 가지. 
- getProductDetailInfoViaWeb 🍇 : SSR doc
- getProductDetailInfo 🍇 : 비공식 API



---
### 2. 도메인 설계
#### 🥝 랭킹추적 유스케이스와 랭킹키워드상품 도메인.




---
### 3. 코드작성


---
### 데이터 중심 애플리케이션의 OOP란
블룸버그와 같이 다양한 데이터를 가지고 서비스를 하는 경우 (각 테이블들은 relation을 맺고있고, 시계열 데이터와 연결되어있음) 어떤식으로 OOP 설계를 하면 좋을까?

1. 처음에 종목을 등록하면
그 종목에 대한 데이터를 여러곳에서 불러온 후 파싱해서 
해당 종목에 대한 데이터를 DB에 저장해
2. 그 종목에 대한 테마 태그를 등록하면
테마 종목 수익률에서 해당 종목이 몇번째인지 추적해서 기록해.
3. 시계열로 매일 사용자들이 등록한 종목과 관련 테마 태그에 대해 
테마 수익률에서의 종목 순위를 기록해
4. 그 종목에 대한 데이터를 매일 불러와서 시계열로 데이터를 저장해

⁃⁃⁃⁃⁃⁃⁃⁃⁃
Asset 클래스
- 각 자산/종목을 객체로 표현한다
- 속성: 종목 티커, 관련 데이터(가격, 거래량 등), 테마 태그 등
- 메소드: 자산 데이터 로드, 자산 데이터 업데이트, 자산 테마 태그 관리

DataManager 클래스
- 데이터 소스로부터 자산 데이터를 가져오고 파싱하는 역할
- 메소드: 외부 API 호출, 데이터 파싱, DB 저장/업데이트

테마 클래스 (내 경우 키워드)
- 속성: 테마ID, 이름, 포함된 자산 목록은 별도 테이블

RankingManager 클래스
- 매일 랭킹을 업데이트한다
- 메소드: 일별 순위 계산
    - ranking_keyword_products
        - 랭킹에 대한 정보는 RankingManager만이 조작할 수 있다.
        - 키워드 등록의 주체는 사용자지만, 랭킹을 저장하는건 매니저의 역할이다.

UserInterface 클래스
- 사용자 입력을 받고 결과를 출력한다
- 메소드: asset 등록, 테마 태그 등록, 결과 조회


---
가상의 매니저가 핵심적인 행동을 수행하는 구조는 괜찮은가?
아이템스카우트에는 Theater와 같은 존재가 없다.
서비스 차원의 가상의 매니저가 존재할 뿐.
tracking product는 그 자체로 행동을 할 수 있는 주체라 생각하고싶지만
상상이 안간다.
물론 `bag`가 스스로 자신의 현금양을 보고 티켓을 넣는 것까지 할 수 있지만.
정말로 `tracking product`가 데이터를 가져오는 메서드를 갖고있어야 하는지는 의문.


```
const trackingProduct = dataManager
                .createTrackingProductByScraping(user, url);
```

⁃⁃⁃⁃⁃⁃⁃⁃
🍋 취지
새롭게. 오브젝트 관점에서.
가상의 DataManager, RankinManager를 두고
이 오브젝트가 외부 요청을 하거나 랭킹 추적을 하도록.

유스케이스를 훨씬 세세하게 파악해야한다.
불리는 모든 컨트롤러(네트워크탭),
**모든 함수들까지.
그런다음 더 세세한 범위로 유스케이스를 묶으라는 말이다**

⁃⁃⁃⁃⁃⁃⁃⁃⁃
잠깐.
유저가 하는 행동 - 상품등록, 키워드 등록, 결과 조회 
조차도 어떤 인터페이스 클래스라는 말인가?
tracking-api에서는 어떤 클래스에서 이런 메소드들을 달아놨을까?

사실 Service Class고..
따라서 유스케이스는
- 상품등록 : RegisterTrackingProduct
- 키워드 등록: RegisterKeyword
    - 여기서 랭킹 추적도 들어가는데
    - 수행 주체가 RankingManager 클래스다. 
- 결과 조회: ViewData

#### UseCase @@@
1. ProductRegister UseCase 
- DataManager Class의 

2. KeywordRanking UseCase

3. TrackingReport UseCase


4. Batch - Cluster (이건 나중에)

⁃⁃⁃⁃⁃⁃⁃⁃⁃
그렇다. 소프트웨어의 동작은 사실 어떤 가게의 판매원이 하는 행동과도 같다.
이 판매원의 행동은, 
- 사용자의 멤버십 등급을 보고 상품을 등록해도 된다 안된다를 판단하고,
- 상품을 등록할 경우 
    - DataManager에게 상품 데이터를 가져올 것을 요청하고 (데이터 등록 권한 위임)
    - 이 DataManager는 DB 기록까지 완료한다.
- 키워드를 등록할 경우


---
#### 발상의 전환
사실 그런 생각도 해본다.
모든 알고리즘 문제나 일하면서 마주치는 OOP 설계들을-
블룸버그와 금융시장의 다이내믹한 생성과 종료에 빗대어 생각해보면
모든 것이 다르게 보인다는 것을..


---
detailInfo를 저장하는 다른 부분.
crawlProduct
- 배치: 
crawler/bin/product_tracking.js, 
utils/batch/product_tracking.js
- 

잠깐.
유저 트래킹 정보 추가는 
유저가 직접 하는 것이 아닌가?
적어도 등록하는 행위 자체는 
User.buy처럼
User.register 라던가..


---
#### 정규화가 잘 되어있으면 relation이 필요없다

 * @@@ 엔터티 정의시 OneToMany 관계 넣을 것인가? 굳이? 
 * 정규화가 잘 되어있어서. tracking_product의 url이 변경되더라도 매핑테이블은 id 프라이머리키만 참조하고 있으므로.
 * select 해오면 그만.


 ---
#### 🍇 코드를 분석할 때 가장 중요한 것
웹을 면밀히 살펴보는 것.

Flow
한번의 호출에서 getProductInfo, getProdudctDetailInfo . 
전자는 웹페이지 doc 자체, 후자는 비공식 API. 

두 군데에서 크롤링을 해서 각각의 DB 테이블에 저장을 하고 있었고.
정말로 하나로 가져올 수 없나? 라는 생각.

getProductDetailInfo에 보니 `reviewAmount`라는 데이터가 필요해서 
추가 크롤링 요청을 하는 것 같고.
그런데 reviewAmount를 필터에 검색해보니(Fetch 말고 전체)
Doc에 있는 것을 발견.
그럼 사실 웹페이지 크롤링 한번으로 둘다 구할 수 있는거였음.

이때 궁금해지는 건
💎 네이버는 이 스마트스토어 웹페이지를 만들때 클라이언트에 데이터를 어떤식으로 전달하는걸까?
doc에 보면 `window.__PRELOADED_STATE__` 필드에 핵심 정보들이 있는데, 서버가 필요한 정보를 일단 다 주는구나.
이건 **서버사이드 렌더링** 방식으로 하고 있다는 사실. 
**최신 웹 프레임워크- Next 트렌드**를 알면 바로 떠오를 만한 것들.

💎 getProductDetailInfo 에서 비공식 API URL 주소를 만들때,
이런 API가 불린다는 사실을 어떻게 알았을까?
이건 써봐야 함.
페이지 상단 리스트를 이것저것 클릭해보면
필요한 부분만 새로 렌더링한다는 것을 알 수 있고.
이때 Fetch가 네트워크 탭에 찍힘.
이 API를 쓰는거다.

그럼 스마트스토어 크롤링은
1. Web
2. closed API

둘 다 쓰는 이유가 뭘까?
일단 1.Doc 크롤링은 무조건 필요함
2를 쓰려면 channelUid를 알아야해서.

2는 원래 시계열 데이터 수집을 위해 쓰고있음.
매번 Doc 불러와서 파싱하기보다
필요한 데이터만 불러오는 API를 직접 호출해 한번에 배치 수집하는거겠지. 

그럼 구비해둬야 할 스마트스토어 크롤링 요청은 두 개.
1. 스마트스토어 URL doc
- 여기서 파싱이 두 가지임.
    - 1) nvmid 등 기본 정보
    - 2) reviewAmount 등 디테일 정보 -> 이게 getDetailInfo
2. 비공식 API
- `https://smartstore.naver.com/i/v2/channels/${channelUid}/products/${mallPid}?withWindow=false`

