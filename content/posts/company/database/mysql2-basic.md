+++
categories = ['Company']
series = ['Database']
tags = ['MySQL']
title = "mysql2"
date = 2024-09-20T22:50:44+09:00
draft = true
+++
## Pool, connection 기본

### 시도

```ts
export class MySQL {
	private static pool: Pool

	static async initialize() {
		if (!this.pool) {
			this.pool = mysql.createPool({
				host: process.env.DB_HOST,
				port: 3306,
				user: process.env.DB_USER,
				password: process.env.DB_PASS,
				database: process.env.DB_NAME,
			});
		}
	}

	static async query(sql: string, values: any[]) {
		if (!this.pool) {
			throw new Error('Pool is not initialized');
		}

		const conn = await this.pool.getConnection();
		try {
			const [rows] = await conn.query(sql, values);
			return rows;
		} catch (err) {
			console.error(err);
			throw new Error('Failed to query');
		} finally {
			conn.release();
		}
	}
}
```

---
## ESM vs CommonJS
- `tsconfig.json`의 `"module": "ESNext(ESM)"`: TypeScript 컴파일러가 TypeScript 파일을 JavaScript 파일로 변환할 때, 어떤 모듈 시스템을 사용할지를 지정
- `package.json`의 `"type": "module"`: Node.js가 JavaScript 파일을 실행할 때, 해당 파일을 어떤 모듈 시스템으로 해석할지를 지정

tsc로 빌드해보면
`import { TestClass } from "./test-class";`
test.js에는 ESM 형식으로 import 되어있고,
Nodes는 package.json의 "type": "module"에 따라 test.js를 ESM 시스템으로 해석하려 할 것임.
그런데 문제는 ./test-class라는게 확장자가 없다는것.
🔮 **그래서 ESM으로 컴파일하고 해석할 때는 .js 확장자를 붙여줘야 한다는 점에 유의**

---