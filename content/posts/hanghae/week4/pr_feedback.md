+++
title = 'week4 PR 피드백'
date = 2025-07-07T00:20:22+09:00
draft = true
+++

🟢🟢🟢WEEK4 PR 포인트 (my)

- 'QueueTokenService Integration Test': redis client mock 주입 적절한지?
-> new IORedis()가 동기적이어서 constructor로 충분함.
(만약 IORedis 연결 자체가 비동기적으로 설정되어야 한다면 onModuleInit이 적합하지만, 현재 new IORedis(...)는 동기적으로 객체를 생성합니다.)

- OnModuleDestroy로 직접 생명주기 관리하는 경우 주의
quit(callback?: Callback<"OK">): Result<"OK", Context>;
export interface ResultTypes<Result, Context> {
    default: Promise<Result>;
    pipeline: ChainableCommander;
}
-> 결국 프로미스 반환임. 
quit 메서드는 선택적으로 callback 함수를 인자로 받을 수 있습니다.
**🔺JavaScript/Node.js 환경에서 함수가 콜백을 인자로 받는다는 것은 해당 함수가 비동기적으로 동작할 가능성이 매우 높다는 강력한 신호입니다. 작업이 완료되면 결과를 콜백 함수를 통해 전달하기 때문입니다.**
-> async onModuleDestroy(): Promise<void> {
    await this.connection.quit();
} 가 되어야 함.

* 문제상황
QueueProducer 가 RedisService에 의존.
QueueProducer의 OnModuleDestroy가 먼저 불리고,
그다음 RedisService의 OnModuleDestroy가 불림.
db가 그러하듯 의존/child가 먼저 정리되고 부모가 정리되는것.

useFactory: 동적으로 프로바이더를 생성해야 할 때 (예: 다른 서비스를 주입받아 조합하거나, 조건에 따라 다른 인스턴스를 반환해야 할 때) 사용합니다.
useValue: 이미 존재하는 객체, 인스턴스, 또는 상수 값을 그대로 주입할 때 사용합니다.



