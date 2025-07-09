+++
title = 'week4 시행착오, 배운점'
date = 2025-07-07T00:20:22+09:00
draft = true
+++

🟣 WEEK4 배운 것

### OnModuleInit (NestJS 생명주기 훅):
OnModuleInit 인터페이스를 구현하고 onModuleInit() 메서드를 정의해야 합니다.
NestJS는 모듈의 모든 의존성이 해결되고, 주입된 프로바이더들이 준비된 후, 그리고 해당 모듈이 의존하는 다른 모듈들의 onModuleInit이 모두 완료된 후에 이 메서드를 호출합니다.
주요 용도:
**🔺비동기적인 초기화 작업 (예: 데이터베이스 연결 설정, 외부 서비스 접속). 
반면 constructor 생성자는 async일 수 없습니다.**
다른 서비스가 완전히 초기화된 후에 실행되어야 하는 로직.
모듈이 완전히 준비되었을 때 한 번 실행해야 하는 설정.
async onModuleInit(): Promise<void> 형태로 비동기 작업을 수행할 수 있습니다.

*
// common module
{
    provide: REDIS_CLIENT,
    useFactory🔮: (): Redis => {
        const client = new Redis({
            host: process.env.REDIS_HOST,
            port: Number(process.env.REDIS_PORT),
        });
이거 하나로 
redisService를 쓰는 모든 프로바이더 의존성 override를 다 피할 수 있었음.
useFactory로 일일이 넣어줄뻔 했는데..
🔮이젠 test에 client 하나만 오버라이드 해주면 됨.!
// test
.overrideProvider(REDIS_CLIENT)
.useValue(RedisClientRef)


### 테스트 의존성
#### 테스트 모듈의 providers:
- Test.createTestingModule({ ... providers: [ReservationService, ...] })와 같이 테스트 모듈의 providers 배열에 특정 서비스(ReservationService)를 넣으면, NestJS는 💎💎💎이 테스트만을 위한 새로운 ReservationService 인스턴스를 만들려고 시도합니다.
- 이때, 이 새로운 ReservationService 인스턴스가 필요로 하는 모든 의존성들은 테스트 모듈의 컨텍스트 내에서 찾을 수 있어야 합니다. 즉, 테스트 모듈의 providers 배열에 직접 명시되어 있거나, 테스트 모듈이 imports 하는 다른 모듈들이 export 하고 있어야 합니다.
- 🔺IUserRepository가 에러 없이 동작하는 이유: AuthModule이 Global이라서 알아서 exports 되어서임.
🔺🔺 결론은 테스트하려는 서비스가 의존하는 모든 걸 provider 배열에 넣어줘야함.

#### override
유닛테스트에선 그 서비스가 의존하는 모든 클래스를 mocking했다면,
통합테스트에선 infra layer 안에서도 external만 testcontainer로 따로 주입하는거임.
-> db, redis 클래스는 실제로 쓰되, 커넥션만 별도 컨테이너로.


#### 테스트 의존성 토큰
서비스 안에서 토큰 상수로 주입했으면 테스트에서도 provider에 동일한 상수로 주입해야함. useClass.
@Inject('QueueTokenService')
private readonly queueTokenService: ITokenService

