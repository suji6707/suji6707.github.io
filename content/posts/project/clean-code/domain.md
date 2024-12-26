+++
title = 'Domain'
date = 2024-11-10T00:20:22+09:00
draft = true
+++
## DDD

```ts
export abstract class BaseDomain {
    toEntity<T>(): T {
        return JSON.parse(JSON.stringify(this));
    }

    static fromEntity<T extends BaseDomain>(this: new (e: any) => T, entity: any) {
        const instance = new this(entity);
        return instance;
    }
}
```
Q. 왜 toEntity에는 static이 안붙나?

`toEntity` 메서드를 `static`으로 만들 수 없는 이유는 이 메서드가 인스턴스의 상태에 의존하기 때문입니다. `toEntity` 메서드는 현재 객체의 속성들을 JSON으로 직렬화하고 파싱하여 새로운 객체를 반환하는 방식으로 구현되어 있습니다. 이 과정에서 `this` 키워드를 사용하여 현재 인스턴스의 속성에 접근합니다.

`static` 메서드는 클래스의 인스턴스가 아닌 클래스 자체에 속하기 때문에 `this` 키워드를 사용할 수 없습니다. `static` 메서드 내에서 `this`는 클래스 자체를 가리키며, 인스턴스의 속성이나 메서드에 접근할 수 없습니다.

따라서 `toEntity` 메서드를 `static`으로 만들면 `this`를 사용할 수 없기 때문에 인스턴스의 상태를 직렬화할 수 없게 됩니다. 이 메서드는 인스턴스 메서드로 유지되어야 현재 인스턴스의 상태를 기반으로 동작할 수 있습니다.

만약 `toEntity`를 `static` 메서드로 만들고 싶다면, 인스턴스를 매개변수로 받아서 처리하는 방식으로 변경해야 합니다. 예를 들어:

```typescript
export abstract class BaseDomain {
    static toEntity<T>(instance: BaseDomain): T {
        return JSON.parse(JSON.stringify(instance));
    }

    async create(session: SessionDomain): Promise<number> {
        const entity = BaseDomain.toEntity(session);
        // ...
    }
}
```
이렇게 하면 `toEntity` 메서드를 `static`으로 만들 수 있지만, 인스턴스를 매개변수로 전달해야 합니다. 이 방식은 `toEntity`가 인스턴스의 상태를 필요로 하기 때문에 인스턴스 메서드로 유지하는 것보다 직관적일 수 있습니다.

---
`this`는 JavaScript와 TypeScript에서 객체의 메서드 내에서 호출될 때, 그 메서드를 호출한 객체 인스턴스를 참조합니다. `BaseDomain` 클래스의 `toEntity` 메서드에서 `this`는 `BaseDomain`을 상속받은 클래스의 인스턴스를 참조합니다.

즉, `BaseDomain`을 직접 인스턴스화할 수는 없지만, 이를 상속받은 클래스가 인스턴스화되었을 때, 그 인스턴스의 메서드로 `toEntity`가 호출되면 `this`는 그 인스턴스를 가리킵니다.

예를 들어, `SessionDomain`이 `BaseDomain`을 상속받는다고 가정하면:

```typescript
class SessionDomain extends BaseDomain {
    constructor(public userId: number, public token: string) {
        super();
    }
}

const session = new SessionDomain(1, 'token123');
const entity = session.toEntity();
```

위 코드에서 `session.toEntity()`를 호출하면, `toEntity` 메서드 내의 `this`는 `session` 인스턴스를 참조합니다. 따라서 `JSON.stringify(this)`는 `session` 객체를 JSON 문자열로 변환합니다.

이처럼 `this`는 메서드를 호출한 객체 인스턴스를 참조하기 때문에, `BaseDomain`을 상속받은 클래스의 인스턴스에서 `toEntity`를 호출하면 그 인스턴스의 상태를 기반으로 동작하게 됩니다.

---