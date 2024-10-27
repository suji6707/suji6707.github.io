+++
title = 'vercel & '
date = 2024-08-31T00:20:22+09:00
draft = true
+++
## Introduction




#### 싱글 페이지 애플리케이션(SPA)
전통적인 다중 페이지 애플리케이션(MPA, Multi-Page Application)과 달리, 하나의 HTML 페이지로 구성된 웹 애플리케이션입니다. SPA는 초기 로딩 시 필요한 모든 자원을 한 번에 로드하고, 이후 사용자가 상호작용할 때마다 페이지 전체를 다시 로드하지 않고 필요한 부분만 동적으로 업데이트합니다.



---

`<script>`가 자바스크립트
`<template>`이 html 영역


---

# HTML 기초

html의 속성
- 요소에 추가적인 내용을 담고싶을 때 (태그가 곧 요소)
- CSS: 클래스, id
	- html 요소에 이름을 주는 방법
- img 태그의 src=값
- a 태그의 href=값

--
Block-level elements
- 앞뒤로 줄바꿈 나타남.
-  단락(Paragraphs), 목록(lists), 네비게이션 메뉴(Navigation Menus), 꼬리말(Footers) 등

Inline elements
- 항상 블록레벨 요소에 포함되어있음.
- `<a>` , 텍스트(Text)를 강조하는 요소인 `<em>`,`<strong>` 등

--
HTML에 CSS 와 JavaScript 적용하기
```js
// <head> 부분
<link rel="stylesheet" href="my-css-file.css" />

// <body>앞 또는 본문 끝
<script src="my-js-file.js"></script>

```

--
#### 실제 유의미한 태그
div
a
input
ul, ol, li
table
canvas

나머지는 파생. button도 input에서-
header, section, main 같은건 시맨틱 태그임. h1, h2가 대표적.
실질적으론 div랑 비슷.

--


---
### 시행착오

```js
// App.vue 에서 <div> 뻬고 바로. -> 전체 크기 안바뀌게
<template>
  <router-view />
</template>

#app 
	height: 100%;

// index.html: body, app에 height 직접 설정해줌
<body style="height: 100vh; margin: 0">
  <noscript>
    <strong>We're sorry but <%= htmlWebpackPlugin.options.title %> doesn't work properly without JavaScript enabled.
        Please enable it to continue.</strong>
  </noscript>
  <div id="app"></div>
```

### scss
Scoped CSS. - 이름 겹쳐도 O
Nested를 지원함.

```js
<style lang="scss" scoped>
```

```
nav {
  ul {
    margin: 0;
    padding: 0;
    list-style: none;
  }

  li { display: inline-block; }

  a {
    display: block;
    padding: 6px 12px;
    text-decoration: none;
  }
}
```

---

TODO
- CSS flex
- html 문서구조 마저