+++
title = 'week1 PR 피드백'
date = 2025-07-07T00:20:22+09:00
draft = true
+++


WEEK 1. TDD

Q1. 애플리케이션 확장성 (Scale-out): 현재 Map을 사용한 락은 애플리케이션 인스턴스가 하나일 때만 유효합니다. 만약 로드밸런서를 사용해 여러 인스턴스로 확장(scale-out)할 경우, 각 인스턴스가 자신만의 Map을 가지므로 유저 ID에 대한 락이 공유되지 않아 동시성 문제가 여전히 발생할 수 있습니다. 분산 락(Distributed Lock, 예: Redis의 SETNX 또는 Redlock 알고리즘, ZooKeeper 등) 도입을 고려하고 계신가요?


Q2. 락의 타임아웃 및 재시도 정책: setTimeout(resolve, 100)으로 100ms씩 대기하는데, 만약 락을 점유한 작업이 예상보다 훨씬 오래 걸리거나 실패하여 락을 해제하지 못하는 경우, 다른 요청들은 무한정 대기하게 될 수 있습니다.
락 대기 최대 시간(timeout) 설정 및 초과 시 에러 처리 (예: "일시적인 오류가 발생했습니다. 잠시 후 다시 시도해주세요.") 전략이 있나요?
🔴 Busy-waiting 중 최대 재시도 횟수 제한은 고려되었나요?
- 대안: 
  - 메시지큐: 충전 요청을 바로 처리하는 대신, 메시지 큐(예: RabbitMQ, Kafka, AWS SQS)에 작업을 적재하고, 별도의 워커(Worker)가 큐에서 작업을 하나씩 꺼내 순차적으로 처리하는 방식. 동시접근 자체가 줄어듦.
  - Redis 분산락(Redis의 SETNX 등으로 분산 락을 시도하고, 락 획득에 실패하면 요청을 바로 거절하거나, 또는 해당 요청 정보를 Redis의 리스트(List) 같은 자료구조에 넣어두고 나중에 재처리)
  - SELECT ... FOR UPDATE 같은 DB 락을 사용하면, 락이 해제될 때까지 DB 커넥션이 대기하게 됩니다. 이 경우 DB 커넥션 풀 관리와 타임아웃 설정이 중요합니다.
  - (HTTP 429 Too Many Requests 또는 503 Service Unavailable) 같은 에러를 반환하고 클라이언트 단에서 재시도를 유도

Q3. 락 해제 보장: charge 메소드 내에서 락을 획득한 후, userPointRepository.selectById나 historyRepository.insert 등 DB 작업 중 예외가 발생하면 this.lock.delete(userId)가 호출되지 않아 해당 유저에 대한 락이 영구적으로 남아있게 됩니다. try...finally 구문을 사용해 어떤 상황에서든 락이 반드시 해제되도록 보장해야 하지 않을까요?
- 🔴 중간 에러 발생시키고 락이 해제되는지 확인

Q4. 락의 영향 범위: charge 메소드 전체에 락을 거는 것이 최선일까요? 예를 들어, userPointRepository.selectById(userId) 호출은 락 없이 수행하고, 실제 데이터 변경이 시작되는 시점부터 락을 거는 등 락의 범위를 최소화할 수 있는 부분이 있을까요? (현재 로직에서는 selectById 결과에 따라 분기하므로 쉽지 않아 보이지만, 일반적인 고민 사항입니다.)

기타
* 멱등성(Idempotency): charge 요청이 네트워크 오류 등으로 인해 중복으로 전달될 경우, 사용자에게 포인트가 이중으로 충전될 수 있습니다. 각 요청에 고유한 ID(예: requestId 또는 idempotency-key)를 부여하고, 이를 서버에서 확인하여 이미 처리된 요청인지 검사하는 멱등성 처리

* 데드락(Deadlock) 가능성: 현재 코드는 단일 리소스(userId)에 대한 락이므로 데드락 위험은 낮지만, 여러 리소스를 함께 락킹해야 하는 더 복잡한 로직이 추가될 경우 데드락 발생 가능성을 항상 염두에 두어야 합니다. (락 순서 일관성 유지 등)


