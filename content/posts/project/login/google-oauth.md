+++
title = '소셜로그인'
date = 2024-10-21T00:20:22+09:00
draft = true
+++
## Google Login
정석: https://www.robinwieruch.de/how-to-roll-your-own-auth/

---
쿠키랑 Authorization은 다름.
- 쿠키: 세션 관리, 사용자 설정 저장 등 `request.headers.cookie;`
- Authorization: API 인증, 토큰 기반 인증 시스템 등 `request.headers.authorization;`

🍎쿠키의 세션을 체크하는 것이므로 barer 토큰은 안쓰고, req.cookies['key']가 중요

---
### API 문서 - HTTP/REST
1. 점진적 승인: code 받기
- GET 요청
```
https://accounts.google.com/o/oauth2/v2/auth?
 scope=https%3A//www.googleapis.com/auth/drive.metadata.readonly&
 access_type=offline&
 include_granted_scopes=true&
 response_type=code&
 state=state_parameter_passthrough_value&
 redirect_uri=https%3A//oauth2.example.com/code&
 client_id=client_id
```

2. 승인 코드를 토큰으로 교환: 액세스토큰 받기
- POST 요청, body는 없고 params url encoded로 정보 전달함. 
- 결과의 `id_token`에 scope에 해당하는 정보 담겨있음. 디코딩.
```
POST /token HTTP/1.1
Host: oauth2.googleapis.com
Content-Type: application/x-www-form-urlencoded

code=4/P7q7W91a-oMsCeLvIaQm6bTrgtp7&
client_id=your_client_id&
client_secret=your_client_secret&
redirect_uri=https%3A//oauth2.example.com/code&
grant_type=authorization_code
```

3. API 호출
```
GET /drive/v2/files HTTP/1.1
Host: www.googleapis.com
Authorization: Bearer access_token
```

---
### oAuth callback을 서버로 보내는 방식
💎 엄밀히 말하면 oauth 콜백을 서버로 보내는 게 맞다.
클라에서 state가 일치한다고 하면 무조건 믿어야하는건데, 서버가 그래서는 안됨.
-> 클라이언트는 서버로 post 요청만 하고 전부 서버에서 컨트롤.

네, 맞습니다. 보안을 강화하기 위해 OAuth 흐름을 서버 측에서 더 많이 제어하는 방식을 설명드렸습니다. 이 방식에서는 다음과 같은 흐름을 따릅니다:

1. 클라이언트 측 시작:
   - 사용자가 "Google로 로그인" 버튼을 클릭합니다.
   - 클라이언트는 서버에 OAuth 시작을 요청합니다 (예: GET /auth/google/start).

2. 서버 측 처리:
   - 서버는 state를 생성하고 쿠키에 저장합니다.
   - Google OAuth URL을 생성합니다 (state 포함).
   - 클라이언트에게 이 URL을 응답으로 보냅니다.

3. 클라이언트 리다이렉션:
   - 클라이언트는 서버로부터 받은 URL로 사용자를 리다이렉트합니다.

4. Google 인증 후 콜백:
   - Google은 서버의 콜백 URL로 리다이렉트합니다.
   - 서버는 state를 검증하고 액세스 토큰을 요청합니다.

이 방식의 장점:
- 클라이언트 ID와 같은 민감한 정보를 클라이언트에 노출하지 않습니다.
- 서버에서 state 검증을 더 안전하게 수행할 수 있습니다.
- CSRF 공격에 대한 보호를 강화할 수 있습니다.

그러나 클라이언트에서 직접 Google OAuth URL로 리다이렉트하는 방식도 여전히 사용됩니다. 이 경우:
- 클라이언트는 Google OAuth URL을 직접 구성하고 리다이렉트합니다.
- 서버는 콜백 처리와 토큰 교환만 담당합니다.

두 방식 모두 유효하며, 보안 요구사항과 애플리케이션 구조에 따라 선택할 수 있습니다.

---
### state 검증에 쿠키 사용
- state 생성 시 HTTP-only, Secure 플래그가 설정된 쿠키에 저장합니다.
- 콜백 시 쿠키에서 state를 읽어 URL 파라미터의 state와 비교합니다.

1. 쿠키의 동작:
   - 쿠키는 도메인에 종속됩니다.
   - Google OAuth 페이지로 리다이렉트되더라도, 원래 도메인(예: yourapp.com)에서 설정한 쿠키는 브라우저에 유지됩니다.

2. 콜백 처리:
   - Google이 사용자를 yourapp.com/auth/callback으로 리다이렉트할 때, 브라우저는 자동으로 yourapp.com 도메인에 대한 쿠키를 포함시킵니다.
   - 이때 이전에 설정한 state 쿠키도 함께 전송됩니다.

3. 서버의 검증 과정:
   - 서버는 콜백 요청을 받을 때 두 가지 정보를 확인합니다:
     a) URL 파라미터로 전달된 state 값
     b) 요청에 포함된 쿠키에서 읽은 state 값
   - 이 두 값을 비교하여 일치하는지 확인합니다.

예시 코드:

```python
@app.route('/auth/google/callback')
def auth_callback():
    # URL 파라미터에서 state 읽기
    url_state = request.args.get('state')
    
    # 쿠키에서 state 읽기
    cookie_state = request.cookies.get('oauth_state')
    
    if not url_state or not cookie_state or url_state != cookie_state:
        return "Invalid state", 400
    
    # state 검증 성공, 계속해서 OAuth 처리
    # ...
```

이렇게 하면 서버는 OAuth 흐름이 동일한 브라우저 세션에서 시작되었음을 확인할 수 있으며, CSRF 공격을 효과적으로 방지할 수 있습니다.

-> 이 경우 클라이언트는 google API client id를 알 필요도 없는데
어쨌든 서버로부터 url을 받으면 client id가 포함되어있어서 노출되긴 함

---
공격자가 client id만 알면 대신 로그인요청을 할수있는거 아닌가?

네, 맞습니다. 공격자가 클라이언트 ID를 알고 있다면 OAuth 로그인 요청을 시작할 수 있습니다. 하지만 이것만으로는 완전한 공격을 수행하기 어렵습니다. 이유는 다음과 같습니다:

1. 리다이렉트 URI 제한:
   - OAuth 설정에서 허용된 리다이렉트 URI만 사용할 수 있습니다.
   - 공격자는 자신의 서버로 인증 결과를 받을 수 없습니다.

2. 클라이언트 시크릿:
   - 액세스 토큰을 얻기 위해서는 클라이언트 시크릿이 필요합니다.
   - 이는 서버 측에서만 안전하게 보관됩니다.

3. State 파라미터:
   - CSRF 공격을 방지하기 위해 사용됩니다.
   - 서버에서 검증하므로 공격자가 임의로 생성한 state는 유효하지 않습니다.
-> 즉 state는 요청의 리다이렉트의 주체가 올바른곳에서 왔다는 것을 검증할 수 있음. 💎

4. 사용자 동의:
   - 사용자는 여전히 로그인하고 권한을 부여해야 합니다.

그러나 공격자가 클라이언트 ID를 사용하여 할 수 있는 잠재적인 위협이 있습니다:

1. 피싱 공격:
   - 진짜처럼 보이는 로그인 페이지를 만들어 사용자를 속일 수 있습니다.

---

쿠키는 웹사이트가 브라우저에 저장하는 작은 텍스트 파일로, 여러 식별자-값으로 구성됨. 
특정 도메인에 대해 요청 헤더에 자동으로 포함됨.
-> 그래서 local.itemscout.io에 여러 키값들이 짬뽕되어있는 것.

쿠키 설정 과정:
1. 서버가 HTTP 응답 헤더에 Set-Cookie를 포함시켜 쿠키 정보를 전송합니다.
2. 브라우저는 이 정보를 받아 로컬에 저장합니다.

🟢브라우저의 자동 포함 원리:
1. 요청 생성: 브라우저가 웹 요청을 준비할 때
2. 쿠키 확인: 저장된 쿠키 중 해당 요청의 도메인, 경로와 일치하는 것을 찾습니다.
3. 헤더 추가: 일치하는 쿠키를 요청의 Cookie 헤더에 추가합니다.
4. 전송: 쿠키가 포함된 요청을 서버로 전송합니다.

예시 설명:
'https://example.com에서 설정된 쿠키는 https://example.com으로의 요청에 포함됩니다.'

1. 쿠키 설정:
   - example.com 서버가 응답에 Set-Cookie 헤더를 포함:
     ```
     Set-Cookie: user=john; Domain=example.com; Path=/; Secure
     ```

2. 브라우저 저장:
   - 브라우저는 이 쿠키 정보를 로컬에 저장합니다.

3. 후속 요청:
   - 사용자가 https://example.com/page로 접속하려 할 때
   - 브라우저는 저장된 쿠키 중 도메인(example.com)과 경로(/)가 일치하는 것을 찾습니다.
   - 일치하는 쿠키를 요청 헤더에 추가:
     ```
     Cookie: user=john
     ```

4. 서버 수신:
   - example.com 서버는 요청에 포함된 쿠키를 읽어 사용자를 식별할 수 있습니다.

이 과정은 브라우저에 의해 자동으로 처리되며, 개발자가 직접 관여할 필요가 없습니다. 이러한 메커니즘 덕분에 웹사이트는 상태를 유지하고 사용자를 식별할 수 있게 됩니다.

---
### Credentials
CORS는 크로스 오리진 요청에 대해 쿠키를 포함하지 않는다. 
CORS는 서버, 클라가 요청에 쿠키를 포함해되 되는지를 인지할 것을 요구함으로써 
쿠키를 active decision으로 만든다. 
클라는 credential 프로퍼티를 true로 설정함으로써 쿠키포함을 허용.
서버도 반드시 Access-Control-Allow-Credentials 헤더에 넣어야
서버가 쿠키가 포함되는걸 허용함을 의미. 

The spec doesn't require a pre-flight (additional roundtrip to check if the server will allow credentials) for GET requests. -> GET요청은 pre flight 필수 아님.
브라우저는 프리 플라이트 대신 항상 요청을 수행하여 withCredentials가 설정된 경우 쿠키를 전송하지만, 
withCredentials가 설정된 경우 응답을 수신할 때  응답에 Access-Control-Allow-Credentials 헤더가 설정된 경우에만 결과를 호출 자바스크립트에 전달/노출합니다. 헤더가 없으면 응답을 노출하지 않음

결국 브라우저가 컨트롤하는 것임.
브라우저는 CORS 정책을 통해 자동으로 쿠키를 포함하지 않으며, 명시적으로 설정된 경우에만 포함하는 것이고,
credential 옵션도 마찬가지로 브라우저가 판단함. 

네, 맞습니다. 응답을 호출하는 JavaScript에 노출할지 결정하는 주체는 **브라우저**입니다. 브라우저는 다음과 같은 방식으로 동작합니다:

1. **요청 전송**: `withCredentials` 옵션이 설정된 경우, 브라우저는 쿠키를 포함하여 요청을 전송합니다.

2. **응답 수신**: 서버로부터 응답을 받으면, 브라우저는 응답 헤더를 검사합니다.

3. **헤더 확인**: 응답에 `Access-Control-Allow-Credentials: true` 헤더가 있는지 확인합니다.

4. **노출 결정**:
   - 헤더가 존재하면, 브라우저는 응답 데이터를 JavaScript 코드에 노출합니다.
   - 헤더가 없으면, 브라우저는 보안상의 이유로 응답을 JavaScript에 노출하지 않습니다.

이 과정은 브라우저가 CORS 정책을 준수하여 보안을 강화하기 위한 메커니즘입니다. 이를 통해 브라우저는 서버가 명시적으로 허용하지 않는 한, 민감한 데이터를 포함한 응답을 클라이언트 측 코드에 노출하지 않도록 합니다.

Q. 굳이 클라에서 설정을 하면 서버에서도 저 옵션을 넣어야하는데.. CORS만으로 충분하지 않나?

CORS는 출처를 제어하지만, 자격 증명(쿠키 등)을 포함할지 여부는 별도의 보안 문제입니다. 
- CORS: 허용된 출처를 명시
- credentials include: 요청에 쿠키나 인증정보를 포함할지 결정. 민감한 정보.

---

