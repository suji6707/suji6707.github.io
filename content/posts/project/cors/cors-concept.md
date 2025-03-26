+++
categories = ['Web']
tags = ['CORS']
title = 'CORS 에러 원인'
date = 2024-04-06T22:50:44+09:00
draft = true
+++
## CORS
Cross Origin Resource Sharing이란 다른 출처간 요청을 허용하는 것.

Same Origin Policy(SOP): 원래 브라우저는 요청 주소가 origin과 다르면 이상하다고 판단해서 응답을 막음. (악의적인 사이트로 요청 등 보안)
그런데 서버로부터의 응답 헤더에 Access-control-allow-origin에 현재 요청 origin이 포함되어있으면 안전한 것으로 판단해 허용.
서버 입장에서 Access-control-allow-origin은 '아니야 괜찮아' 정도가 아니라 괜찮다고 도장까지 찍어주는 거라 보면 됨.
-> CORS는 응답 헤더에 Access-control-allow-origin: 'localhost:3000'을 넣어줌으로써 3000포트에서 8000포트 서버로 요청을 브라우저에서 허용하도록 해주는 것.
참고로 브라우저 Origin을 바꿀 수는 없으므로 서버에서 응답헤더에 Access Allow 도메인을 넣어줌으로써 CORS 에러를 해결해야 함.

중요한건 동일출처 정책은 브라우저에서 막는다는 것.
(그래서 curl 요청시 브라우저를 거치지 않으므로 CORS 에러 X)
서버 입장에선 브라우저를 거치지 않는 악의적인 요청은 다른 방법으로 막아야 함.

주소창에 바로 localhost:8000 서버 주소를 치면 
Origin이 8000인 것으로 간주되기 때문에 
8000 -> 8000 Same origin 요청이 되어 허용됨.

그리고 브라우저에서 현재 origin과 같은 출처로 요청이 가면 
자체적으로 판단해 Response header에 Access-control-allow-origin가 뜨지도 않는다.



![image](https://github.com/suji6707/suji6707.github.io/assets/111227732/042ed50c-8fa2-4a6f-aba2-1fc01698caea)

---
#### Same-origin policy

다른 출처를 읽을 수 없어도 쓰기는 제한적으로 허용됨. 웹의 기본적인 기능과 하위 호환성을 위한 것입니다.
🔴🔴어차피 내가 다른사이트에 쓸 수 있어도 받는 사이트가 cors로 나를 열어놓지 않으면 못 읽음. write가 소용없음

---
#### CSRF (cross site request forgery)
🔴적어도 로그인은 CSRF 토큰 생성해서 검증 해야함

## **1. CSRF가 어떻게 작동하는지 이해하기**
CSRF 공격은 다음과 같은 방식으로 작동합니다:
1. **사용자가 로그인 상태임.** (예: 은행 웹사이트에 로그인되어 있고, 인증 쿠키가 브라우저에 저장됨)
2. **사용자가 악의적인 웹사이트를 방문함.**
   - 이 웹사이트는 `<form>` 태그 또는 JavaScript 등을 이용해 은행으로 특정 요청(예: 송금 요청)을 보냄.
   - 브라우저는 동일한 사용자 세션(쿠키)을 이용해 요청을 보냄. 브라우저는 요청 자체가 신뢰할 수 없는 출처로부터 온 것인지 알 수 없음.
3. 결과적으로 공격자는 사용자를 속여 그의 인증 정보를 이용해 자신이 원하는 요청을 수행하게 함.

보안상의 문제는 **브라우저가 쿠키를 자동으로 전송하는 점**에서 발생합니다. 즉, 브라우저는 요청이 사용자로부터 온 것인지, 공격자로부터 조작된 것인지 구분하지 않습니다.

🟢🟢🟢 중요한건 내가 은행에 로그인된 상태라면
브라우저가 동일 사이트에는 해당 쿠키를 자동으로 보낸다는 것. 
그걸 못보내게 하는게 cookie lax = `strict`

---
## **2. CSRF 토큰이 방어할 수 있는 이유**

CSRF 토큰은 CSRF 공격으로부터 방어할 수 있는 이유는 다음과 같습니다:

### **(1) CSRF 토큰은 고유하고, 사용자 세션에 바인딩됨**
- 서버는 사용자가 페이지를 요청할 때, 사용자 계정 및 세션과 연관된 고유한 CSRF 토큰을 생성합니다.
  - 예: 사용자가 "송금 페이지"를 열 때 서버는 CSRF 토큰을 함께 응답으로 보냅니다.
  - CSRF 토큰은 HTML 폼에 숨겨진 필드로 포함되거나, JavaScript 요청의 헤더에 포함되어 클라이언트로 전달됩니다.
- CSRF 토큰은 서버 측에서 유효성을 검증할 수 있습니다.
  - 예: 서명(Signature), 세션 ID, 고유한 해시 값 등을 포함.

---
### CSRF 토큰 예시코드

TypeScript를 사용하며 CSRF 토큰을 사용하는 기본적인 예제를 제공하겠습니다. 이 예제는 백엔드와 프론트엔드 두 부분으로 나뉘어 CSRF 토큰을 어떻게 생성하고 검증하는지 보여줍니다. 아래에서는 Express(백엔드)와 Fetch API(프론트엔드)를 사용하는 간단한 구조를 예로 들겠습니다.

## **1. 백엔드: CSRF 토큰 생성 및 검증**

```typescript
import express from "express";
import cookieParser from "cookie-parser";
import crypto from "crypto";

const app = express();
const PORT = 3000;

app.use(cookieParser());
app.use(express.json());

// 저장소 (간단한 예제를 위해 메모리를 사용합니다)
const users: { [userId: string]: string } = {}; // userId -> CSRF 토큰 매핑

// 유틸 함수: CSRF 토큰 생성
function generateCsrfToken(): string {
  return crypto.randomBytes(32).toString("hex");
}

// 라우트 1: 로그인 후 유저 세션과 함께 CSRF 토큰 발급
app.post("/login", (req, res) => {
  const { userId } = req.body;

  if (!userId) {
    return res.status(400).send("User ID is required");
  }

  const csrfToken = generateCsrfToken();
  users[userId] = csrfToken;

  // CSRF 토큰을 쿠키에 저장 (보안 목적으로 HttpOnly를 설정하지 않음 - 브라우저에서 접근 가능하도록)
  res.cookie("X-CSRF-TOKEN", csrfToken, { httpOnly: false, sameSite: "Strict" });

  res.send({ message: "Login successful", csrfToken });
});

// 라우트 2: 민감한 데이터에 대한 POST 요청 (토큰 검증)
app.post("/transfer-money", (req, res) => {
  const { userId, amount } = req.body; // 송금 요청 시 사용자 ID와 금액 등 데이터
  const csrfToken = req.headers["x-csrf-token"]; // 클라이언트가 전송하는 CSRF 토큰

  if (!userId || !amount) {
    return res.status(400).send("Missing userId or amount");
  }

  if (!csrfToken || users[userId] !== csrfToken) {
    return res.status(403).send("Invalid CSRF token");
  }

  // 처리 성공
  res.send({ message: "Money transfer successful!" });
});

// 서버 시작
app.listen(PORT, () => {
  console.log(`Server running on http://localhost:${PORT}`);
});
```

---

## **2. 프론트엔드: CSRF 토큰을 포함한 요청**

### **시나리오**
1. 사용자가 서버에 로그인 요청을 보냅니다.
2. 서버는 CSRF 토큰을 쿠키로 전달합니다.
3. 프론트엔드는 요청을 보낼 때 헤더에 CSRF 토큰을 포함합니다.

### **프론트엔드 코드**

```typescript
// 로그인 함수: 서버에서 CSRF 토큰을 받아옵니다.
async function login(userId: string) {
  const response = await fetch("http://localhost:3000/login", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({ userId })
  });

  if (response.ok) {
    const data = await response.json();
    console.log("Login successful:", data);
  } else {
    console.error("Login failed:", await response.text());
  }
}

// 민감한 요청 함수: 헤더에 CSRF 토큰 포함
async function transferMoney(userId: string, amount: number) {
  // 쿠키에서 CSRF 토큰 읽기
  const csrfToken = getCsrfTokenFromCookie();

  if (!csrfToken) {
    console.error("CSRF token is missing");
    return;
  }

  const response = await fetch("http://localhost:3000/transfer-money", {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
      "X-CSRF-TOKEN": csrfToken, // 헤더에 CSRF 토큰 포함
    },
    body: JSON.stringify({ userId, amount }),
  });

  if (response.ok) {
    const data = await response.json();
    console.log("Transfer successful:", data);
  } else {
    console.error("Transfer failed:", await response.text());
  }
}

// 유틸 함수: 쿠키에서 CSRF 토큰 읽기
function getCsrfTokenFromCookie() {
  const cookies = document.cookie.split("; ").reduce((acc, cookie) => {
    const [key, value] = cookie.split("=");
    acc[key] = value;
    return acc;
  }, {} as { [key: string]: string });

  return cookies["X-CSRF-TOKEN"];
}

// 실행 코드
(async () => {
  await login("user123"); // 사용자 로그인
  await transferMoney("user123", 100); // 송금 요청
})();
```


## **3. 코드의 작동 원리**

1. **로그인 요청:**
   - 사용자가 로그인하면 서버는 사용자에게 고유한 CSRF 토큰을 쿠키로 전달합니다.
   - 이 쿠키는 `document.cookie`로 프론트엔드에서 접근할 수 있습니다.

2. **송금 요청:**
   - 프론트엔드에서는 쿠키에서 CSRF 토큰을 읽어 요청 헤더(`X-CSRF-TOKEN`)에 포함합니다.
   - 서버는 요청 헤더로 전달된 CSRF 토큰과 서버에 저장된 토큰을 비교하여 유효성을 검증합니다.
   - 검증에 실패하면 요청을 차단합니다.


## **4. 보안 고려 사항**
- **HTTPS 사용:** 쿠키와 CSRF 토큰은 HTTPS로 암호화된 채널을 통해 전달되어야 안전합니다.
- **`SameSite=strict`:** 쿠키에 `SameSite` 속성을 사용하여 CSRF 토큰 쿠키가 크로스사이트 요청과 함께 전송되지 않도록 설정합니다.
- **세션 연동:** CSRF 토큰은 사용자의 세션과 연관되어야 합니다.
- **토큰 로테이션:** 토큰이 주기적으로 재생성되도록 하여 재사용 및 도난을 방지합니다.


위 코드는 TypeScript 환경에서 CSRF 토큰을 사용하는 기본적인 예제를 보여줍니다. 실-world에서는 CSRF 보호를 제공하는 라이브러리(예: `csurf` for Express)를 고려하여 보안을 추가로 강화할 수 있습니다! 😊

---
#### 🟢 Q. SameSite=Lax 또는 Strict를 설정하면 브라우저는 크로스 사이트 요청에서 이 쿠키를 전송하지 않아 CSRF 방어에 도움을 줍니다.
이걸로 방지할 수 있는 상황 예를 들어줘.
그리고 csrf, cors도 예시.
내가 이메일에서 악의적인 링크를 클릭한경우 어떻게 위조 요청을 할수있는지,
내가 은행사이트에 일상적으로 들어왔는데 공격자가 내 쿠키를 가지고 뭔가를 할수있는건지 등

좋은 질문입니다! 지금부터 질문 하나하나에 대해 구체적인 예시를 들어가며 설명하겠습니다. **SameSite=Lax 및 Strict 설정의 효과, CSRF와 CORS의 역할, 그리고 공격 가능한 시나리오와 방어 방법**까지 전반적으로 다뤄보겠습니다.

---

## **1. `SameSite=Lax` 또는 `Strict` 설정 효과로 방지할 수 있는 상황**

### **SameSite 쿠키란?**
`SameSite` 속성은 쿠키가 크로스 사이트 요청(다른 출처에서 발생한 요청)에 포함될 수 있는지 여부를 제어합니다. 이 속성을 적절히 설정하면 CSRF 공격과 같은 위조 요청을 방지할 수 있습니다.

### **속성 설명:**
- **SameSite=None:** 크로스 사이트 요청에서도 쿠키가 전송됩니다. (기본값, 예전 브라우저에서 흔함)
- **SameSite=Lax:** 기본적으로 크로스 사이트 요청에서는 쿠키를 전송하지 않지만, **사용자가 직접 링크를 클릭해서 발생한 GET 요청은 예외적으로 허용**합니다.
  - 사용하는 경우: 로그인 상태를 유지하면서도 제한적인 크로스 사이트 동작을 허용.
- **SameSite=Strict:** 크로스 사이트 요청에서는 쿠키를 전혀 전송하지 않습니다. 사용자가 직접 링크를 클릭해도 전송하지 않습니다.
  - 사용하는 경우: 크로스 사이트 동작을 전혀 허용하지 않는 강력한 보안 정책.

### **효과적인 시나리오 예시**

#### **SameSite=Lax로 방지할 수 있는 상황**
- 은행 웹사이트 `https://bank.com`에 로그인된 사용자가 악성 사이트 `https://evil-website.com`를 방문합니다.
- 악성 사이트가 다음과 같은 HTML 폼을 자동으로 제출합니다:

```html
<form action="https://bank.com/transfer-money" method="POST">
    <input type="hidden" name="amount" value="1000000">
    <input type="hidden" name="toAccount" value="hacker123">
</form>
<script>
    document.forms[0].submit(); // 자동적으로 폼 제출
</script>
```

- 기본적으로 브라우저는 동일한 출처에 대해 사용자의 인증 쿠키(`session`, 또는 인증 토큰)가 전송되므로, 요청이 은행 시스템에서 합법적인 요청처럼 보이지만 사용자는 전혀 요청을 의도하지 않았습니다. (이것이 CSRF 공격.)

**SameSite=Lax를 설정 시:**
- 위와 같은 POST 요청은 크로스 사이트 출처에서 발생했기 때문에 쿠키가 전송되지 않음 → 세션 인증 정보가 없으므로 요청이 거부됩니다.

#### **SameSite=Strict로 더 강력히 방지할 수 있는 상황**
- 사용자가 악성 링크 (`https://evil-website.com`)를 직접 클릭한 후 GET 요청으로 `https://bank.com/profile`에 접근하여 쿠키 기반 개인 정보를 조회하려는 경우.
- **Lax**에서는 사용자가 직접 클릭한 요청(GET)에 쿠키를 전송하지만,
🔴🔴**Strict**에서는 이 경우에도 쿠키를 전송하지 않습니다. 따라서 더욱 강력한 방어를 제공합니다.

---

## **2. CSRF와 CORS가 있어도 위조 요청이 가능한지**

CSRF와 CORS는 각각 다른 문제를 해결하려는 메커니즘입니다. 두 개 모두 있어도, 특정 조건에서는 위조 요청이 이루어질 가능성이 존재합니다. 아래에서 **이메일의 악의적인 링크를 클릭하는 사례**를 통해 설명하겠습니다:

### **시나리오: 이메일에서 악성 링크를 클릭한 경우**

1. **CSRF 공격으로 위조 요청 실행**
   - 사용자는 `https://bank.com`에 로그인한 상태입니다. 브라우저는 해당 로그인 세션을 인증하는 쿠키를 가지고 있습니다.
   - 공격자가 이메일에 악성 링크를 포함시킵니다:
     ```
     <a href="https://bank.com/transfer-money?amount=1000000&to=hacker">Click Here!</a>
     ```
   - 사용자가 이메일을 읽는 중 링크를 클릭합니다. `SameSite=Lax`가 없는 경우, 브라우저는 자동으로 `bank.com`의 인증 쿠키를 전송하며, 요청이 실행됩니다.
   - 결과적으로 공격자는 피해자의 승인을 받지 않고도 송금을 구현합니다.

   **방지 방법:**
   - `SameSite=Lax` 또는 `Strict` 설정을 통해 크로스 사이트 요청에서 인증 쿠키를 전송하지 않도록 설정하면, 위 요청은 실행되지 않습니다.

2. **CORS와 CSRF는 서로 다른 역할**
   - CSRF는 위조된 요청을 보낸 후 서버가 이를 실행하도록 유도합니다.
   - CORS는 응답 데이터의 읽기를 방지합니다. 예를 들어:
     ```javascript
     fetch("https://bank.com/transfer-money?amount=1000000&to=hacker");
     ```
     - 요청은 성공적으로 실행되더라도, **CORS 규칙에 따라** 공격자는 해당 요청 결과를 읽을 수 없습니다.
     - 그러나 CSRF 토큰 검증 없이 실행된 요청은 여전히 송금과 같은 영향을 끼칠 수 있습니다.

---

## **3. 공격자가 쿠키로 무엇을 할 수 있는가?**

공격자가 쿠키를 얻는 방법에 따라 공격 가능성은 다릅니다:

### **1) 쿠키를 직접 읽을 수 있다면? (XSS 공격의 경우)**
- 공격자가 브라우저에서 `document.cookie`를 통해 사용자의 쿠키를 읽는 상황이라면:
  - **세션 인증 쿠키 탈취:** 공격자가 피해자의 로그인 세션을 탈취할 수 있습니다.
  - **CSRF 토큰 탈취:** 탈취한 CSRF 토큰을 사용해 CSRF 방어를 우회하고 위조 요청을 실행할 수 있습니다.

   **방지 방법:**
   - 쿠키에 `HttpOnly` 플래그를 설정하여 클라이언트 측 🔴🔴JavaScript에서 쿠키를 읽지 못하도록 합니다.
   - XSS를 방지하려면:
     - 사용자 입력 철저 검증 (예: 입력값 이스케이프)
     - Content Security Policy(CSP) 설정 (`script-src` 제한)

### **2) 공격자가 브라우저를 속여 쿠키를 사용하게 만든다면?**
- **CSRF 시나리오:** 공격자가 위조된 요청을 만들어 사용자의 쿠키를 기반으로 요청을 실행하면, 실제로 같은 세션 인증 값이 서버로 전송될 수 있습니다.
- 방지 방법:
  - `SameSite=Lax` 또는 `Strict`를 사용해 쿠키를 크로스 사이트 요청에서 전송하지 못하도록 합니다.
  - CSRF 토큰을 추가적으로 사용하여 요청 출처를 검증하십시오.

### **3) 쿠키 내 민감한 데이터 열람**
- 공격자가 악성 스크립트를 삽입하거나 네트워크를 가로채서 HTTPS가 없는 경우 민감한 데이터(예: 비밀번호, 계정 정보)를 노출시킬 수 있습니다.
- **방지 방법:**
  - HTTPS만 사용하고, `Secure` 플래그로 쿠키 전송을 제한합니다.

---

## **4. 정리: 모든 요소를 조합해 완벽한 방어 구축**

CSRF와 CORS는 서로 다른 문제를 해결하며, 추가적으로 `SameSite` 옵션과 XSS 방어 메커니즘을 조합해 실행해야 합니다.

### **이메일 링크를 통한 공격 방지**
1. CSRF: `SameSite=Lax` 또는 `Strict`를 사용해 크로스 사이트 쿠키 전송 차단.
2. CSRF 토큰: 추가적으로 각 요청에 고유 CSRF 토큰 검증.
3. HTTPS: 보안이 강화된 HTTPS 채널 사용.
4. CORS: 응답 데이터를 외부에서 읽지 못하도록 차단.

### **쿠키 탈취 공격 방지**
1. XSS 방어: 철저한 입력 검증 및 CSP 설정.
2. 쿠키 속성:
   - `HttpOnly`: JS에서 쿠키 접근 막기.
   - `Secure`: HTTPS 연결에서만 전송.
   - `SameSite`: 크로스 사이트 요청 제한.

---

결론적으로, 보안은 하나의 방어 메커니즘으로 충분하지 않습니다. CSRF, CORS, SameSite, XSS 방지를 조합하여 다층적인 방어를 갖추는 것이 중요합니다. 궁금한 점이 더 있으면 추가로 알려주세요! 😊

---