+++
title = '랭킹추적 리팩토링 2. 응답 형태'
date = 2024-02-25T00:20:22+09:00
draft = true
+++
## 랭킹추적 리팩토링 2. 응답 형태
axios.create() 호출을 통한 인스턴스 생성 위치
- 서비스 클래스 내부.

자. 순서.
scrapeNaverShoppingSearchViaOpenApi
- naverApiKeyRepository.getMinUsedNaverApiKey 를 mock할거임. 



#### Axios
```typescript
import axios from 'axios';
import axiosRetry from 'axios-retry';

// 별도의 axios 인스턴스 생성
const axiosWithRetry = axios.create();

// axios-retry 설정 적용
axiosRetry(axiosWithRetry, {
  retries: 3,
  retryDelay: axiosRetry.exponentialDelay
});

// 사용 예
async function retrieve(options: IWebClientOptions): Promise<IWebClientResponse> {
    this.setOptions(options);
    if (!this._options.url) throw new Error('URL 값이 없습니다.');
    
    const response = await axiosWithRetry(this._options);
    return response.data;
}
```
1. 이 코드에서 axiosWithRetry 인스턴스는 axios-retry 설정이 적용된 상태이며, retrieve 함수에서 이 인스턴스를 사용하여 요청을 보내게 됩니다. 이 방식을 사용하면 전역 axios 인스턴스에 영향을 주지 않고 재시도 로직을 필요한 요청에만 적용할 수 있습니다.

2. 인터셉터와 재시도 로직의 범위
인터셉터를 사용할 때, 전역 axios 인스턴스에 적용되는 것이 기본적인 동작입니다. 그러나 별도의 axios 인스턴스를 생성하여 그 인스턴스에만 인터셉터를 적용할 수도 있습니다. 이렇게 하면 다른 요청에는 영향을 주지 않으면서 특정 요청에만 특별한 로직을 적용할 수 있습니다.

3. 요청별로 인터셉터 적용 여부 결정
각 요청마다 인터셉터의 적용 여부를 결정하려면, 요청을 보낼 때마다 인터셉터를 추가하고 요청이 완료된 후 인터셉터를 제거하는 방식을 사용할 수 있습니다. 이 방법은 다소 번거롭고 비효율적일 수 있으므로, 가능한 한 별도의 인스턴스를 사용하는 방법이 권장됩니다.

### 인터셉터와 AOP
ResponseOutput DTO가 인터셉터로 주입된다(constructor)
이로써 인터셉터는 DTO에 정의된 create 라는 메서드를 쓸 수 있다.
DTO 주입은 transformation 로직이 DTO 안에 캡슐화되어있기 때문에 인터셉터를 더 유연하게 해준다.

DTO 메서드
- create는 도메인 raw 데이터를 클라이언트 응답에 맞게 변형해준다.

Response DTO
- infra/adapter/in/dto 에 정의해두고
- 컨트롤러에서 


```typescript
@Injectable()
export class getTrackingInterceptor implements NestInterceptor<TrackingProductResponseOutput> {
  constructor(private responseDto: TrackingProductResponseOutput) {}

  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    return next.handle().pipe(
      map((data) => {
        const result = this.responseDto.create(data);
        return result;
      }),
    );
  }
}

// 클라이언트 반환 필드 : 도메인 필드
id
tp_id: trackingProductId
mall_pid: mallPid

```

---
## 에러처리
### <임시>
이젠 특정 에러객체의 constructor에 에러메시지만 전달하면 됨
- 그 에러객체는 특정 ErrorCode(영어코드)와 ErrorMessage(한글 설명) 메서드를 갖도록 구현해야하고
- logMessage는 그냥 콘솔에만 남는거고

### 🟣 TODO

---
어떤 에러 상황이 있을까?
- 상품 url이 이상하다 - invalid
- 파싱 에러
    - 특정 필드에 값이 없을때 - InvalidValue
- DB 에러
    - 있어야하는데 없을때: not found
    - 저장 에러
        - productsave failed error
        - 이미 각각의 saveProduct에서 해당 에러객체로 throw 했고. 

어떤 에러를 상속할 수 있을까?
- Internal server: 500
- InvalidValue: 400
- notfound 404
- service unavailable: 503
- unknown: 500


---
#### tip & refactoring
```typescript
try {
    const { productDailyInfo, productInfo } = await this.scrapingAdapter.scrapeHTML(product, true);
    if (!productDailyInfo || (needMetadata && !productInfo)) {
        throw new ScrapeNaverStoreFailedError(`Failed to scrape Naver Store: ${product.url}`);
    }
    return { productDailyInfo, productInfo };
} catch (err) {
    this.logger.warn(err.message);  // 로깅은 에러 메시지만 기록
    throw err;  // 이미 ScrapeNaverStoreFailedError가 발생했으므로 그대로 재발생
}
```

---
HttpExceptionFilter -> ExceptionHandleFilter 로 바꿨음.
- throw 해서 잡지 않으면 http 요청에서 에러가 나고
그걸 `HttpExceptionFilter`로 처리하고 있었는데


#### '확장해서 쓴다'
common/domain/error
-> 도메인에서 이걸 확장해서 써야하므로
infra보다는 domain 폴더내 두었음.

도메인에서 발생하는 에러는 모두 여기서 확장해서 쓰면 됨.

NotFoundError 반드시 확장해서 써야함. 
abstract class라서. 

에러패턴을 보면
DB 어댑터 에러는 보통 not found 나 sql 에러메시지.
-> 하나의 internal-server.error 를 만들었음.
크리티컬한 용도로 여기저기서 확장해서 쓰면 됨!

에러메시지는 디폴트로 WARN으로 찍히는데 
sql 에러는 크리티컬해서 ERROR로 찍히게 해두었음.

#### 상속
AbstractError는 생성자로 logMessage, loglevel을 받는다.

Q. 왜 이렇게 여러 계층이 필요할까?
같은 NOT FOUND라 해도 
statusCode는 404로 같을지언정
ErrorCode는 OOO_NOT_FOUND 로 다 다름.
-> **내 도메인에 맞게 클래스를 확장해서 사용해야 함!**

AbstractError 가 CError 같은 추상객체.
logMessage를 생성자에서 받고,
로그레벨을 정의.


에러 발생시 응답 형태를 바꿀 것.
- 현재: 클라는 status랑 error_msg만 바라보고 있음.
비즈니스 에러 코드인 error_msg를 보고 클라에서 직접 alert 메시지를 정의함.
- 앞으로: 서버에서 `extra_msg` 를 통해 직접 alert 메시지 정의할 것.

```typescript
interface ErrorResponse {
    status: 'warn' | 'error'; // 에러 레벨
    result_code: number; // HTTP 상태 코드
    error_msg: string; // 클라이언트 에러 코드
    extra_msg?: string; // 클라이언트 에러 메시지
}
```

#### 외부 인터페이스 에러
1. 데이터베이스
- internal server error

2. 스크래핑 
- service-unavailable
서비스로직에서 요청 결과가 null일 경우 쓰면 될 것.

---
#### Logger 전체 설정(common)

`PinoLoggerService extends Logger`
-> 이 서비스를 common에 주입하고, 이것을 appmodule에 주입함으로써 동작.


serializers: 
객체 로깅할 때 사용되는 커스텀 함수를 정의.
- req: HTTP 요청 객체를 받아 로그에 포함할 정보를 반환
- res: HTTP 응답 객체를 받아 로그에 포함할 정보를 반환
```typescript
pinoHttp: {
    level: process.env.NODE_ENV === 'production' ? 'info' : 'debug',
    serializers: {
        req: (req) => ({
            traceId: req.headers['x-amzn-trace-id'],
        }),
        res: (res) => ({
            statusCode: res.statusCode,
        }),
    },
```

#### TypeOrmLogger
```typescript
export class CustomTypeOrmLogger extends AbstractLogger {
    // local일 때 쿼리를 강조하도록 했음.
    highlightSql: this.nodeEnv === 'local' ? true : false,

// 일반 쿼리는 info, 슬로우 쿼리는 warn.
case 'info':
case 'query':
    if (message.prefix) {
        this.logger.log(`${message.prefix} ${message.message}`);
    } else {
        this.logger.log(message.message);
    }
    break;

case 'warn':
case 'query-slow':
    if (message.prefix) {
        this.logger.warn(
            `${message.prefix} ${message.message}`,
        );
    } else {
        this.logger.warn(message.message);
    }
    break;
```

