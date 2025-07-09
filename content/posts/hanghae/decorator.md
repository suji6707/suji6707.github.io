+++
title = 'decorator'
date = 2025-07-07T00:20:22+09:00
draft = true
+++

#### To read
https://toss.tech/article/nestjs-custom-decorator



---
`@Reflect.metadata`와 커스텀 데코레이터의 관계를 명확히 설명해드리겠습니다.

## 핵심 개념

`@Reflect.metadata`는 **기본 빌딩 블록**이고, 커스텀 데코레이터는 **이를 활용한 고수준 추상화**입니다.

## 1. @Reflect.metadata의 역할

```typescript
// 이것은 "메타데이터를 저장하는 기본 데코레이터"
@Reflect.metadata("role", "admin")
@Reflect.metadata("permissions", ["read", "write"])
method() {}
```

여기서 `@Reflect.metadata`는:
- **데코레이터 팩토리**입니다 (함수를 반환하는 함수)
- 실제로는 `Reflect.defineMetadata`를 호출하는 데코레이터를 만들어줍니다

## 2. Reflect.metadata의 내부 구현 (개념적)

```typescript
// Reflect.metadata는 대략 이렇게 동작합니다
namespace Reflect {
  export function metadata(key: any, value: any) {
    // 데코레이터 함수를 반환
    return function decorator(target: any, propertyKey?: string) {
      defineMetadata(key, value, target, propertyKey);
    };
  }
}
```

## 3. 커스텀 데코레이터 만들기

### 방법 1: Reflect.metadata를 반환

```typescript
// 이 방법은 "데코레이터로 데코레이터를 만드는" 방식
function Role(role: string) {
  return Reflect.metadata("role", role);  // 기존 데코레이터를 반환
}

function Permissions(...perms: string[]) {
  return Reflect.metadata("permissions", perms);
}

// 사용
class Controller {
  @Role("admin")           // 내부적으로 @Reflect.metadata("role", "admin")
  @Permissions("read", "write")  // 내부적으로 @Reflect.metadata("permissions", ["read", "write"])
  method() {}
}
```

### 방법 2: 직접 메타데이터 설정

```typescript
// 이 방법은 "직접 데코레이터를 구현"하는 방식
function Role(role: string) {
  return function(target: any, propertyKey: string) {
    // 직접 메타데이터 설정
    Reflect.defineMetadata("role", role, target, propertyKey);
  };
}

function Permissions(...perms: string[]) {
  return function(target: any, propertyKey: string) {
    Reflect.defineMetadata("permissions", perms, target, propertyKey);
  };
}
```

## 4. 더 복잡한 커스텀 데코레이터 예시

### 단순한 경우 (Reflect.metadata 활용)

```typescript
// 간단한 경우: 그냥 메타데이터만 저장
function GET(path: string) {
  return Reflect.metadata("route", { method: "GET", path });
}

class ApiController {
  @GET("/users")
  getUsers() {}
}
```

### 복잡한 경우 (직접 구현)

```typescript
// 복잡한 로직이 필요한 경우: 직접 구현
function ValidateEmail() {
  return function(target: any, propertyKey: string) {
    // 1. 메타데이터 저장
    Reflect.defineMetadata("validation:email", true, target, propertyKey);
    
    // 2. 기존 값 저장
    let value = target[propertyKey];
    
    // 3. getter/setter로 실시간 검증
    Object.defineProperty(target, propertyKey, {
      get() {
        return value;
      },
      set(newValue: string) {
        if (newValue && !/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(newValue)) {
          throw new Error(`${propertyKey} must be a valid email`);
        }
        value = newValue;
      }
    });
  };
}

class User {
  @ValidateEmail()
  email: string;
}
```

## 5. 실제 사용 패턴 비교

### 패턴 1: 기본 메타데이터 활용

```typescript
// NestJS 스타일 - 단순하고 선언적
function Controller(prefix: string) {
  return Reflect.metadata("path", prefix);
}

function Get(path: string) {
  return Reflect.metadata("route", { method: "GET", path });
}

@Controller("api/users")
class UserController {
  @Get(":id")
  getUser() {}
}

// 메타데이터 읽기
const controllerPath = Reflect.getMetadata("path", UserController); // "api/users"
const routeInfo = Reflect.getMetadata("route", UserController.prototype, "getUser");
// { method: "GET", path: ":id" }
```

### 패턴 2: 복잡한 로직 포함

```typescript
// 여러 메타데이터를 조합하고 검증 로직 포함
function ApiEndpoint(config: { method: string; path: string; auth?: boolean }) {
  return function(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
    // 여러 메타데이터 저장
    Reflect.defineMetadata("http:method", config.method, target, propertyKey);
    Reflect.defineMetadata("http:path", config.path, target, propertyKey);
    Reflect.defineMetadata("http:auth", config.auth || false, target, propertyKey);
    
    // 기존 메서드 래핑
    const originalMethod = descriptor.value;
    descriptor.value = function(...args: any[]) {
      console.log(`API call: ${config.method} ${config.path}`);
      
      // 인증 체크
      if (config.auth) {
        // 인증 로직...
      }
      
      return originalMethod.apply(this, args);
    };
  };
}

class ApiController {
  @ApiEndpoint({ method: "GET", path: "/users", auth: true })
  getUsers() {
    return "users data";
  }
}
```

## 6. 조합 패턴

```typescript
// 여러 데코레이터를 조합
function HttpGet(path: string) {
  return function(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
    // 기본 메타데이터 데코레이터들을 조합
    Reflect.metadata("http:method", "GET")(target, propertyKey, descriptor);
    Reflect.metadata("http:path", path)(target, propertyKey, descriptor);
    
    // 추가 로직
    console.log(`Registered GET ${path}`);
  };
}

// 또는 데코레이터를 직접 적용
function Post(path: string) {
  return function(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
    // 여러 기본 데코레이터를 한번에 적용
    const decorators = [
      Reflect.metadata("http:method", "POST"),
      Reflect.metadata("http:path", path),
      Reflect.metadata("http:hasBody", true)
    ];
    
    decorators.forEach(decorator => decorator(target, propertyKey, descriptor));
  };
}
```

## 핵심 정리

1. **@Reflect.metadata**: 가장 기본적인 "메타데이터 저장" 데코레이터
2. **커스텀 데코레이터**: 
   - `Reflect.metadata`를 반환하거나 (간단한 경우)
   - 직접 `Reflect.defineMetadata`를 호출 (복잡한 경우)
3. **관계**: 커스텀 데코레이터는 `Reflect.metadata`를 **활용**하거나 **대체**하는 고수준 추상화
4. **선택 기준**:
   - 단순한 메타데이터만 필요 → `Reflect.metadata` 반환
   - 복잡한 로직 필요 → 직접 구현

이렇게 `@Reflect.metadata`는 foundation이고, 커스텀 데코레이터는 그 위에 구축된 편의 기능입니다!

---

