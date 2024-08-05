+++
categories = ['Company']
series = ['Database']
tags = ['TypeOrm Pool Clustor Custom']
title = "TypeOrm"
date = 2024-03-30T22:50:44+09:00
draft = true
+++
## Pool Cluster 재가동 로직

### 해결한 과정
⛑️ typeorm DataSource -> initialize() -> this.driver.connect() ->
MysqlDriver -> PoolCluster -> createConnection

기존 main 서버에서는 Mysql createConnection으로 직접적으로 재가동 로직을 넣었지만,
Nest에서는 TypeOrm을 쓰고있고 - 재가동 방법은 DataSource를 이용한 방법밖에 없다.
- datasource.initialize()가 create 커넥션하는 함수인데,
    - https://github.com/typeorm/typeorm/blob/master/src/data-source/DataSource.ts#L82
    - 1️⃣ 여기서 initialize() 함수를 보면 `this.driver.connect()`가 핵심.
- 그럼 driver는 무엇이냐?
    - https://github.com/typeorm/typeorm/blob/master/src/driver/mysql/MysqlDriver.ts
    - 왼쪽 탭 driver 폴더에 우리가 쓰는 MysqlDriver가 있음 (TypeOrm은 mysql, mongodb, sqlite 등 다양한 데이터베이스 드라이버를 제공한다)
```typescript
// 2️⃣ 직접 node_modules 폴더 보고 MysqlDriver 찾아야 함! 커스텀하는 것이기 때문에..
import { MysqlDriver } from 'typeorm/driver/mysql/MysqlDriver';

const dataSource = new DataSource(options);
await dataSource.initialize();

const driver = dataSource.driver as MysqlDriver;
const poolCluster = driver.poolCluster;
```

- 3️⃣ 그럼 driver의 `connect()` 함수를 본다.
    - 아래와 같이 `createConnectionOptions()`를 쓰고있음을 알 수 있고, 
    - 문제는 protected 함수이기 때문에, 함수를 통째로 우리 코드에 복사하는 수밖에 없었다.
- 아래와 같이 remove 이벤트시 다시 커넥션을 맺는 함수를 추가한다. 

```typescript
// remove 이벤트가 발생하면 똑같은 커넥션을 재귀적으로 계속 만들어줌
poolCluster.on('remove', (node: string) => {
    logger.error(`poolcluster node ${node} was removed.`);
    setTimeout(() => {
        try {
            logger.log(`try ${node} pool reconnect`);
            if (node === 'MASTER') {
                poolCluster.add(
                    'MASTER',
                    createConnectionOptions(
                        options,
                        options.replication.master,
                    ),
                );
            } else if (node.startsWith('SLAVE')) {
                poolCluster.add(
                    node,
                    createConnectionOptions(
                        options,
                        options.replication.slaves[
                            Number(node.slice(5))
                        ],
                    ),
                );
            }
            logger.log(`${node} pool reconnect success`);
        } catch (e) {
            // logger.error('poolcluster remove callback error');
            logger.error(`${node} pool reconnect failed`);
            logger.error(e);
        }
    }, 15000);
});
```

🔴 [remove 테스트] 서버가 죽거나 커넥션이 remove 되는 것 자체는 문제가 아니다.
- 서버가 죽어서 재시작하면 OK
- 서버가 안죽어도 remove된 커넥션을 복구하면 OK

진짜 문제는
서버가 안죽고 remove를 복구하지 못하는 상황이다.
설정된 errorCount가 5개?를 넘어서면 더이상 커넥션 restore을 하지않게 되는데,
서버가 죽지 않았기 때문에
DB 접근하려 할 때마다 `pool does not exist` 하면서 작동을 안하게 되는 것.

-> 그래서 이제 무한대로 재시동하도록 바꿨다고 보면 된다.
TypeOrm 설정옵션(restore)에 의존하지 않고 직접 컨트롤하겠다는 것.

---
### 처음에 생각했던 방법

```typescript
import { Injectable, Logger } from '@nestjs/common';
import { DataSource, DataSourceOptions } from 'typeorm';
import { addTransactionalDataSource } from 'your-transactional-module-path'; // 실제 경로로 변경하세요

@Injectable()
export class CustomDataSourceProvider {
    private dataSource: DataSource;
    private readonly logger = new Logger(CustomDataSourceProvider.name);

    constructor(private readonly options: DataSourceOptions) {
        this.initializeDataSource();
    }

    private async initializeDataSource() {
        this.dataSource = addTransactionalDataSource(new DataSource(this.options));

        // 데이터베이스 연결 이벤트 리스너 추가
        this.dataSource.initialize().then(() => {
            this.logger.log('DataSource initialized');
        }).catch((error) => {
            this.logger.error('DataSource initialization failed', error);
            this.connectWithRetry(); // 초기 연결 실패 시 재시도
        });

        this.dataSource.on('error', (error) => {
            this.logger.error('DataSource error detected', error);
            this.connectWithRetry(); // 연결 중 에러 발생 시 재시도
        });

        await this.connectWithRetry(); // 초기 연결 시도
    }

    private async connectWithRetry(retryCount = 5, delay = 5000) {
        for (let attempt = 1; attempt <= retryCount; attempt++) {
            try {
                await this.dataSource.connect();
                this.logger.log('Database connection established');
                return;
            } catch (error) {
                this.logger.error(`Connection attempt ${attempt} failed:`, error);
                if (attempt < retryCount) {
                    this.logger.log(`Waiting for ${delay / 1000} seconds before next attempt...`);
                    await new Promise(res => setTimeout(res, delay));
                } else {
                    this.logger.error('Failed to connect to the database after multiple attempts.');
                    // 필요한 경우 애플리케이션을 종료하거나 다른 오류 처리 로직을 추가할 수 있습니다.
                }
            }
        }
    }

    getDataSource(): DataSource {
        return this.dataSource;
    }
}

// MysqlConfigProvider에서 CustomDataSourceProvider를 사용하도록 수정
import { Injectable } from '@nestjs/common';
import { DataSourceOptions, DataSource } from 'typeorm';
import { ConfigService } from '@nestjs/config';
import { CustomDataSourceProvider } from './custom-data-source.provider'; // 실제 경로로 변경하세요

@Injectable()
export class MysqlConfigProvider {
    constructor(private readonly configService: ConfigService) {}

    createDataSourceOptions(): DataSourceOptions {
        return {
            type: 'mysql',
            host: this.configService.get<string>('DB_HOST'),
            port: this.configService.get<number>('DB_PORT'),
            username: this.configService.get<string>('DB_USERNAME'),
            password: this.configService.get<string>('DB_PASSWORD'),
            database: this.configService.get<string>('DB_NAME'),
            synchronize: true, // 개발 환경에서만 true로 설정
            logging: false,
            entities: [__dirname + '/../**/*.entity{.ts,.js}'],
        };
    }

    async createCustomDataSource(): Promise<DataSource> {
        const options = this.createDataSourceOptions();
        const provider = new CustomDataSourceProvider(options);
        return provider.getDataSource();
    }
}


// TypeOrmModule.forRootAsync의 useClass 대신 useFactory와 inject를 사용하여 CustomDataSourceProvider를 초기화합니다.
import { Module, Global } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';
import { ConfigModule, ConfigService } from '@nestjs/config';
import { MysqlConfigProvider } from './mysql-config.provider'; // 실제 경로로 변경하세요

@Global()
@Module({
    imports: [
        ConfigModule.forRoot(),
        TypeOrmModule.forRootAsync({
            imports: [ConfigModule],
            inject: [ConfigService],
            useFactory: async (configService: ConfigService) => {
                const mysqlConfigProvider = new MysqlConfigProvider(configService);
                return mysqlConfigProvider.createCustomDataSource();
            },
        }),
    ],
})
export class DatabaseModule {}
```