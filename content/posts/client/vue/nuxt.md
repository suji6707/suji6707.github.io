+++
title = 'Vue'
date = 2024-10-26T00:20:22+09:00
draft = true
+++
## Nuxt

SSR은 서버에서 사전에 HTML을 생성해 클라이언트로 보내주는 것.
Nextjs는 버셀 서버 리소스를 활용하고
Nuxt는 내장 Node.js 서버 사용(버셀로 Nuxt 배포할 수 있는데 필수는 아님)
- Nuxt.js는 기본적으로 npm run dev 명령어로 개발 서버를 실행할 때, npm run start 명령어로 프로덕션 서버를 실행할 때 사용할 수 있는 내장 Node.js 서버를 제공합니다. 

---
### NuxtPage, app.vue
Nuxt.js에서 페이지를 생성하면, `pages` 폴더 안의 각 파일은 자동으로 애플리케이션의 라우트에 연결됩니다. 예를 들어, `pages/index.vue` 파일은 `/` 경로에 매핑됩니다. 

**`NuxtPage` 컴포넌트는 `app.vue` 파일을 사용할 때 필요합니다.** `app.vue`는 애플리케이션의 레이아웃을 정의하는 파일이며, 모든 페이지에서 공통으로 사용되는 요소들을 포함합니다. 

`app.vue` 파일 내에서 `<NuxtPage />` 컴포넌트를 사용하면 현재 라우트에 매핑된 페이지 컴포넌트를 렌더링할 수 있습니다. 즉, `app.vue`는 앱의 뼈대를 구성하고, `<NuxtPage />`는 각 페이지의 내용을 뼈대 안에 채워 넣는 역할을 합니다.

**예시:**

```js
// app.vue
<template>
  <div>
    <header>내 웹사이트</header>
    <NuxtPage /> 
    <footer>푸터</footer>
  </div>
</template>
```

```js
// pages/index.vue
<template>
  <h1>Index 페이지</h1>
</template>
```

위 예시에서 `/` 경로에 접속하면 `app.vue`에 정의된 헤더와 푸터 사이에 `index.vue`의 내용이 나타납니다. 

**`NuxtPage`를 사용하지 않으면 `app.vue`에 정의된 내용만 표시되고, 실제 페이지 내용은 렌더링되지 않습니다.**

**참고:**

* 각 페이지 컴포넌트는 반드시 하나의 루트 엘리먼트만 가져야 합니다. 
* HTML 주석도 엘리먼트로 간주되므로 주의해야 합니다.
* `NuxtPage` 컴포넌트를 사용하면 서버 사이드 렌더링이나 정적 사이트 생성 시에도 페이지 내용이 정확하게 표시됩니다.


---


