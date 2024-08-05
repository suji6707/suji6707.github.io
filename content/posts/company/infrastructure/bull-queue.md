+++
categories = ['Company']
series = ['Bull-Queue']
title = "TypeOrm"
date = 2024-07-11T22:50:44+09:00
draft = true
+++
## 🟢 Nestjs bull queue
추상화고 나발이고.
비상시엔 nodeJS bull queue 써야한다.

job을 add하고 (레디스 큐에)
그것을 FIFO로 가져와서 수행한다-.

애초에 왜 필요한가?
서버에 한번에 100개의 요청이 들어오는데
그 서버가 한번에 처리할 수 있는 요청이 최대 20개라면?
아마 메모리 때문에 터질 것이다.

그럼 해야 할 일을 큐에 넣어두고 꺼내서 worker에 시키면 된다.

`lscpu`를 해보면 DuckDB 쿼리를 수행하는 서버(query01~04?)는 
CPU 코어가 24개고, 
이 서버에서 ncs-rank-query-server의 BULL_COUNT .env 값은 1이다.
즉, 여기서 일을 하는 것이다.
(Bull Count = 0는 프로듀서, Bull Count > 0은 컨슈머.)

이 서버의 최대 작업 CPU는 100% x 24 = 2400%,
쿼리 작업은 CPU 300~400%를 찍고, 피크 600%를 가정했을 때
4개의 Worker를 두면 CPU 최대치에 딱 맞게 일할 것으로 보인다.

자.
Nestjs 구현상
- job add: 컨트롤러에서 API를 통해 `queryQueue.add`
  - 각 job는 func이 있음
- worker.ts는 query01~04 서버에서 standalone application으로 실행됨.
  - auto restart: true인데, while에서 job을 10개 처리할 때마다 프로세스를 종료하게 했고, 재시작해야하기 때문.
  - 참고로 프로듀서 서버에선 main.ts를 일반 앱으로 실행중.
  - 즉, query01~04 서버는 일반적으로 유저 컨트롤러 요청을 받는 서버가 아니라,
  레디스에 기록된 잡을 수행하기만 하는 서버라는 것임.



---
### NestJS의 추상화?

1. 원래 bullmq 기본 사용법
- 생성자에서 QueueService.instance가 이미 존재하는지 확인하고, 존재하면 새로운 인스턴스를 생성하지 않고 기존 인스턴스를 반환

```typescript
import { Queue } from 'bullmq';

enum Queues {
  DEFAULT = 'default',
}

export default class QueueService {
  private queues: Record<string, Queue>;
  private defaultQueue: Queue;

  // 정적변수. 단일 인스턴스 보장
  private static instance: QueueService;

  private static QUEUE_OPTIONS = {
    defaultJobOptions: {
      removeOnComplete: false, // 작업 완료 시 제거 여부
      removeOnFail: false, // 작업 실패 시 제거 여부
    },
    connection: {
      // Redis 서버 연결 옵션
      host: process.env.REDIS_HOST,
      port: process.env.REDIS_PORT,
    },
  };

  constructor() {
    if (QueueService.instance instanceof QueueService) {
      return QueueService.instance;
    }

    // 인스턴스가 존재하지 않는 경우에만 인스턴스 생성
    this.queues = {};
    QueueService.instance = this;

    this.instantiateQueues();
  }

  // 큐 초기화
  async instantiateQueues() {
    this.defaultQueue = new Queue(Queues.DEFAULT, QueueService.QUEUE_OPTIONS);
    this.queues[Queues.DEFAULT] = this.defaultQueue;
  }

  getQueue(name: Queues) {
    return this.queues[name];
  }
}
```

2. NestJS에서 Bull을 사용하는 경우, 큐의 초기화는 일반적으로 NestJS의 의존성 주입(Dependency Injection) 메커니즘을 통해 이루어집니다. 이는 주입된 큐가 어플리케이션의 수명 주기 동안 한 번만 초기화되고, 이후에는 동일한 인스턴스를 재사용할 수 있도록 합니다.

1) Bull 모듈
- BullModule.forRoot와 BullModule.registerQueue를 사용하여 큐를 초기화합니다.
2) QueryController
- @InjectQueue 데코레이터를 사용하여 큐를 주입받습니다. 이 과정에서 큐는 이미 초기화된 상태로 주입됩니다.

--
- 즉, 지금도 여전히 NestJS의 Bull 모듈 설정을 통해 큐를 초기화하고 
컨트롤러로 주입해 쓰고있지만
- 기존 consumer 대신 worker.ts로 작업을 수행하고 있는 것이다.
  - 기존 컨슈머는 '@nestjs/bull'의 @Processor, @Process() 추상화를 쓰고있었고
  - 지금의 worker.ts는 레디스의 BULL_QUEUE_NAME(`ncs_resource::`) 에서 직접 job을 가져다가 각 job의 func 이름에 맞는 usecase를 불러서 처리하고 있는 것.

사실 NestJS에서 컨슈머 클래스는 잡을 add하고 listen하는 방식을 정의한다지만
`mport Queue from 'bull';` 라이브러리에서 직접 아래처럼
큐 prefix + name 설정하고 getNextJob로 가져오면 그만임.

```typescript
    const queue = new Queue('query', {
        prefix: configService.get('BULL_QUEUE_NAME'),
        redis: {
            host: configService.get(REDIS_HOST),
            port: Number(configService.get(REDIS_PORT)),
            password: configService.get(REDIS_PASSWORD),
        },
    });

    let i = 0;
    while (i < 10) {
        const job = await queue.getNextJob();
```

🤔 궁금한건
NestJS 컨슈머 버전에선 `@Process({ concurrency: parseInt(process.env.BULL_QUEUE_COUNT || '0') })`
으로 큐의 동시작업을 제어했다면
지금은 pm2 run.json 클러스터 단에서 4 instance가 동시작업한다는건가? 



🤔 로컬에선 프로듀서 / 컨슈머가 같은 컴퓨터라
BULL_COUNT가 1.

로컬에선
api server(3008) - [ query server(3009) - query01(3009) ]

운영환경에선 반대로 
[ api server - query server ] - query01~04
테섭 api 서버와 query 서버가 같은 컴터에 있음.
도메인만 다를뿐, nginx로 포트포워딩 하겠지. 