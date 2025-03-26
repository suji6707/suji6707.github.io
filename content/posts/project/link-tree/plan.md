+++
title = 'LinkTree'
date = 2024-10-08T00:20:22+09:00
draft = true
+++
## 링크트리 clone

💎💎💎💎 LINK HUB 💎💎💎💎
프로필 여러개 OK.
프로필 안에 링크 아카이빙한다고 보면 됨.
좌우 스와이프는 X.
기본적으로 아래로 스크롤임.

우선은 유저가 수동으로 링크(특정 사이트/소셜계정의 포스트)를 넣으면 업로드되지만
나중에는 피드형태로 업데이트, 알람이 오면 내가 그걸 소비하는 방식도 좋을듯.
조금 쓸쓸한 SNS? but 여기에 AI로 요약을 넣어준다던가.

기본적으로는 링크를 넣으면 TLDR 요약글처럼 보여지는 것이면 좋겠음.

한 프로필내 여러 아카이빙 그룹이 있고
그 그룹내 여러 링크들.



@@@TODO Linkhub
Authguard 를 하나더 만들어서 항상 true 반환, 
req.loginStatus 같은거 붙여서 주기

new AugthGuard(인자) 활용해도 좋음.

🔴 다른 AuthGuard 하나 더. 항상 true하되 쿠키있으면 req.user 넣기. 없으면 edit/wrench 없으면 됨.

🔴 onMounted로 async 함수들 옮김. 어차피 인스턴스생성이 훨씬 빨라서.

onMounted(() => {
  fetchLoginUser();
  fetchViewedUser();
});
watch -> 받아온 값을 가지고 연산.

🔴 storeToRefs



이건 블로그 페이지의 content인데, 이걸 한국어로 요약해줘.
script나 style 태그가 있다면 무시하고,
본문 외에 댓글이나 광고글, 글목록 리스트 등이 있다면 무시해줘


---

typeorm / prisma. utc / drizzle

컨셉 생각해보기. o1

한 유저가 여러개의 프로필을 가질 수 있고
내 계정뿐 아니라 다른 사람의 소셜 링크들을 넣을 수 있다.


#### phase 1 (2주내)
- 화면구성 기획 draw.io (SaaS를 보는눈!)
	- 직접 써보면서. 내 깃헙주소 등
- 데이터베이스 설계 & 마이그레이션 작성
- 아주 간단한 로그인 API -> 나중에 소셜로?
	- 여러 유저가 있을 수 있는 환경만 만들어놓으려함.
- 큰 틀에서의 클라이언트 화면
	- 어떤 API가 필요한지 더 구체화시키기 위해

#### phase 2
- 링크 CRUD API
- 이미지 업로드 API

#### phase 3
- 소셜로그인으로 변경 (oAuth2)
- QR코드 생성
- URL Shortner 활용해서 마케팅용 링크 생성

#### 해보면 좋을 것
깃허브 액션 + 테스트(how e2e & 최소한의 유닛테스트)

#### 인프라
오라클 4코어, 도커로 배포?

---
### 진행순서

프로젝트 진행 순서를 잘 정리하면 효율적이고 체계적으로 작업을 완료할 수 있습니다. 아래는 위의 작업들을 수행하는 데 있어 추천하는 순서와 그 이유입니다.

### 1. 프로젝트 초기화 및 환경 설정
- **백엔드와 프론트엔드 모두**
  - **이유**: 프로젝트의 기본적인 구조를 먼저 설정함으로써 이후 작업들이 원활하게 진행될 수 있도록 합니다. 개발 환경을 미리 설정해두면 코드 작성과 디버깅이 더 용이해집니다.

### 2. 데이터베이스 설계 및 구현
- **백엔드**
  - **이유**: 데이터베이스가 제대로 설계되지 않으면 이후의 기능 구현이 어렵거나 비효율적일 수 있습니다. 유저와 링크 데이터를 어떻게 저장하고 관리할지 미리 정의해야 합니다.

### 3. 유저 인증 및 관리 기능 구현
- **백엔드 먼저, 프론트엔드 나중에**
  - **이유**: 유저 인증은 대부분의 기능의 기초가 되기 때문에 먼저 구현해야 합니다. 백엔드에서 인증 API를 먼저 구현한 후 프론트엔드에서 이를 이용한 로그인 및 회원가입 기능을 추가합니다.

### 4. 링크 관리 API 및 프론트엔드 UI 구현
- **백엔드 먼저, 프론트엔드 나중에**
  - **이유**: 백엔드에서 링크를 추가, 수정, 삭제하는 API를 먼저 구현한 후, 프론트엔드에서 이를 호출하여 UI를 구성합니다. 이렇게 하면 프론트엔드에서 백엔드 API를 실제로 테스트하면서 개발할 수 있습니다.

### 5. 이미지 업로드 기능 구현
- **백엔드 먼저, 프론트엔드 나중에**
  - **이유**: 이미지 업로드는 서버에서 처리되어야 하는 작업이므로, 백엔드에서 이를 처리할 API를 먼저 구현하고 프론트엔드에서 이를 호출하여 이미지를 업로드하고 미리보기를 표시합니다.

### 6. 유저 페이지 라우팅 및 데이터 연동
- **백엔드와 프론트엔드 병행**
  - **이유**: 유저별 링크 페이지를 동적으로 생성하는 기능은 백엔드와 프론트엔드가 모두 필요합니다. 백엔드에서 유저 데이터를 제공하고, 프론트엔드에서 이를 받아서 페이지를 렌더링합니다.

### 7. 테스트 작성 및 배포
- **백엔드와 프론트엔드 모두**
  - **이유**: 모든 기능이 구현되면, 유닛 테스트와 통합 테스트를 작성하여 코드의 안정성을 확인합니다. 이후 최종적으로 배포를 진행합니다.

### 전체 작업 순서 요약
1. **프로젝트 초기화 및 환경 설정**
2. **데이터베이스 설계 및 구현**
3. **유저 인증 및 관리 기능 구현**
   - 백엔드 -> 프론트엔드
4. **링크 관리 API 및 프론트엔드 UI 구현**
   - 백엔드 -> 프론트엔드
5. **이미지 업로드 기능 구현**
   - 백엔드 -> 프론트엔드
6. **유저 페이지 라우팅 및 데이터 연동**
   - 백엔드와 프론트엔드 병행
7. **테스트 작성 및 배포**

이 순서를 따르면 프로젝트가 체계적으로 진행될 수 있으며, 각 단계에서 발생할 수 있는 문제를 최소화할 수 있습니다. 성공적인 프로젝트 진행을 기원합니다!

---

### 회원가입/로그인
- passport 쓰기. 
  - 💎이메일 인증 후 개인정보 입력.
  - 일반로그인, 구글로그인 둘다 붙일 때 어떻게 해야할지? 
  - oAuth2는 youtube data API 에 하다만거 있음.
  - Nest 컨벤션 따라가보기(책 참조!).
  - 세션 토큰 있어야함.
    - 늘 하던거(session 테이블 - 헤더 userId)는 당연 하겠지만
    - 쿠키에 다른 정보도 넣어보고
    - 무엇보다 미들웨어. 세션체커만 하면 되겠지만 구현 예시들 찾아보기.
      - session checker
        - device check
      - web checker
  - 가입 동의화면 붙여보기.


---

### Spotify 참조

`SameSite=None`은 쿠키가 크로스 사이트 요청에서도 전송될 수 있도록 허용하는 설정입니다. 이 설정은 주로 다음과 같은 경우에 사용됩니다:

1. **서드파티 서비스 통합**: 
   - 예를 들어, 사용자가 다른 웹사이트에 임베된 Spotify 플레이어를 사용할 때, Spotify의 인증 쿠키가 필요합니다. `SameSite=None`을 설정하면 이러한 서드파티 서비스가 정상적으로 작동할 수 있습니다.

2. **OAuth 및 SSO**:
   - OAuth나 SSO(Single Sign-On) 같은 인증 흐름에서는 여러 도메인 간에 쿠키를 주고받아야 할 때가 많습니다. 이 경우 `SameSite=None`이 필요합니다.

3. **광고 및 분석 도구**:
   - 광고 네트워크나 분석 도구가 여러에 걸쳐 사용자 활동을 추적할 때도 사용됩니다.

`SameSite=None`을 사용할 때는 `Secure` 속성을 함께 설정해야 합니다. 이는 HTTPS 연결에서만 쿠키가 전송되도록 하여 보안을 강화합니다.

---
### Token
Spotify에서 Authorization 토큰과 Client 토큰을 모두 사용하는 이유는 다음과 같습니다:

1. **Authorization 토큰:**
   - **사용자 인증 및 권한 부여:** 사용자가 Spotify 계정에 로그인할 때 발급됩니다. 사용자의 개인 데이터에 접근하거나 사용자 계정과 관련된 작업을 수행할 때 필요합니다.
   - **OAuth 2.0 프로토콜:** 주로 OAuth 2.0을 통해 발급되며, 사용자 권한을 기반으로 API 요청을 인증합니다.

2. **Client 토큰:**
   - **애플리케이션 인증:** 특정 애플리케이션이 Spotify API에 접근할 있도록 인증합니다. 사용자와 무관하게 애플리케이션 자체의 신원을 확인하는 데 사용됩니다.
   - **공개 데이터 접근:** 사용자 인증 없이도 접근 가능한 공개 데이터에 대한 요청을 처리할 때 사용됩니다.

**이 둘을 분리하는 이유:**

- **보안:** 사용자 데이터와 애플리케이션 인증을 분리하여 보안을 강화합니다.
- **유연성:** 애플리케이션이 사용자 인증도 일부 기능을 수행할 수 있도록 합니다.
- **효율성:** 사용자 인증이 필요하지 않은 경우 Client 토큰을 사용하여 불필요한 인증 절차를 줄입니다.

이러한 구조는 Spotify가 다양한 사용 사례를 지원하고, 보안과 효율성을 동시에 유지할 수 있도록 도와줍니다.

// 토큰 관리를 위한 클래스
```js
class SpotifyTokenManager {
  constructor() {
    this.authToken = null;
    this.clientToken = null;
  }

  async getAuthToken() {
    if (!this.authToken) {
      // OAuth 2.0 인증 흐름을 통해 토큰 획득
      this.authToken = await this.performOAuthFlow();
    }
    return this.authToken;
  }

  async getClientToken() {
    if (!this.clientToken) {
      // 클라이언트 인증을 통해 토큰 획득
      this.clientToken = await this.getClientCredentials();
    }
    return this.clientToken;
  }

  async performOAuthFlow() {
    // OAuth 2.0 인증 흐름 구현
    // 실제 구현은 복잡할 수 있으며, 라이브러리 사용을 권장
  }

  async getClientCredentials() {
    // 클라이언트 인증 구현
    // 주로 클라이언트 ID와 시크릿을 사용
  }
}

// Spotify API 요청을 위한 클래스
class SpotifyAPI {
  constructor(tokenManager) {
    this.tokenManager = tokenManager;
  }

  async getUserPlaylists(userId) {
    const authToken = await this.tokenManager.getAuthToken();
    const response = await fetch(`https://api.spotify.com/v1/users/${userId}/playlists`, {
      headers: {
        'Authorization': `Bearer ${authToken}`
      }
    });
    return response.json();
  }

  async searchPublicPlaylists(query) {
    const clientToken = await this.tokenManager.getClientToken();
    const response = await fetch(`https://api.spotify.com/v1/search?q=${query}&type=playlist`, {
      headers: {
        'Authorization': `Bearer ${clientToken}`
      }
    });
    return response.json();
  }
}

// 사용 예시
async function main() {
  const tokenManager = new SpotifyTokenManager();
  const spotifyAPI = new SpotifyAPI(tokenManager);

  // 사용자 플레이리스트 조회 (Auth 토큰 사용)
  const userPlaylists = await spotifyAPI.getUserPlaylists('user123');
  console.log('User Playlists:', userPlaylists);

  // 공개 플레이리스트 검색 (Client 토큰 사용)
  const searchResults = await spotifyAPI.searchPublicPlaylists('rock');
  console.log('Search Results:', searchResults);
}

main().catch(console.error);

```
