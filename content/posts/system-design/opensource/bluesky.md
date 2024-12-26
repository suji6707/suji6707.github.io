+++
title = 'BlueSky'
date = 2024-10-27T00:20:22+09:00
draft = true
+++
## BlueSky
https://newsletter.pragmaticengineer.com/p/bluesky

이 글에서 PDS는 **개인 데이터 서버(Personal Data Server)**를 의미합니다. Bluesky 시스템에서 PDS는 다음과 같은 역할을 합니다.

**1. 사용자 데이터 저장 및 관리:**

* 각 사용자는 자신의 게시물, 팔로워, 팔로잉 목록 등을 포함한 모든 데이터를 저장하는 PDS를 갖습니다.
* Bluesky가 호스팅하는 PDS를 사용하거나, 사용자가 직접 서버를 운영하여 자체 호스팅 PDS를 사용할 수 있습니다.

**2. 분산 네트워크 구축의 핵심 요소:**

* 기존 중앙 집중식 소셜 네트워크와 달리 Bluesky는 사용자 데이터를 분산하여 저장합니다.
* 이러한 분산형 구조는 Bluesky를 특정 회사의 통제에서 벗어나 자 censorship resistance를 제공합니다.

**3. 데이터 동기화 및 전파:**

* Relay 서비스는 모든 PDS를 크롤링하여 새로운 이벤트(게시물, 좋아요 등)를 수집합니다.
* 수집된 이벤트는 Firehose라는 시스템을 통해 다른 PDS 및 서비스에 전파됩니다.

**4. 다양한 애플리케이션 및 서비스와의 연동:**

* PDS는 AT Protocol을 통해 다양한 애플리케이션 및 서비스와 통신합니다.
* 사용자는 Bluesky 공식 앱 외에도 PDS와 연동되는 서드파티 앱을 자유롭게 사용할 수 있습니다.

**5. 확장성 및 성능 향상:**

* Bluesky는 처음에 PostgreSQL을 사용했지만, 확장성 문제로 인해 ScyllaDB와 SQLite로 마이그레이션했습니다.
* 각 사용자에게 SQLite 데이터베이스를 제공함으로써 PDS 운영 비용을 절감하고 성능을 향상시켰습니다.

요약하자면, PDS는 Bluesky 시스템에서 사용자 데이터 저장 및 관리, 분산 네트워크 구축, 데이터 동기화, 다양한 서비스와의 연동, 확장성 및 성능 향상에 중요한 역할을 합니다.

---
### PDS구현 
PDS 구현 방법은 Bluesky 아키텍처의 핵심이며, 사용자에게 데이터 소유권과 제어권을 제공하는 데 중요한 역할을 합니다. 

**1. 저장 방식:**

* Bluesky는 초기에 PostgreSQL을 사용했지만, 확장성 문제로 인해 ScyllaDB와 SQLite로 마이그레이션했습니다. 
* **Appview**: 읽기 작업이 많은 Appview 서비스에는 수평적 확장이 가능한 wide-column 데이터베이스인 **ScyllaDB**를 사용합니다.
* **개인 데이터 서버(PDS)**: 각 사용자에게는 개별 SQLite 데이터베이스가 할당됩니다. **SQLite**는 단일 파일로 저장되고 설정이 간편하여 PDS 운영에 이상적입니다.

**2. 페더레이션:**

* Bluesky 네트워크는 페더레이션을 통해 서로 다른 PDS 간의 상호 작용을 가능하게 합니다. 
* 사용자는 Bluesky에서 호스팅하는 PDS를 사용하거나 직접 서버를 운영하여 자체 호스팅 PDS를 사용할 수 있습니다.
* **릴레이 서비스**: 릴레이는 여러 PDS에서 이벤트를 수집하여 네트워크 전체에 정보를 전파합니다.
* **크롤러**: 릴레이 서비스는 크롤러를 사용하여 각 PDS를 순회하며 새로운 이벤트를 수집합니다.

**3. 모듈식 아키텍처:**

* Bluesky는 피드 생성, 보기 로직, 중재 기능과 같은 다양한 기능을 별도의 서비스로 분리하는 모듈식 아키텍처를 사용합니다.
* 이러한 모듈성을 통해 개발자는 네트워크의 특정 부분에 집중하여 자체 서비스를 구축하고 Bluesky 생태계에 통합할 수 있습니다.

**4. API 및 프로토콜:**

* Bluesky는 PDS 간의 통신 및 데이터 교환을 위해 AT Protocol(Authenticated Transfer Protocol)을 사용합니다.
* AT Protocol은 HTTP 및 JSON을 기반으로 구축된 오픈 소스 프로토콜입니다.

**5. 보안:**

* 각 PDS는 사용자 데이터 보호를 위해 암호화 및 인증과 같은 보안 조치를 구현해야 합니다.
* Bluesky는 분산 환경에서 보안 문제를 해결하기 위해 노력하고 있으며, PDS 운영자에게 보안 모범 사례에 대한 지침을 제공합니다.

**요약:**

Bluesky의 PDS 구현은 데이터베이스 기술, 페더레이션 메커니즘, 모듈식 설계, API 및 보안 조치를 결합하여 사용자에게 진정한 데이터 소유권과 제어권을 제공하는 분산형 소셜 네트워킹 경험을 제공합니다.

---
### Relay

Bluesky에서 릴레이 서비스는 분산된 PDS들 사이에서 정보를 효율적으로 전파하는 중요한 역할을 수행합니다. 릴레이는 크게 '이벤트 수집'과 '정보 전파' 두 가지 기능을 수행하며, 각 기능은 다음과 같이 구체적으로 작동합니다.

**1. 이벤트 수집**

* **구독**: 릴레이는 자신에게 연결된 PDS들을 구독합니다. 즉, 특정 PDS에서 새로운 이벤트(게시글, 댓글, 좋아요 등)가 발생하면 릴레이가 이를 즉시 인지할 수 있도록 설정합니다.
* **Firehose**: 각 PDS는 발생하는 이벤트들을 Firehose라는 스트림에 게시합니다. 릴레이는 연결된 PDS들의 Firehose를 지속적으로 모니터링하며 새로운 이벤트를 실시간으로 수집합니다.
* **필터링**: 릴레이는 특정 종류의 이벤트만 선택적으로 수집하도록 설정할 수 있습니다. 예를 들어, 특정 주제나 사용자와 관련된 이벤트만 수집하거나, 특정 PDS에서 발생한 이벤트만 수집할 수 있습니다.

**2. 정보 전파**

* **다른 릴레이와 연결**: 릴레이는 다른 릴레이들과 연결되어 보다 광범위한 네트워크를 형성합니다. 이를 통해 특정 릴레이에 직접 연결되지 않은 PDS의 이벤트도 전파될 수 있습니다.
* **캐싱**: 릴레이는 수집한 이벤트를 일정 시간 동안 캐싱하여,  동일한 정보를 여러 번 요청하는 경우 빠르게 제공할 수 있습니다.
* **푸시 & 풀 방식**: 릴레이는 푸시 방식과 풀 방식을 모두 사용하여 정보를 전파합니다. 
    * **푸시 방식**:  새로운 이벤트 발생 시, 릴레이는 자신에게 연결된 다른 릴레이나 PDS에 해당 정보를 능동적으로 전달합니다.
    * **풀 방식**: 정보를 필요로 하는 릴레이나 PDS가  다른 릴레이에게 정보를 요청하면, 해당 릴레이는 캐시된 정보를 제공합니다.

**예시**

1. 사용자 A가 자신의 PDS에 새로운 게시글을 작성합니다.
2. 사용자 A의 PDS는 해당 이벤트를 Firehose에 게시합니다.
3. 사용자 A의 PDS를 구독하는 릴레이는 해당 이벤트를 수집합니다.
4. 릴레이는 수집한 이벤트를 자신과 연결된 다른 릴레이와 PDS에 전파합니다.
5. 사용자 B가 Bluesky 앱을 통해 자신의 타임라인을 확인합니다.
6. 사용자 B의 PDS는 연결된 릴레이에게 최신 정보를 요청합니다.
7. 릴레이는 캐시된 정보 중 사용자 B와 관련된 정보를 선별하여 사용자 B의 PDS에 전달합니다.

이처럼 릴레이 서비스는 Bluesky 네트워크에서 발생하는 다양한 이벤트를 효율적으로 수집하고 전파하여 분산된 환경에서도 정보 공유가 원활하게 이루어지도록 돕는 중요한 역할을 합니다. 

---
### Feed Generator
https://github.com/bluesky-social/feed-generator

Bluesky의 Feed Generator는 사용자에게 맞춤형 피드를 제공하는 핵심 요소입니다. 사용자가 Bluesky에서 다양한 게시물을 접할 때, 단순히 시간 순서대로 보는 것이 아니라, 사용자의 관심사나 다른 기준에 따라 재구성된 피드를 보게 됩니다. 이러한 맞춤형 피드 생성을 담당하는 것이 바로 Feed Generator입니다.

**Feed Generator 작동 방식:**

1. **데이터 수집**: Feed Generator는 Bluesky 네트워크에서 발생하는 다양한 이벤트 정보를 수집합니다. 
    * **Firehose**: 릴레이 서비스가 제공하는 Firehose를 통해 실시간으로 새로운 게시물, 댓글, 좋아요 등의 이벤트 정보를 얻습니다.
    * **PDS**: 특정 사용자의 PDS에 접근하여 해당 사용자의 팔로잉 목록, 좋아요 표시한 게시물 등의 정보를 가져올 수 있습니다.

2. **데이터 처리 및 필터링**: 수집된 정보를 바탕으로 Feed Generator는 사용자에게 필요한 정보만 선별하고 가공합니다.
    * **팔로잉**: 사용자가 팔로잉하는 계정들의 게시물을 우선적으로 수집합니다.
    * **관심사**: 사용자의 활동 (좋아요, 댓글, 리포스트 등)을 분석하여 관심 있는 주제나 계정을 파악하고, 관련 게시물의 우선순위를 높입니다.
    * **시간**: 최신 게시물일수록 우선순위를 높게 설정할 수 있습니다.
    * **인기도**: 좋아요, 댓글, 리포스트 수 등을 기반으로 게시물의 인기도를 측정하여 우선순위에 반영할 수 있습니다.

3. **피드 생성 및 업데이트**:  Feed Generator는 처리된 정보를 기반으로 사용자에게 보여줄 최종 피드를 생성합니다.
    * **정렬**: 사용자의 관심사, 시간 순서, 인기도 등을 고려하여 게시물을 정렬합니다.
    * **필터링**: 특정 키워드, 사용자, 주제를 차단하거나 허용하는 필터를 적용할 수 있습니다.
    * **페이징**: 한 번에 모든 게시물을 보여주는 대신, 일정 개수씩 나누어 보여주는 페이징 기법을 사용하여 성능을 향상시킵니다.

4. **피드 제공**: 생성된 피드는 사용자의 요청에 따라 Bluesky 앱에 전달되어 화면에 표시됩니다.

**핵심 특징**:

* **분산화**: Bluesky의 Feed Generator는 중앙 집중식이 아닌 분산형 구조를 가지고 있습니다. 즉, 누구나 자신만의 Feed Generator를 만들어 운영할 수 있으며, 사용자는 자신에게 맞는 Feed Generator를 선택하여 사용할 수 있습니다.
* **맞춤화**: 사용자는 자신에게 맞는 다양한 기준을 설정하여 자신만의 맞춤형 피드를 만들 수 있습니다.
* **투명성**: Feed Generator의 알고리즘은 공개되어 있어 누구나 어떤 방식으로 피드가 생성되는지 확인할 수 있습니다.

**결론**:

Bluesky의 Feed Generator는 단순한 게시물 나열을 넘어 사용자 개개인에게 최적화된 정보 경험을 제공하는 중요한 역할을 합니다. 분산화, 맞춤화, 투명성을 통해 사용자 중심의 소셜 네트워킹 경험을 제공하는 것이 Bluesky Feed Generator의 핵심 목표입니다. 

--
### Feed generator

네, 이 Feed Generator 프로젝트는 기본적으로 서버 측 애플리케이션입니다. 이 서버는 Bluesky 네트워크의 일부로 작동하며, 사용자의 Personal Data Server(PDS)와 상호 작용합니다.

클라이언트에서 이 Feed Generator를 사용하려면 다음과 같은 과정을 거칩니다:

1. **Feed 구독**:
   - 사용자는 Bluesky 앱(클라이언트)에서 특정 Feed Generator를 구독합니다.
   - 이 과정에서 Feed Generator의 at-uri가 사용자의 PDS에 등록됩니다.

2. **Feed 요청**:
   - 사용자가 Bluesky 앱에서 해당 피드를 보려고 할 때, 앱은 사용자의 PDS에 요청을 보냅니다.
   - 요청 예시: `GET /xrpc/app.bsky.feed.getFeed?feed=at://did:example:alice/app.bsky.feed.generator/my-custom-feed`

3. **PDS의 처리**:
   - PDS는 Feed Generator의 DID를 확인하고, Feed Generator 서버에 `getFeedSkeleton` 요청을 보냅니다.
   - 이 요청에는 사용자 인증 정보(JWT)가 포함됩니다.

4. **Feed Generator의 응답**:
   - Feed Generator는 요청을 처리하고 피드 스켈레톤(게시물 URI 목록)을 PDS에 반환합니다.

5. **PDS의 데이터 hydration**:
   - PDS는 받은 URI 목록을 바탕으로 실제 게시물 내용, 사용자 정보 등을 채웁니다.

6. **클라이언트로의 응답**:
   - PDS는 완성된 피드 데이터를 Bluesky 앱(클라이언트)에 반환합니다.

7. **클라이언트의 표시**:
   - Bluesky 앱은 받은 데이터를 사용자에게 표시합니다.

따라서 클라이언트 개발자의 입장에서는 Feed Generator와 직접 통신할 필요가 없습니다. 대신, Bluesky의 표준 API를 사용하여 PDS와 통신하면 됩니다. 

만약 자체 클라이언트를 개발한다면:

1. Bluesky API를 사용하여 사용자 인증을 처리합니다.
2. 구독 가능한 Feed Generator 목록을 가져오고 사용자가 선택할 수 있게 합니다.
3. 선택된 Feed Generator의 피드를 요청할 때는 표준 `app.bsky.feed.getFeed` API를 사용합니다.
4. 받은 피드 데이터를 파싱하고 UI에 표시합니다.

이렇게 하면 Feed Generator의 복잡성은 서버 측에 숨겨지고, 클라이언트는 일관된 API를 통해 다양한 Feed Generator의 결과를 쉽게 표시할 수 있습니다.

```js
import axios from 'axios';

async function getFeed() {
  const response = await axios.get('https://your-feed-generator.com/xrpc/app.bsky.feed.getFeedSkeleton', {
    params: {
      feed: 'at://did:example:alice/app.bsky.feed.generator/whats-alf',
    },
    headers: {
      Authorization: `Bearer YOUR_JWT_TOKEN`,
    },
  });

  const feedSkeleton = response.data;
  // Process and display the feed
}

getFeed();
```

---
### AT Protocol
https://github.com/bluesky-social/atproto
https://atproto.com/guides/overview

Bluesky에서 AT Protocol(ATP)은 전체 시스템의 근간을 이루는 핵심 기술입니다. 탈중앙화 소셜 네트워크를 구축하기 위해 다양한 측면에서 활용됩니다. 

**1. 데이터 구조 정의 (Lexicon)**

AT Protocol은 Lexicon이라는 스키마 정의 언어를 사용하여 데이터 구조를 정의합니다. Bluesky는 Lexicon을 활용하여 게시물, 댓글, 좋아요, 팔로우 등 소셜 네트워크 기능에 필요한 데이터 구조를 정의합니다. 

* 예시: 게시물 Lexicon은 텍스트 내용, 이미지, 첨부 파일 등 게시물 관련 정보를 담는 구조를 정의합니다.

Bluesky에서 AT Protocol (atproto)는 다음과 같이 구체적으로 사용되었습니다:

1. 데이터 구조 및 저장:
   - `@atproto/repo` 패키지를 사용하여 사용자 데이터를 Merkle Search Tree (MST) 구조로 저장합니다.
   - 예: 사용자의 게시물, 팔로워 목록 등이 이 구조로 저장됩니다.

2. 식별자 및 인증:
   - `@atproto/identity` 패키지로 DID(Decentralized Identifier)와 handle을 관리합니다.
   - 예: 사용자의 고유 식별자 "alice.bsky.social"은 DID로 변환되어 관리됩니다.

3. API 정의 및 통신:
   - `@atproto/lexicon`으로 API 스키마를 정의하고, `@atproto/xrpc`로 HTTP API 통신을 처리합니다.
   - 예: 게시물 작성 API "com.atproto.repo.createRecord"가 lexicon으로 정의되고 xrpc로 호출됩니다.

4. 클라이언트 라이브러리:
   - `@atproto/api` 패키지로 클라이언트 애플리케이션에서 Bluesky API를 쉽게 사용할 수 있게 합니다.
   - 예: 
     ```typescript
     import { BskyAgent } from '@atproto/api'
     
     const agent = new BskyAgent({ service: 'https://bsky.social' })
     await agent.login({ identifier: 'alice@example.com', password: 'xxxx' })
     await agent.post({ text: 'Hello, world!' })
     ```

5. 서버 구현:
   - PDS(Personal Data Server)와 AppView 서비스가 AT Protocol을 기반으로 구현되었습니다.
   - 예: PDS는 사용자의 개인 데이터를 저장하고 관리하며, AppView는 앱 특화 기능을 제공합니다.

6. 암호화 및 보안:
   - `@atproto/crypto` 패키지로 암호화 서명과 키 관리를 처리합니다.
   - 예: 사용자 인증 및 데이터 무결성 검증에 사용됩니다.

이러한 구현을 통해 Bluesky는 분산형, 자체 인증 소셜 네트워크를 AT Protocol 기반으로 구축했습니다.

---
### MST
AT Protocol에서 MST의 키 깊이 계산과 저장 공간 효율성을 위한 키 압축 방식 설명

이 예시는 AT Protocol의 MST(Merkle Search Tree)에서 키의 깊이(depth)를 계산하는 방법과 키 압축 방식을 설명하고 있습니다. 각 부분을 자세히 살펴보겠습니다.

1. 키의 깊이 계산 예시:

   ```
   2653ae71: depth "0"
   blue: depth "1"
   app.bsky.feed.post/454397e440ec: depth "4"
   app.bsky.feed.post/9adeb165882c: depth "8"
   ```

   이 예시들은 다양한 키들의 깊이를 보여줍니다. 깊이는 다음과 같이 계산됩니다:
   - 키를 SHA-256으로 해시합니다.
   - 해시 결과의 선행 이진 0의 개수를 세고 2로 나눕니다(내림).
   - 결과값이 키의 깊이가 됩니다.

   예를 들어, "blue"의 깊이가 1이라는 것은 해시 결과의 첫 2~3비트가 0이라는 의미입니다.

2. 키 압축 방식:

   MST의 각 노드에서 키(바이트 배열)는 공통 접두사를 생략하여 압축됩니다. 이는 저장 효율성을 높이기 위한 방법입니다.

   압축 방식의 특징:
   - 각 엔트리는 이전 키와 공유하는 바이트 수를 표시합니다.
   - 노드의 첫 번째 엔트리는 전체 키를 포함하며, 공통 접두사 길이는 0입니다.
   - 이 압축은 노드 내부에서만 적용되며, 트리의 여러 노드에 걸쳐 적용되지 않습니다.
   - 이 압축 방식은 필수적이며, 이를 통해 MST 구조가 구현에 관계없이 결정적(deterministic)이 됩니다.

예시:
노드에 다음과 같은 키들이 있다고 가정해봅시다:
```
app.bsky.feed.post/123
app.bsky.feed.post/456
app.bsky.feed.comment/789
```

이 키들은 다음과 같이 압축될 수 있습니다:
1. "app.bsky.feed.post/123" (전체 키, 공통 접두사 길이 0)
2. "456" (이전 키와 19바이트 공유)
3. "comment/789" (이전 키와 13바이트 공유)

이러한 압축 방식을 통해 저장 공간을 크게 절약할 수 있으며, 특히 유사한 키가 많은 경우 효과적입니다.

이 접근 방식은 MST의 효율성과 결정성을 보장하면서도 저장 공간을 최적화하는 데 중요한 역할을 합니다.

---
### Ozone
https://github.com/bluesky-social/ozone

 Bluesky에서 Ozone은 콘텐츠에 레이블을 지정하는 웹 인터페이스로, 사용자가 직접 PDS에 Ozone을 설치하여 활용하는 것은 아닙니다. Ozone은 Bluesky 네트워크에서 콘텐츠 Moderation을 분산화하기 위해 만들어진 독립적인 서비스입니다. 

다음은 Ozone과 PDS의 관계를 자세히 설명합니다.

**Ozone의 역할:**

* **콘텐츠 레이블 지정:** Ozone은 게시물, 사용자 프로필 등 Bluesky 콘텐츠에 레이블을 지정하는 데 사용됩니다. 
* **분산형 Moderation:** Ozone을 통해 사용자는 자신만의 Moderation 정책을 설정하고, 특정 콘텐츠를 필터링하거나 강조 표시할 수 있습니다.
* **맞춤형 피드 생성:** Ozone에서 생성된 레이블은 사용자 맞춤형 피드를 생성하는 데 사용될 수 있습니다. 예를 들어, 특정 주제에 관심 있는 사용자는 해당 주제와 관련된 콘텐츠만 볼 수 있습니다.

**PDS와의 관계:**

* **Ozone은 PDS에서 생성된 콘텐츠를 가져와 레이블을 지정합니다.** 즉, Ozone은 PDS와 분리된 서비스이지만 PDS의 데이터를 기반으로 작동합니다.
* **PDS는 Ozone에서 생성된 레이블을 사용하여 콘텐츠를 필터링하거나 표시할 수 있습니다.** 예를 들어, 특정 레이블이 지정된 콘텐츠를 숨기거나 경고 메시지와 함께 표시할 수 있습니다.

**PDS에 Ozone을 활용하는 구체적인 방법:**

* **Ozone 서비스 연동:** PDS 운영자는 자신의 PDS에 Ozone 서비스를 연동하여 Ozone에서 생성된 레이블을 활용할 수 있습니다.
* **API 활용:** Ozone은 API를 통해 레이블 정보를 제공합니다. PDS는 이 API를 사용하여 Ozone에서 생성된 레이블을 가져와 콘텐츠 Moderation에 활용할 수 있습니다.
* **커뮤니티 기반 Moderation:** Ozone을 통해 특정 커뮤니티는 자체 Moderation 정책을 설정하고, 해당 정책을 반영한 레이블을 생성하여 PDS에 적용할 수 있습니다.

**요약:**

Ozone은 PDS에 직접 설치되는 것이 아니라, PDS와 연동하여 콘텐츠 Moderation을 분산화하고 사용자 맞춤형 경험을 제공하는 독립적인 서비스입니다. PDS 운영자는 Ozone API를 활용하여 Ozone에서 생성된 레이블을 자신의 Moderation 정책에 반영할 수 있습니다. 

---
