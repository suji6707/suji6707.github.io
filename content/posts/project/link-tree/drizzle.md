+++
title = 'Drizzle'
date = 2024-10-10T00:20:22+09:00
draft = true
+++
## Drizzle

postgresql로 ㄱ.
객체지향 데이터베이스?


---

NestJS에서 Drizzle ORM을 사용하는 예시를 보여드리겠습니다. 이를 위해 모듈, 서비스, 그리고 컨트롤러를 만들어 보겠습니다.

1. 먼저, Drizzle 설정을 위한 모듈을 만듭니다:

```typescript
// src/drizzle/drizzle.module.ts
import { Module, Global } from '@nestjs/common';
import { ConfigModule, ConfigService } from '@nestjs/config';
import { Pool } from 'pg';
import { drizzle } from 'drizzle-orm/node-postgres';
import { DrizzleService } from './drizzle.service';

@Global()
@Module({
  imports: [ConfigModule],
  providers: [
    {
      provide: 'DRIZZLE_ORM',
      useFactory: async (configService: ConfigService) => {
        const pool = new Pool({
          connectionString: configService.get<string>('DATABASE_URL'),
        });
        return drizzle(pool);
      },
      inject: [ConfigService],
    },
    DrizzleService,
  ],
  exports: [DrizzleService],
})
export class DrizzleModule {}
```

2. Drizzle 서비스를 만듭니다:

```typescript
// src/drizzle/drizzle.service.ts
import { Injectable, Inject } from '@nestjs/common';
import { NodePgDatabase } from 'drizzle-orm/node-postgres';

@Injectable()
export class DrizzleService {
  constructor(
    @Inject('DRIZZLE_ORM') private readonly db: NodePgDatabase,
  ) {}

  async executeQuery(query: string) {
    return this.db.execute(query);
  }
}
```

3. 컨트롤러에서 Drizzle 서비스를 사용합니다:

```typescript
// src/app.controller.ts
import { Controller, Get } from '@nestjs/common';
import { DrizzleService } from './drizzle/drizzle.service';

@Controller()
export class AppController {
  constructor(private readonly drizzleService: DrizzleService) {}

  @Get()
  async getHello() {
    const result = await this.drizzleService.executeQuery('SELECT 1');
    return { result };
  }
}
```

4. 마지막으로, 앱 모듈에 DrizzleModule을 임포트합니다:

```typescript
// src/app.module.ts
import { Module } from '@nestjs/common';
import { ConfigModule } from '@nestjs/config';
import { DrizzleModule } from './drizzle/drizzle.module';
import { AppController } from './app.controller';

@Module({
  imports: [
    ConfigModule.forRoot(),
    DrizzleModule,
  ],
  controllers: [AppController],
})
export class AppModule {}
```

이 설정을 사용하면 NestJS 애플리케이션에서 Drizzle ORM을 사용할 수 있습니다. `DrizzleService`를 통해 데이터베이스 쿼리를 실행할 수 있으며, 필요한 곳에서 이 서비스를 주입하여 사용할 수 있습니다.

환경 변수에 `DATABASE_URL`을 설정하는 것을 잊지 마세요. 이 URL은 PostgreSQL 데이터베이스 연결 문자열이어야 합니다.

---