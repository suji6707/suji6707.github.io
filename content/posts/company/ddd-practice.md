+++
title = 'My First Post'
date = 2024-02-25T00:20:22+09:00
draft = true
+++
## 도메인주도 설계

This is **bold** text, and this is *emphasized* text.

생성자. 팩토리. 도메인엔터티.
리팩토링의 핵심이 뭔가? 

Sales라는, KeywordSales 도메인 아래에 존재하는 보조도메인의 역할이 뭔가?
크롤링 데이터를 받아서 매출을 계산하는 로직을 담당한다.
- 객체를 생성할 때부터 productList를 넘기기 때문에 
생성자에서 매출을 계산해서 속성으로 넣어버리는게 Best.
- 아니면 최소한 팩토리 함수내 create 후에라도 리턴하기 전 계산해주던가.

    constructor(productList: ProductRawInfo[]) {
        this._productList = productList;
        this._refinedData = [];
        this._totalCount = 0;

        this.getSalesData();
    }

	getSalesData(): void {



메인 도메인 두 개
- KeywordSales  (KeywordAggregate)
- CategorySales

트랜잭션 아니 base orm extends 어떻게 하는지

### DB 저장은 어떻게 하는가
- 엔터티. 
- 도메인
- 팩토리 생성자
- 어댑터, 레포지토리(interface)

BaseTypeOrmRepository에서 셋팅해둔 덕에 
repository.save는 도메인 -> 엔터티 -> 도메인으로 변환해서 돌려준다(id가 채워져있음)

💎 Category Sales의 경우 
생성자에서 바로 계산할 순 없다. 대신 팩토리 함수 create에서 해야한다.
왜냐면 계산을 위한 파라미터로 KeywordSales 배열을 생성자로 넘겨야한다는 말인데-
이렇게 되면 Entity -> Domain reconstitute에서 곤란해진다.
DB에 저장된 필드만 new CategorySales()의 인자로 들어갈텐데
집계를 위한 배열이 들어간다는건 옛날의 수단이 지금 갑자기 소환해야한다는 것이므로..


### 컨트롤러
localhost:3005/datalab/category/sales/:categoryId
- app.setGlobalPrefix('api/datalab/'); 이렇게 해도 되나? 🍎
- getAllSubcategory sales 




### 유닛테스트


### 🍎 질문 
Q. 왜 dataSource: DataSources는 private이 아닐까?
- transaction manager 파일을 전부 지우셨음.
Q. response.interceptor, core.dto 파일 같이 보기. 
- 예외처리 어떻게 일괄적으로 했는지
인터셉터란게 결국은 미들웨어 아닌가? next
ErrorCommonResponse 가 예외처리 코드 작성시 중요할듯. 

```typescript
export class KeepLoginTypeOrmAdapter
    extends BaseTypeOrmRepository<KeepLoginEntity>
    implements IKeepLoginRepository
{
    private readonly logger: Logger = new Logger(KeepLoginTypeOrmAdapter.name);

    constructor(dataSource: DataSource) {
        super(dataSource, KeepLoginEntity);
    }
```

`a` is number

        id: optional<ID>,
        categoryId: number,
        year: number,
        month: number,
        createdAt: Date,
        updatedAt: Date,
        deletedAt: optional<Date>,