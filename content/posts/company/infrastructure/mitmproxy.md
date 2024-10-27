+++
categories = ['Proxy']
series = ['Proxy']
date = 2024-09-23T22:50:44+09:00
draft = true
+++
## Mimproxy

### 흥미로운 사례
🍎 Arc- 다른 누군가의 브라우저에 그들 몰래, 어떤 웹사이트에 접속한다

요청이 안간다 -> firebase (database as a backend service)
mitmproxy를 써서 인터셉터(요청 가로채기) 해보니 
```
firebase
  .collection("boosts")
  .where("creatorID", "==", "UvMIUnuxJ2h0E47fmZPpHLisHn12");
```
를 발견.

create boost를 하면 내 creatorID를 알 수 있고
해당 creatorID에 대한 정보들 중
특정 웹사이트에 대한 커스텀이 가능하다- 

웹사이트 커스터마이징: 사용자는 특정 웹사이트의 특정 요소를 차단하거나 글꼴, 색상을 변경
-> 피해자의 createorID로 특정 웹사이트에 대한 악성 boost 생성
-> 커스텀에 악의적인 JS 코드를 적용

- boost 기능 비활성화
- Firebase ACL 규칙 검토
- Firebase에서 다른 제품으로 개발

--
firestore는 백엔드를 작성하지 않아도 되도록 해주는 데이터베이스-백엔드 서비스로, 개발자가 데이터베이스 보안 규칙을 작성하고 사용자가 직접 데이터베이스에 접근할 수 있게 합니다
Frida라는 도구를 사용해서 iOS 앱에서 실행되는 Firestore 데이터베이스 질의를 가로채고 기록
Firestore를 단순히 인증 용도로만 사용하는 것이 아니라 앱의 중요 데이터를 저장

---

### 특정 웹사이트의 이미지를 차단하는 방법

**웹사이트 명시:**

특정 웹사이트를 명시하는 부분은 `response` 함수 내에서 `flow.request.pretty_url`을 체크하는 조건문입니다. 이 조건문은 특정 웹사이트의 URL을 포함하는지 확인합니다.

```python
if "example.com" in flow.request.pretty_url:
```

위 코드에서 `"example.com"` 부분을 원하는 웹사이트의 도메인으로 변경하면 됩니다. 예를 들어, google.com의 이미지를 차단하려면 `"google.com"`으로 변경합니다.

### 서버를 계속 켜둬야 하는지에 대한 질문

**mitmproxy 운영:**

mitmproxy는 클라이언트와 서버 사이에 위치한 프록시 서버로 작동합니다. 따라서 mitmproxy를 사용하여 트래픽을 가로채고 조작하려면 해당 프록시 서버가 항상 실행 중이어야 합니다. 클라이언트(예: 웹 브라우저)는 mitmproxy를 통해 모든 요청을 보내고, mitmproxy는 요청과 응답을 가로채고 조작한 후 클라이언트와 서버 간의 통신을 중계합니다.

### 예시 코드 다시 한 번 설명

```python
from mitmproxy import http

def response(flow: http.HTTPFlow) -> None:
  """이미지 응답을 차단하는 함수"""
  # 요청 URL이 특정 웹사이트를 포함하는 경우
  if "example.com" in flow.request.pretty_url:
    # Content-Type 헤더가 이미지인 경우
    if flow.response.headers.get("content-type", "").startswith("image"):
      # 응답 내용을 비웁니다.
      flow.response.content = b""
      # 차단 메시지를 콘솔에 출력합니다.
      print(f"이미지 차단: {flow.request.pretty_url}")

# mitmproxy 스크립트 실행 시 자동으로 호출되는 함수
# addons을 통해 mitmproxy에 기능을 추가할 수 있습니다.
addons = [
    response,
]
```

### 실행 방법

1. 위 코드를 `block_images.py` 파일로 저장합니다.
2. 터미널에서 다음 명령어를 실행하여 mitmproxy를 시작합니다.

```bash
mitmdump -s block_images.py
```

3. 이제 클라이언트(예: 웹 브라우저)를 mitmproxy를 통해 트래픽을 라우팅하도록 설정합니다. 이를 위해 브라우저의 프록시 설정을 변경하거나, 운영 체제의 네트워크 설정에서 프록시를 지정할 수 있습니다.

### 요약

- 특정 웹사이트를 명시하려면 `flow.request.pretty_url` 조건문을 사용하여 URL을 필터링합니다.
- mitmproxy는 클라이언트와 서버 사이에 위치한 프록시 서버로 작동하며, 지속적으로 실행되어야 합니다.
- mitmproxy를 통해 웹 트래픽을 라우팅하면 특정 웹사이트의 이미지를 차단할 수 있습니다.

이와 같이 mitmproxy를 사용하면 다양한 방식으로 웹 트래픽을 조작할 수 있습니다. 더 궁금한 점이 있다면 언제든지 물어보세요! 😊


---

네, 맞습니다. 로컬 PC에서 mitmproxy를 실행하고 별도의 설정을 하지 않으면 **본인 PC의 브라우저에서 발생하는 HTTP/HTTPS 트래픽만** 가로채서 조작할 수 있습니다. 

다른 웹사이트 요청은 일반적으로 사용자의 PC에서 바로 해당 웹사이트 서버로 전송되기 때문에, 로컬 PC에서 실행 중인 mitmproxy를 거치지 않습니다. 

mitmproxy를 사용하여 특정 웹사이트의 이미지를 차단하려면, **해당 웹사이트 트래픽이 mitmproxy를 거치도록 설정**해야 합니다. 

이는 일반적으로 다음 두 가지 방법 중 하나를 사용하여 수행합니다.

1. **브라우저 프록시 설정:** 사용하려는 브라우저의 설정에서 **수동 프록시 설정**을 활성화하고, **로컬 PC의 IP 주소와 mitmproxy가 사용하는 포트 번호(기본값: 8080)**를 입력합니다. 이렇게 하면 해당 브라우저에서 발생하는 모든 웹 트래픽이 mitmproxy를 거치게 됩니다.

2. **시스템 전체 프록시 설정:** 운영 체제 (Windows, macOS, Linux)의 네트워크 설정에서 **시스템 전체 프록시**를 설정할 수 있습니다. 이렇게 하면 해당 PC에서 실행되는 모든 애플리케이션의 웹 트래픽이 mitmproxy를 거치게 됩니다.

**주의 사항:**

* mitmproxy를 사용하여 HTTPS 트래픽을 가로채려면 **mitmproxy에서 제공하는 SSL 인증서을 브라우저에 설치**해야 합니다. 자세한 내용은 mitmproxy 공식 문서를 참조하세요.
* 다른 사람의 PC에서 발생하는 트래픽을 가로채는 것은 **불법 행위**입니다. mitmproxy는 개발 및 테스트 목적으로만 사용해야 합니다.

결론적으로, 로컬 PC에서 mitmproxy를 실행하면 기본적으로 본인 PC의 브라우저에서 발생하는 트래픽만 가로챌 수 있습니다. 다른 웹사이트의 트래픽을 가로채려면 브라우저 또는 시스템 전체 프록시 설정을 변경하여 트래픽이 mitmproxy를 거치도록 해야 합니다. 
