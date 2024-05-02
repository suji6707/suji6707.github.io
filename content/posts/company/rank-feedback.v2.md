+++
title = '랭킹추적 리팩토링 1-2. 상품 등록'
date = 2024-02-25T00:20:22+09:00
draft = true
+++
## 랭킹추적 리팩토링 1-2. 상품 등록

### 주요 논의사항
### 공통사항
마인드셋.
회사에서 시간을 준 건 엄밀히 '가치창출'의 영역임.
더 나은 기술력을 가질 수 있는 기반을 만들 시간을 준다 - 
그러한 차원에서 자기계발도 되어야 한다.

원오프로 끝이 아니라.
네이버에 가도 견줄만한 실력을 갖추겠다는 마인드셋.
그것이 되어있으면
재밌게 할 수 있을 것이다.

#### 서비스에서 주입, 도메인 로직은 독립적으로
다른 코드가 영향을 받지 않도록 한다는 게 핵심.

도메인 안에서 collaborators = 내가 협력하는 도메인들을 어떻게 주입받을것인가 고민.
단순히 new 하면 안된다.
하나의 도메인이 다른 도메인을 new 하고 있다? ㄴㄴ -> 강하게 결합되므로.
단일책임원칙이 아니게 됨.

A가 B뿐 아니라 C에 대한 책임도 져야하면 
의존성이 여러개가 생기므로 
**도메인에서는 서로가 new를 하면 안된다.**

#### 클라이언트 / 서버 아키텍처
클라는 화면이 아니라 서비스 레이어라고 보면되는데.
main함수에 해당

클라에서 모든 객체를 생성해야하고.
main에서 메시지를 주고받아서 비즈니스를 구현한다.

#### 개방폐쇄 원칙
Product 안에 title, price, url이 존재한다면
이 title이 한국어/외국어가 있으면 
product가 소스코드를 수정하지 않고도 둘 다 허용해야하고
그걸 client 코드(서비스 레이어단)에서 제공해야한다는 것.

서비스에서 어떤걸 주입시키느냐에 따라 코드가 달라지도록. 
en, ko 둘다 해결할 수 있는. 

---
### 키워드분석(지홍님)
```
👠 trend-keyword-analyst.factory.ts 파일 !
>> generateKeywordStats 함수 잘 봐바.
어떻게 abstract class로 할 수 있는지
```
팩토리에는 비즈니스가 들어가면 안됨
정책은 자기자신만 갖고 있어야 함.

keywordId가 일치하는 것들을 넣는 작업이 
monthly, prdCnt, 등 세 개의 도메인에 반복되므로
이걸 각자의 도메인으로 로직을 내리는 게 좋다.

---
### 상품등록(나)

#### 1. 레거시 서버 API로 데이터를 가져오는 방법도 있다
유저 role이 있는데
레거시 서버에서 갖고와도 될듯??

subscription DB를 직접 조회하는게 못마땅하니.
엄연히 도메인이 분리된건데.

MSA를 하면 DB도 다를거라. 
DB 연결 끊었을 때 문제가 되는 상황도 염두에 둔다면
API로 받는것도 좋음!

**service 정보까지 같이 보낼수도 있음!!
직접 조회할필요 없다면
레거시에 구현하면 될듯**
구현체는 나중에 고민해보자.

---
## 💎 팩토리 패턴에만 타입에 따라 다른 도메인 객체를 생성하도록
나머지는 각자 도메인에서 정의한 메서드에서 처리

일단 플랫폼은 네이버로 한정하고
네이버 안에서 product 타입만 구분해서 상속하도록 추상화해본다.
- url 만으로 스토어 타입을 알 수 있고,
생성자에 url을 넣음과 동시에 
SmartstoreProduct의 경우 getter `type`에서 smartstore enum을 던져주도록 함.
쇼핑윈도우는 enum 1
- 이것에 따라 생성자에서 다른 도메인을 생성함

**Product에 apiUrl 속성 추가**

`scrapeNaverStoreViaAPI` 할 때
url을 던져줄게 아니라
IProduct 자체를 던져줘야.
서비스에선 그 타입을 구분하지 않고
파서에서 그 속성을 읽어 서로다른 스크래퍼를 생성해 파싱하도록.

parse factory
instanceof 
- smartstore product면 factory.create -> smartstore scraper를 만들고
브랜드스토어면 brandstore scraper
여기에선 instancecof가 있어도 될거같은데.

type을 프로퍼티로 들고있는게 더 나을듯.
스스/브랜드/윈도우
- 근데 브랜드스토어는 거의 똑같다는데

적절한 스크래퍼를 생성


🍎 server 코드
- 아래 productApiUrl를 팩토리패턴 create 함수 안에서
apiUrl 속성을 넣는 데 저 로직을 써야함.
- reconstitute에도 동일한 로직으로 도메인 생성해야하므로
저것만 함수로 따로 빼두기. 아무데나
```javascript
getProductDetailInfoViaWeb
// 하나가 스스, 다른게 쇼핑윈도우
data = data.product?.A || data.productDetail?.A?.contents;

// getProductDetailInfo는 ApiUrl 자체가 다르고.
async function getProductDetailInfo(productInfo, sleep = 1500) {
    const productApiUrl = isSmartStore ? url.startsWith('https://brand.naver.com/') ? `https://brand.naver.com/n/v2/channels/${channelUid}/products/${mallPid}?withWindow=false` : `https://smartstore.naver.com/i/v2/channels/${channelUid}/products/${mallPid}?withWindow=false`
        : `https://shopping.naver.com/v1/products/${mallPid}`;
```

🍎 선정산 참조
```typescript
export class GlobalScmScrapingAdapter implements IScmScraper {
    constructor(
        private readonly clsService: ClsService,
        @Inject(CRAWLING_HTTP_SERVICE)
        private readonly crawlingHttpService: CrawlingHttpService,
        @Inject(ADDITIONAL_EMAIL_USECASE)
        private readonly additionalEmailService: IAdditionalEmailService,
    ) {}

    private newScrapInstance(scmType: ScmType): DefaultScraper {
        if (scmType === ScmType.COUPANG_WING) {
            return new CoupangWingScraper(
                this.additionalEmailService,
                this.crawlingHttpService,
                this.clsService,
            );
        }

```

---
때로는 순서를 거슬러서 해보자.

오히려 상품리스트 조회 부분에서 
도메인을 분리하고 클래스 상속을 했어야 한다는 걸 깨달을 수도 있다.

## 🍋 정책에 대한 생각

1. 상품리스트 조회시
- 화면에서 마크 표시할 때- 관련 도메인 필요할 것
- 최근꺼랑 가장 앞에꺼라 비교하는 로직이 필요할 것.

2. 경쟁상품 추가시
- 몇개까지 가능?
- 추적 비교상품 추가 API
- 어쨌거나 Tracking Product 도메인에 달려야하는 것 아닌가?
```
https://api.itemscout.io/api/v2/tracking/product/catalog
```
경쟁상품은 어디에 묶을 것인가
tracking product? 
product?

키워드가 상품에 묶이고 
경쟁상품이 상품에 묶여도 되고.

3. 스토어 타입
- 테이블 칼럼은 type 0, 1로 구분되겠지만
소스코드로 녹이는 게 필요할 것.
- 스마트스토어와 브랜드스토어에 가격을 보여주는 부분이 달라진다고 하면.
if else 로 분기처리를 해야할 것.
- 하나의 정책이라도 다르게 가져가는 순간
if else 가 줄줄이 달리게 됨..

- 우리 서비스는 계속 변경되어야하는데. 크롤링이라.
이때 수정을 한다고 하면
그 타입을 알아야 함.

4. Tracking Product 개방폐쇄 원칙
- 너무 폐쇄되어있다. 
- 상품만 추적할 것이냐??
예를들어 상품만 추적하는게 아니라 
해당 쇼핑몰 자체를 추적한다던가
방문자, 매출이라던가.
그러는 순간 새로 파야하게 됨.
- 💎 Tracking으로 둘듯. 상품 외 다른 타입도 확장될 수 있도록.

5. 기타 
비즈니스를 표현하는 코드.
- 판매가 리뷰 평점 유형 -> 이런거 아직 하나도 안넣었음..
다 조회로직 할 때 맞닥뜨리게 됨.
```javascript
GET https://api.itemscout.io/api/v2/tracking/product/355785
// prev_info ? 나머지는 productDaily 정보인데
```

OOO
user trackings 라는 aggregate 랑 협력할 수 있어야 하고.
조회리스트할 때는 협력하는 것들 고민해보면 될것!

Product에도 유동적인 비즈니스가 들어갈거라 고민해봐야 함. (협력요소들을)