+++
title = '세션 관리'
date = 2024-10-21T00:20:22+09:00
draft = true
+++
## Session

---
### 버킷으로 관리
```ts
const setCache = await this.redis.hset(
	'session:' + token.slice(0, 4),
	token.slice(4),
	entity.userId.toNumber(),
);
```
16^4 = 65,536개의 버킷이 생성될 수 있음.
#### Q. 왜 hset이 충돌 문제를 완화할 수 있다는거지?

맞습니다! 키가 충돌한다면 같은 버킷에 들어가게 됩니다. 말씀하신 부분이 바로 이 전략의 중요한 점을 건드립니다. 

키 충돌이 발생해도 문제가 완화되는 이유는 **충돌을 완전히 없애는 것이 아니라, 특정 버킷에 몰리는 데이터의 양을 줄이는 것에 목표를 두기 때문**입니다.

**좀 더 자세히 설명하자면:**

1. **단일 키 사용 시:** 모든 토큰이 하나의 해시 테이블에 저장되므로, 충돌 발생 시 해당 테이블 내에서 충돌 해결 메커니즘(Chaining이나 Open Addressing)을 사용해야 합니다. 이는 검색 성능을 저하시키는 주요 원인이 됩니다.

2. **토큰 앞부분을 키로 사용 시:** 
   - 65,536개의 버킷으로 데이터가 분산되므로, 단일 키 사용 시보다 특정 버킷에 데이터가 몰릴 확률이 훨씬 줄어듭니다. 
   - 즉, 충돌이 발생하더라도, 영향을 받는 데이터의 규모가 줄어들기 때문에 성능 저하가 제한적입니다.

**비유를 들어 설명하자면:**

- 100만 명이 사는 도시에 우체국이 하나밖에 없다고 가정해 보겠습니다. 모든 사람이 같은 우체국을 이용해야 하므로 혼잡이 심할 것입니다.
- 이 도시에 우체국을 100개 만들면 어떨까요? 여전히 특정 우체국에 사람이 몰릴 수 있지만, 전체적인 혼잡은 크게 줄어들 것입니다.

**결론:**

토큰 앞부분을 키로 사용하는 전략은 충돌을 완전히 없애는 것이 아니라, **충돌의 영향을 최소화하고 데이터를 분산하여 전체적인 성능을 향상시키는 데 목적**이 있습니다. 

물론, 이상적으로는 충돌이 전혀 없는 것이 가장 좋겠지만, 현실적인 시스템 제약과 성능을 고려했을 때, 이러한 분산 전략은 매우 효과적인 방법입니다. 

---
### Session row를 새로할 것인가 업데이트할 것인가
Session 토큰을 유니크키 & UserId + device token을 유니크키

네, 정확히 그렇습니다! 이 접근 방식은 매우 유용하고 실용적입니다. userId와 device_type을 함께 유니크 키로 사용하면 사용자당 각 디바이스 타입에 대해 하나의 활성 세션만 유지할 수 있습니다. 이 방식의 장점을 살펴보겠습니다:

1. 세션 관리 최적화:
   - 각 사용자는 디바이스 타입별로 하나의 활성 세션만 가질 수 있습니다.
   - 예: 한 사용자가 모바일, 태블릿, 데스크톱 각각에 대해 하나의 세션을 가질 수 있습니다.

2. 자동 세션 갱신:
   - 같은 사용자가 같은 디바이스 타입으로 로그인하면 기존 세션을 자동으로 갱신할 수 있습니다.

3. 데이터베이스 공간 효율성:
   - 불필요한 중복 세션을 방지하여 데이터베이스 공간을 절약할 수 있습니다.

4. 보안 강화:
   - 디바이스 타입별로 세션을 관리함으로써 비정상적인 로그인 패턴을 더 쉽게 감지할 수 있습니다.

이를 구현하기 위해 테이블 구조를 다음과 같이 수정할 수 있습니다:

```sql
CREATE TABLE sessions (
    id SERIAL PRIMARY KEY,
    user_id INTEGER NOT NULL,
    device_type VARCHAR(50) NOT NULL,
    token VARCHAR(255) UNIQUE NOT NULL,
    device_token VARCHAR(255),
    last_activity TIMESTAMP,
    expires_at TIMESTAMP,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    UNIQUE (user_id, device_type)
);
```

이제 세션을 생성하거나 업데이트할 때 다음과 같은 SQL을 사용할 수 있습니다:

```sql
INSERT INTO sessions (user_id, device_type, token, device_token, expires_at)
VALUES (:user_id, :device_type, :token, :device_token, :expires_at)
ON CONFLICT (user_id, device_type) 
DO UPDATE SET 
    token = EXCLUDED.token,
    device_token = EXCLUDED.device_token,
    expires_at = EXCLUDED.expires_at,
    last_activity = CURRENT_TIMESTAMP;
```

이 쿼리는 새 세션을 삽입하거나, 이미 존재하는 경우 기존 세션을 업데이트합니다.

이 접근 방식의 추가적인 이점:

1. 간편한 세션 관리: 사용자별, 디바이스 타입별로 세션을 쉽게 조회하고 관리할 수 있습니다.
2. 일관된 사용자 경험: 같은 디바이스 타입에서는 항상 최신 세션이 유지됩니다.
3. 보안 정책 적용 용이: 디바이스 타입별로 다른 보안 정책을 적용할 수 있습니다.

이 방식은 대부분의 애플리케이션에서 효과적으로 작동하며, 세션 관리를 단순화하면서도 필요한 유연성을 제공합니다.

---
### 참고) 왜 디바이스를 식별?

3. 사용자 경험 개선:
   - 사용자에게 현재 로그인된 디바이스 목록을 제공할 수 있습니다.
   - 사용자가 특정 디바이스에서만 로그아웃할 수 있는 옵션을 제공할 수 있습니다.

4. 디바이스별 설정:
   - 각 디바이스마다 다른 설정이나 권한을 부여할 수 있습니다.
   - 예를 들어, 모바일 디바이스와 데스크톱에 다른 기능을 제공할 수 있습니다.

5. 분석 및 인사이트:
   - 사용자의 디바이스 사용 패턴을 분석할 수 있습니다.
   - 어떤 디바이스에서 주로 접속하는지, 얼마나 자주 사용하는지 등의 정보를 수집할 수 있습니다.

---
