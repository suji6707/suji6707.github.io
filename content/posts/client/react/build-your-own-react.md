+++
title = 'Build your own react'
date = 2024-08-28T00:20:22+09:00
draft = true
+++

💙🩵 Build your own react

<Part1> virtualDOM, renderer

타입스크립트가 React.element를 만들어줌
- HTML Tag, any props, and then any children
DOM(document object model)은 HTML 구조를 specify하는 것.
virtual DOM은 비슷하나 어떻게 그릴지 구현에 따라 다름.

ReactElement vs ReactComponent
>> 컴포넌트는 virtualDOM을 매핑할 뿐. 함수를 리턴하도록 하고, tag가 함수면 tag에 인자로 props, children을 넘겨주면 됨.
- state 등 복잡한 작업을 할 때 element를 직접 사용할 수 없으므로 we may want to wrap them in a function which is known more popularly as ReactComponent.
- Components which are simply functions wrapping VirtualDOM instead of the actual VirtualDOM withtag , props and children

Render our VirtualDOM "어떻게 DOM 트리에 렌더링할 것인가?"
- React 컴포넌트를 실제 DOM 요소에 렌더링하기 위해, HTML 파일에 특정 요소가 필요
-> div id=myapp을 마운트 포인트 요소로 삼는다.
이 요소는 index.html에 존재하며, 이 요소를 기준으로 DOM 트리를 구성한다.
- render(el, container) => el은 리액트 컴포넌트 <App/>, 두번째는 실제 DOM 요소 document.getElementById('myapp')가 전달됨
-> 브라우저 API를 사용하여 DOM 노드를 생성하는 것.

지금까지 DOM 렌더링까지 단계를 생각해보면
1. App 컴포넌트: 태그를 document.createElement(tag) API를 통해 DOM 노드로 생성하고
2. props 전달, children 요소 재귀적으로 append

      <h2>Hello React!</h2>

지금은 child가 그냥 텍스트라, 재귀적으로 그릴때 tag, props, children 요소가 없어 에러가 남.


---
< Part 2> 상태관리, 리액트 훅
언제 setState가 불렸는지 알고 업데이트할 것인가?
항상 리렌더를 한다-

Re-render
- setState 함수 안에 reRender() 함수를 넣는다.
컴포넌트내 useState 먼저 하고 리턴 jsx.
- 현재 문제
1. useState 불릴때마다 초기화되고 상태유지 안됨
    - global variable.
2. 업데이트가 아닌 새로 append를 함.
    - ''로 clear 후.

Statefulness, Global State 관리
- useState가 여러개일때. 
- 서로다른 state에 접근할 때 cursor++

---
< Part 3 > Suspense, Cuncurrent Mode 등 최적화
1. 보통: Fetch on Render
- 일단 렌더링, 그다음 fetch.
- initial 렌더 이후 fetch 데이터가 준비되면 State를 통해 데이터를 채운다.
- 여기서 문제는 fetche될 데이터에 의존하면 waterfall 현상 발생

2. Fetch then Render
fetch를 하고 나서 setState

3. Suspense
fetch 먼저하는 건 마찬가지이나 
fetching- render - finish fetching
다 기다렸다가 렌더링하는게 아님.
바로 렌더링하고, fetch가 완료되면 state에 반영함

어떻게 구현?
DOM 생성 프로세스에 기다릴동안 뭘 표시할지 fallback을 알려줘야함.

* createResource
모든 비동기호출 및 resolved 값을 추적하기 위해 키를 씀.
- createResouce(asyncTask, key)


