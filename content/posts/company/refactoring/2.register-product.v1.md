+++
title = '랭킹추적 리팩토링 1. 상품 등록'
date = 2024-02-25T00:20:22+09:00
draft = true
+++
## 랭킹추적 리팩토링 1. 상품 등록

### 주요 논의사항
#### Tracking Product는 상속인가 추상클래스 구현인가?
    - 상위 클래스를 둔다면 그 의도는?
    - 쿠팡 등 여러 플랫폼.
    - 물론 url 외 모든 정보를 metadata로 묶으면 상위 클래스를 상속하도록 할 수도 있지만.
    - 지금은 그런 확장성까진 안간거라. 그리고 nvmid, mallPid는 밖으로 빼둬야 함.
    - 💎 따라서 메서드만 구현하도록 추상클래스를 두는게?

혹은 네이버 안에서 Tracking Product 를
smartstore, brandstore, shopping window로 상속하도록 할 수 있는데- 그게 url만 좀 바뀌지 딱히 크게 구분할 이유는 없어서 패스.

참고로 추상클래스 구현은
메시지만 교환한다. 부모 객체 자체를 공유하는 게 아니라. 
함수도 반드시 각각 구현해야하고.

⁃⁃⁃⁃⁃⁃⁃⁃⁃⁃
#### Permission에 대해
- 왜 auth가 아닌 trackings 모듈에 두는 게 좋은가?
비즈니스 로직이라서. 
- 몇 개 상품/키워드를 추적할 수 있느냐는 비즈니스로 들어와있어야 함.

서버에서 유저 권한을 붙여서 role만 전달하면
트래킹 서버에서는 User의 role 속성 안에 구체적인 허용 개수 등을 json 타입으로 갖고있을듯.

- 가상의 Permission Policy 오브젝트를 선언 (도메인? 서비스?)
    - 이 policy가 권한 체크를 수행한다.
    - **User 도메인 생성시 가상의 권한을 넣어주고 addProduct할 때 권한 개수 따져서 실패처리 할 것**
    - new User 시 constructor로 permission 넣어주고 그것에 맞게 개수가 고정되어 있을듯?

```
getProductsLimit
getKeywordsLimit
```

##### Permission logic
- _get
    - User의 auth 정보에 따라 
        - 6, 7, 8이면 ext.rank.gosu=6
        - 5이면 전부 1
        - 4면 gosu만 1
        - 3이면 ranking만 1
        - 1이면 전부 0인데
            - GroupUser.findInfo로 그룹 존재하면 전부 1,
            - group 속성 추가해서 반환
    - 구독중인 서비스가 있으면 serviceId에 따라
        - 15부터 rank 1 (14는 extension만 있음)
        - 16은 rank 2

    
- product limit
    - ranking, group으로 판단.
    - group이 아니면 아래 로직
    - ranking이
        - 1이면 10
        - 2: 20, 3: 40, 5부터는 백개 이상

/usage API 응답 자체는 추적가능 개수로 나가겠지만
서버단에선 Policy로 유저가 추적하려는 상품개수?를 인자로 받아서 
trackable한지 계산후 가능하지 않으면 에러를 던지는 로직이 필요함.
키워드는 개수 체크해서 가능한 개수까지만?


##### 현재 개수
user_trackings를 체크한다 (tracking products)

💎 tracking product 도메인과 tracking keyword 도메인을 따로 두자.
물론 keyword.trackings => trackingProduct.keywords로 
relation을 두어도 좋겠지만
한편으로는 키워드 분석 프로젝트에서는 trackings가 필요없음.
즉, user_tracking_keywords (tracking-keyword) 에서만 
user - product - keywords 관계가 설정되어있으면 될듯.
그리고 굳이 product에 릴레이션 설정하지 말고 find로 해오면 됨. 

뭐가 있어야 할까?




##### 가능 개수



---
##### "도메인 관점에서 policy를 넣을까 말까" -> 분리하기로
어떻게보면 사고방식을 배우는 건데-
그분이 말하고자 하는건 Policy 로직을 상품등록하는 유스케이스와 분리해두고
DI로 주입할거냐의 이야기임.
(그게 아니면 선정산 서비스의 경우 아예 생성자에서 policy를 넣어주긴 했었음. calculate 로직에 policy가 적용되는거라)

Policy 서비스를 따로 만들면 
거기서 user, group user, subscription 같은 외부 레포들을 갖다 써야할거고.
실제 허용여부 boolean은 도메인에서 다 하겠지. switch case랑.

서비스로 분리한다면
서비스를 만들고 그걸 감싸는 데코레이터를 만들어도 될듯.
데코레이터에서 그 함수를 부르면 되니까.
유스케이스에서는 permission 유스케이스가 쓰였는지도 모를것.

--
user tracking ?
사실상 그냥 Tracking임. userId가 속성에 있어서 굳이.

어차피 유저 조회 유스케이스에서 
User 도메인으로 products, keywords가 묶여서 나갈거라.

(참고)
service vs usecase
- service는 데이터 읽기/쓰기 작업을 캡슐화.
또는 외부 API 리소스에 직접 접근하는 로직
- usecase: 비즈니스 요구사항을 충족하는 작업의 흐름. 여러 service를 조합하여 비즈니스 로직을 수행. 


```
// 클라이언트용
router.get('/usage', async (req, res, next) => {}
let productLimit = await PermissionLogic.getProductsLimit(userId);
let keywordLimit = await PermissionLogic.getKeywordsLimit(req.currentUser);
```

```
// 서버용

```

⁃⁃⁃⁃⁃⁃⁃⁃⁃⁃
#### 데이터 매니저의 역할은 사실 어댑터임
비즈니스 핵심로직은 Tracking Product 도메인과 Ranking 으로 내려가야함!!

순서는
1. Register usecase에서 register 호출
2. Permission Policy 오브젝트가 유저의 권한을 체크해 추적가능한 개수를 넘어서면 에러를 던진다
3. 바로 outport 어댑터로 스크랩 함수를 호출한다
4. 어댑터에선 scrape smartstore 후 두 개를 파싱한다
5. TrackingProduct 도메인 생성시 파싱된 객체를 넣는다
6. 마지막으로 user tracking product에 삽입한다.

사실 User라는 도메인이 존재한다기보다는
User가 가진 Tracking Product라는 가방만이 있을 뿐.
여기에 담을 수 있느냐 없느냐,
이 Product는 어떤 키워드가 등록되어있느냐.
따라서 유저와 관련된 도메인은 User Tracking Product만 있으면,
그 하위에 User Tracking Keywords를 OneToMany로 두면 끝.

```
@@@ TODO
- nestjs-base 체리픽 remote 설정해서.
- 쿠팡(tracking-api)에서 permission logic 체크
- 선정산에서 policy 쓰는 방식 체크
```

--- 
#### 🍋 구현레벨
router.get('/check -> 등록.
router.get('/update/:tid -> 업데이트. 언제 불리지?는 모르더라도
-> 어떤정보를 어떻게 업데이트하는지가 중요.

force 를 둘필요 없이 
각 라우트에 따라 처리.
simple은 tracking product를 들고오므로 matchUrl 할필요 없음

---
#### 기타 논의 사항들
##### 로거
DB 레이어 - 옵셔널 리턴이 아니라 없으면 throw Error를 할 것.
서비스 함수에선 catch(()=> failed to save '')
-> 💎 앞으로 어댑터 쿼리함수 작성할 때 select 결과가 없으면 catch에서 throw Error 처리.

DB에서 발생한 에러는 DB 에러로 '로깅'을 하고싶은건데.
throw err를 한다면 -> 서비스로 throw를 한다는거고.
서비스에서는 데이터베이스에 대한 에러를 내가 표현할 필요가 없게 됨.!!
-> 이전처럼 if (!saved) throw Error 를 서비스에서 할게 아니라
물론 catch로 하긴 하는거지만
먼저 DB 어댑터단에서 에러가 나는거라. 좀더 명확히 구분된다는 의미인듯?

로거는 common에서 주입할거임.
- tracking api는 다 pinno logger인데.
그렇게 되면 전부 pino logger에 의존하게 됨.
그러면 로거 바꾸려면 다바꿔야..
- 근데 커먼을 쓰면 
common에서 로거를 만들어
main.ts에서 그 Logger만 주입해주면 됨.
- 이젠 (서비스단에서) pino를 안쓰고. 
원래는 컨텍스트 지정하려고 했던건데
이렇게 상단에서 주입하면 common도 컨텍스트 지정 됨. 


SQL쿼리 로그:
```
Typeorm에 pinno logger를 주입한거임.
- common/persistence / logger / pino-typeorm-logger.ts
- mysql-config에 new PinoTypeormLogger() 에 주입했음.
```

로그를 winston으로 교체
헤더도 컨트롤러 처음 요청할 때만 찍히긴 할텐데
이것도 과하다 싶으면 
웹서버(Nginx)에서 헤더를 쓴다던가
L7 레이어에서 분석하는것도 가능함.

appModule에 EventEmitter 주입되어있음
-> 배치쪽에선 EventDriven 설계시 필요할수도 있음.

로그를 비즈니스용으로 분석할 수도 있다.
우리는 전부 버리고 있지만..
그 경우 pino가 아니라 직접 구현해야할 수도.?


#### 캐싱
API 만 캐싱할 것인가
데이터를 캐싱할 것인가?

```
결론은 캐시를 제외하고 구현할 수 있게 할 거고,
서비스 레벨에선 캐시를 전혀 몰라도 되게끔 인터셉터나 annotation 으로 처리하는게 목표.
AOP.
어쨌든 캐시의 구현체는 레디스가 아니게 될 것.
레디스는 우리처럼 사람마다 조회하는 키워드가 너무 다르고
무엇보다 무한한? 키워드를 지원하기에는 - 
히트율이 떨어져서 본질적인 의미를 상실했다고 본다. 
```

내가 말하는, 서비스단에서 데이터 가공을 해야하는 건 데이터 캐싱이고 API 캐싱도 있음.
응답 자체가 캐싱된게 API 캐싱이고 이건 Nest에서는 컨트롤러단도 안거치고 미들웨어/인터셉터 단에서 나갈 수 있음.

데이터 캐싱은 서비스 레이어로 내려와야.
캐시에서 가져온 데이터들로 한번 가공해야 할 때. 매핑이라던가.


```
요청을 막아준다.
서버 부하를 막아준다.
비즈니스 횟수를 

캐시의 의도가 완전히 다름.

DB 레이어에서 캐싱을 하냐
레디스 메모리에서 캐싱을 하냐의 차이일뿐
비즈니스 레이어에서 연산을 하는 건 똑같음.

지금 말하는 연관키워드쪽은 데이터영역 캐싱인데.

---
데이터 레벨 캐싱이 있고
http 도 캐싱이 있고,
캐시히트 레벨을 두 개 둘 수 있는것.

로컬, 리모트캐시, 데이터베이스 캐시가 있는데.

매커니즘은 똑같고.

* 데이터베이스 캐싱의 의미?
java 에서는 매퍼라는 걸로 
SQL -> 쿼리 조회를 함.

문서 작성. 동일한 스펙
데이터 쿼리가 같으면.

이게 어디에 저장되냐 - 
레디스냐, 로컬 메모리에 두면 거기도 되고, 
어디에 저장할거냐 라는 스토리지 메커니즘의 문제인거지.

스토리지 히트를 할거냐만 말하면 됨. 

스펙이 같은 데이터면 동일하게 나가면 되는데.
레디스든 로컬이든 다른 RDBS 써도 됨.
저장소라는 개념으로 보면.

* 데이터 캐시라고 함.
외부 메모리일수 있고 그게 레디스로 쓰고있는 것.
구현체의 이름인거임.
멤캐시를 써도 되고.
NoSQL이라던가. 
```

#### 키워드 분석 도메인 논의
Keyword Analyst 라는 상위 클래스는
연관키워드 분석 구현체, 키워드 차트 구현체, 키워드 컨텐츠 구현체 등
여러 Analyst 구현체로 나뉜다.
아마 abstract implements로 갈 것 같고. 하나의 메시지를 상속할 뿐.

오브젝트 생성시 데이터를 주입.

price, prdCnt, monthlySearch 등 각각의 도메인들은
그 멤버변수로 this.keyword가 있을듯. 
이 keyword는 id, keywordStr만 가진 빈 껍데기에 불과하고.

여러 Analyst에서 각 도메인들을 keyword.id 중심으로 모아서 합쳐서 내보낼 것. 

---
(참고)
Tracking Product는 싱글톤이 아님.
전체 애플리케이션에 걸쳐 하나만 존재하는가?
심지어 한 명의 유저 요청에 대해서도 여러개 생성됨.
따라서 멀티 스레드 작업 다 가능함. 


---
#### 타입 상속
똑같은 인터페이스/타입에 필드 하나만 다르다고 똑같이 복사해서 쓸 필요 없다.
extends 상속하면 됨.

`this._metadata = productInfo;` 에서 _metadata에 nvmid, mallPid가 중복해서 들어가겠지만- 여기선 문제가 되지 않음.
꺼내쓰지 않으면 되므로.
문제는 entity 변환할 때 단순히 spread로 하면 nvMid가 두 개가 되어 오류가 나므로 풀어서 써줘야한다는 점 정도.?

심지어 ProductInfo > ProductMetadata 집합이 모두 포함되어있을 때
ProductInfo 자리에 ProductMetadata 를 명시해도 
타입스크립트는 허용함. 
요구되는 필드가 모두 있기만 하면 에러를 내지 않음..

즉 모자라면 에러지만 있을 게 다있고 추가로 필드가 더 있어도 그건 
꺼내쓰지만 않으면 되므로 상관이 없다는 것.

```typescript
export interface ProductInfo extends ProductMetadata {
    nvMid?: string;
    mallPid?: string;
}

export interface ProductMetadata {
    type: number;
    hash?: string;
    storeId?: number;
}

export class Product extends Aggregate {
    private _url!: string;
    private _nvMid?: string; // review_hash
    private _mallPid?: string;
    private _metadata?: ProductMetadata;

    // constructor 생략

    setProductInfo(productInfo: ProductInfo) {
        this._nvMid = productInfo.nvMid;
        this._mallPid = productInfo.mallPid;
        this._metadata = productInfo;
    }
}
```

---
#### 배송정보?
```javascript
"productDeliveryLeadTimes": [{
    "rangeNumberText": "1",
    "rangeText": "일 이내",
    "leadTimeCount": 40,
    "leadTimePercent": 38
}, {
    "rangeNumberText": "2",
    "rangeText": "일 이내",
    "leadTimeCount": 51,
    "leadTimePercent": 48
}],
"sellerDeliveryLeadTimes": [{
    "rangeNumberText": "1",
    "rangeText": "일 이내",
    "leadTimeCount": 160,
    "leadTimePercent": 44
},
```

제공된 데이터는 특정 상품의 배송 시간과 관련된 정보를 나타내고 있습니다. 여기에는 productDeliveryLeadTimes와 sellerDeliveryLeadTimes 두 가지 섹션이 포함되어 있습니다. 각 섹션은 배송 시간 범위와 그 범위 내에서의 배송 건수 및 해당 배송 건수가 차지하는 비율을 나타냅니다.

productDeliveryLeadTimes
이 섹션은 상품이 구매자에게 도달하는 데 필요한 일수를 나타내며, 각 항목은 다음과 같은 정보를 포함합니다:

rangeNumberText: 배송 시간 범위를 나타내는 숫자입니다.
rangeText: "일 이내" 또는 "일 이상"과 같이 시간 범위를 설명하는 텍스트입니다.
leadTimeCount: 해당 배송 시간 범위에 속하는 배송 건수입니다.
leadTimePercent: 전체 배송 건수 대비 해당 범위의 배송 건수가 차지하는 비율입니다.
예를 들어, 첫 번째 항목에서는 "1일 이내"에 배송이 완료되는 건수가 40건이고, 이는 전체 배송 건수의 38%를 차지한다는 것을 의미합니다.

sellerDeliveryLeadTimes
이 섹션은 판매자가 상품을 배송하기 시작하는 데 필요한 일수를 나타냅니다. 구조는 productDeliveryLeadTimes와 유사합니다.

이 데이터는 판매자 또는 제품 관리자가 배송 효율성을 평가하고 고객에게 예상 배송 시간을 더 정확하게 알려줄 수 있도록 도와줍니다. 또한, 배송 프로세스의 개선점을 식별하는 데 사용될 수 있습니다.