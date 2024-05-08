+++
title = '유저 트래킹'
date = 2024-05-07T00:20:22+09:00
draft = true
+++
## User Tracking

## TODO
### 도식화
rag graph oop
순서도가 있는.
복잡한 함수는 하나의 순서도로,
그렇지 않으면 도메인 프로퍼티+메서드, 유스케이스.
(usecase랑 service는 구분)

### 어댑터 테스트?
방금 만든 adapter -> e2e 그대로 복사해서 거기에 유닛테스트 만들기.
DB라서 e2e가 나음.
-> 이런경우 어댑터 유닛테스트 만드는게 맞나?

---
### 도메인
유저는 상품을 트래킹한다
트래킹되는 상품은 그룹에 속해있고(optional) 순서를 갖는다

🏀 언제 생성하는가?
- Tracking: create product usecase
- Tracking Group: create group usecase
- order는 컨트롤러로 요청왔을 때 어댑터로 바로.


### 유스케이스
🍀 유스케이스 이름
- 추적할 상품을 등록하는 유스케이스 : create-product.usecase
	- create (Transaction)
		- createProduct
		- createTracking
	- delete
- X 유저와 추적상품을 연관시키는 유스케이스 : -> 이거 API 없애라고 하자!
- 유저가 추적하는 상품을 조회하는 유스케이스: list-tracking.usecase

그룹
- 그룹 생성/삭제 유스케이스: group.usecase
- 추적상품을 그룹에 등록하는 유스케이스: track-group.usecase

키워드
- 추적상품에 키워드를 등록하는 유스케이스: track-keyword.usecase
- 키워드내 해당 상품 순위를 추적하는 유스케이스: keyword-rank.usecase


🍀 도메인 이름

상품: Product
- tpId, metadata

추적상품: Tracking
- userId, tpId, order, group, orderInGroup

그룹: TrackingGroup
- userId, name, order

키워드랭킹: KeywordRank
- tpId, keyword, rank


 
---
💎 지금
두 개를 해보자. 
- group.order? 추가
- updateOrder
: 인자는 { id: order: }

작업 순서
1. 유스케이스 add tracking to group


2. 그룹 순서 도메인 설계/작성


#### 순서 변경은 두 가지 유스케이스다.
1. 그룹 자체의 순서 변경 - Group UseCase
2. 그룹내 순서 변경 - Reorder Tracking Usecase



---
### 💎 정리.
"그룹과 상품간의 관계를 정의하는 도메인이 필요하다"
근데 그게 다대다 관계일 필요는 없다.
그냥 그룹내 순서를 조정해주는 도메인이 필요할 뿐.
왜?
진짜 그냥 다대다면 typeorm 단에서 매핑하는 DAO가 존재함.
근데 내가 필요한 건 관계내 'Order' 라는 다른 속성이고
무엇보다 조회시 그룹 또는 상품에 products[], groups[] 같은 배열을 들고있을 필요가 없다.

save를 하려면 반드시 도메인이 필요한데 
**매핑정보를 save할 때는 유스케이스 & 어댑터 조합으로 가능하고**
get할 때는 이미 쿼리단에서 order 순으로 다 정렬이 된 채로 배열을 반환해 전달하면
그 자체로 order이기 때문에..

그리고 raw query라 해서 도메인 설계 의미가 없는 건 아님.
원래 user_tracking.v2.model 이라는 파일에 
유저를 중심으로 하는 모든 상품쿼리가 다 있었는데,
이걸 '그룹내 상품조회', '유저가 추적하는 상품목록 id[]' 등 다양한 목적으로
유스케이스 & 어댑터 조합으로 쓸 수 있기 때문.

내 경우엔 
#### 🍑 유스케이스
1. 조회: get product list
2. 추가: add to group
3. 순서: reorder tracking
	- setOrder: group product order adapter
	- userId, groupId, tpId를 받아서 OrderInGroup 도메인을 생성한 후 group product order adapter를 호출한다

그외
그룹관련 유스케이스
1. 생성, 순서조정

#### 🍑 어댑터
1-1. mapping product info adapter : 상품정보 매핑 - produdct info 테이블들 포함
1-2. group product list adapter : 그룹내 상품조회 - UPT, URT
2. group product edit adapter : 추가/삭제 - UPT only
3. group product order adapter : 순서조정 - UPT, user tracking 

그외
그룹관련 어댑터
1. group adapter: 그룹 생성, 순서 조정

#### 🍑 DTO
1. 조회
2. 추가
3. 순서 조정
- tagId
- order


---
"정말로 그룹 도메인에 order를 둘 필요 없는지"
"tracking product에 group이 없어도 되는지"
- 그룹을 만든다 V
	- URT에서 해당 유저의 그룹 개수를 카운트. 20개 이상이면 TOO MANY REQUEST
	- 순서로직 X
- 그룹에 상품을 넣는다 V
	- tracking 도메인에 있음? 
	- UPT에 user, group, tpId를 넣는다
		- 도메인: OrderInGroup
		- 어댑터: Reorder Tracking Adapter
			- 
	- 


- 그룹 순서를 변경한다 V
	- 연산이 있음. reOrder: 

- 그룹 조회한다 V
IGetTrackingGroupsUseCase.getList(userId) <- userId가전달되서 쿼리하면됨(order: user_registered_tags)

그룹 이름을 변경한다

그룹을 제거한다


getListWithKeywordStats
- 뭐지? /list에서 UserTracking에 해당 유저와 그룹id를 기반으로 tracking

```javascript
1. 그룹 자체 순서 변경
/v2/tracking/tag/order
orders { '0': 1, '79': 2, '80': 3 }
ordersList [
  { id: '0', order: 1 },
  { id: '79', order: 2 },
  { id: '80', order: 3 }
]
-> orderInfos, 각각의 id와 order를 params, queries 배열에.
params [ '0', 1, '79', 2, '80', 3 ]
queries [
  'SELECT ? AS id, ? AS new_order',
  'SELECT ? AS id, ? AS new_order',
  'SELECT ? AS id, ? AS new_order'
]

 UPDATE user_registered_tags AS urt                     
 JOIN (                         
 	SELECT '0' AS id, 1 AS new_order 
	UNION ALL SELECT '79' AS id, 2 AS new_order 
	UNION ALL SELECT '80' AS id, 3 AS new_order                     ) AS vals ON urt.id = vals.id                     
	SET `order` = new_order                     
	WHERE urt.user_id = 2;                  

```





### 그룹 내의 상품조회
```
IGetProductsInGroupUseCase.getList(userId, groupId)

groupId: 0 => 전체테이블 user_trackings에서 tracking_product_id 빼오면되(order: user_trackings)

groupId != 0 => user_product_tags에서 tp_id를 가져옴 (order: user_product_tags)
```


근데 더 중요한건
상위 개념에서 
유저와 상품간의 관계임.


🐳 유저가 상품을 추가한다
- user tracking에 추가한다 (누가 어떤 상품을 추적하는지)
	- Tracking 도메인을 생성하고 
		- 생성시 userId, product를 넣어둔다.
		- {post} /v2/tracking/product 
		add tracking product -> UserTracking.save
			- 요청시 tpId BODY로 주면 그걸로 Product 생성해서 Tracking 도메인에 넣기.

Q. user tracking:사실 userId, tpId만 있으면 끝임. 
order는 처음에 옵셔널로 주고 DB 스키마단에서 default 1을 넣을 것인가
생성자에서 굳이 1의 값을 넣는? ㄴㄴ

🐳 유저가 그룹을 만든다
- Group 매핑을 만든다 (누가 어떤 그룹을 갖고있는지)
	- Group 도메인 생성시 userId, group name을 넣어둔다.

🐳 유저가 그룹에 상품을 추가한다
- Group 도메인의 product 배열에 추가한다??
ㄴㄴ. 해당 product의 group 속성에 넣어줘야 함.
사실 우리는 '그룹'이 가진 상품들을 저장하지 않는다.

누가 어떤 그룹을 만들었고
누가 어떤 상품을 어떤 그룹에 넣어뒀냐를 저장할 뿐.

그래서 유저가 '그룹과 그룹내 상품들을 조회한다'고 했을 때
유저가 만든 그룹 id들을 조회하고
유저가 추적 중인 상품들 중에서 해당 그룹id에 해당하는 것을 가져오는 것.
- user-product-tags에서 해당 tag_id(group id)를 가진 상품 id를 가져온다

그룹 조회 API
1. v2/tracking/tag : 유저가 가진 그룹
2. /v2/tracking/product/list?pageNum=1&tagId=79 : 그룹내 상품 및 상품정보

그룹 등록 API
1. POST v2/tracking/tag


🐠 tagId가 나오는 케이스 두 가지
- router.get('/list', async (req, res) => {
- router.get('/shorts', async (req, res) => {
	- shorts에도 productInfo 정보는 다 있음. 추적 키워드만 없을뿐.


#### 순서 변경
그룹에도 order가 있고
그룹내 상품도 order가 있다

`/v2/tracking/tag/order`
그룹 자체의 순서 조정 (reorder tracking group)

`/v2/tracking/product/order`
전체 또는 그룹내 상품의 순서 조정
: tagId = 0 여부에 따라 구분

1. 전체에서 순서변경: user_trackings에 반영됨
- tagId: 0
2. 그룹내 순서변경: user_product_tags에 반영됨 
- [{id: 201, order: 1}, {id: 198, order: 2}, {id: 199, order: 3}]
- tagId: N
order 위에서부터 순서로 user_trackings.id를 보냄.(BODY)

자.
그러면 order라는 필드는 어디어디에 존재해야하는가?
- 그룹 자체 (TrackingGroup.order)
- 유저트래킹 (Tracking.order): 전체상품내 순위
- Tracking.orderInGroup : 그룹내 순위 

---
임시.

현재 
userId = 2, tagId = 79 과일, tpId = 166 
upt order = 0
ut order = 1
urt order = 100000

tpId 167 추가

```sql
SELECT
	ut.id AS id,
	ut.tracking_product_id AS tp_id,
	tp.type AS type,
	tp.mall_pid as mall_pid,
	ifnull(sn.name, '확인불가') AS mall_name,
	ifnull(tpi.price, 0) AS price,
	tpi.keywords AS keywords,
	ifnull(tpi.review_count, 0) AS review_count,
	ifnull(tpi.review_score, 0) AS review_score,
	ifnull(tpi.recent_sale, 0) AS recent_sales,
	ifnull(tpi.title, tp.title) AS title,
	ifnull(tpi.image, tp.image) AS image,
	tpi.category_id AS nv_category_id,
	tp.reg_date AS reg_date,
	ut.created_at AS created_at,
	tpif.is_overseas AS is_overseas,
	tpif.is_tied AS is_tied 
FROM (
	//  누가 어떤 그룹내 어떤 상품을 추적하고 있는지 -> UserTracking + UserProductTags 
	SELECT
		ut.*,
		upt.order AS tagOrder
	FROM
		item_scout.user_trackings ut
			LEFT JOIN user_product_tags upt
				ON ut.tracking_product_id = upt.tracking_product_id
				AND upt.tag_id = ?
	WHERE ut.user_id = ? AND ut.tracking_product_id IN (?)
) AS ut

// 해당 유저가 추적하는 상품들에 대해 상품정보를 가져옴 = TrackingProducts, Detail Info는 최신날짜만.
LEFT JOIN tracking_products AS tp
ON ut.tracking_product_id = tp.id
LEFT JOIN (
	SELECT * FROM tracking_product_info
	WHERE (tracking_product_id, date) IN (
		SELECT tracking_product_id, max(date) AS date FROM tracking_product_info
		WHERE tracking_product_id IN (?)
		GROUP BY tracking_product_id
	)
) AS tpi
ON tpi.tracking_product_id = tp.id

// 아까 가져온 상품정보의 storeId를 통해 StoreName을 가져옴
LEFT JOIN store_names as sn
ON tp.store_id = sn.id

// 상품 id를 가지고 info_flags를 가져옴
LEFT JOIN (
	SELECT * FROM tracking_product_info_flags
	WHERE (tracking_product_id, date) IN (
		SELECT tracking_product_id, max(date) AS date
		FROM tracking_product_info_flags
		WHERE tracking_product_id IN (?)
		GROUP BY tracking_product_id
	)
) AS tpif 
ON tpif.tracking_product_id = tp.id

// user_product_tags.order 오름차순(그룹내 상품 추가한 순으로) - user_trackings.order 오름차순 - user_trackings.id 내림차순
ORDER BY ut.tagOrder ASC, ut.order ASC, ut.id DESC;
`;
```


---
### 유저 정보를 가져오는 방법
```typescript
// 컨트롤러에 레거시 가드 추가
@Permit()
@UseGuards(AuthLegacyGuard)

const authUserString = req.headers['auth-user'] as string;
this.clsService.set(AUTH_USER, authUser);

// 컨트롤러/서비스에서 authContextService 주입해서 사용
const authUser = this.authContextService.getUser();

```
- AUTH_USER에 (파싱된) 헤더의 유저정보가 담겨있음.
- AuthContextService.getUserId만 하면 정보 가져올 수.

일단은 컨트롤러에서 dto로 넘겨줄 것인가?
그러기엔 보통 dto에 userId 같은 auth 정보는 정의를 안함..
dto는 진짜로 클라에서 넘겨주는 객체 그 자체고
서비스단에서 dto를 받아서 거기에 userId가 필요하면 추가해서 쓸듯.

Ia