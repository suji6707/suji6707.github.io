+++
title = 'Code Mirror'
date = 2024-10-26T00:20:22+09:00
draft = true
+++

💎 code mirror
shallowRef: 최상위 속성만 반응형으로. -> view 변수의 값 자체만 추적하고 내부 속성 변경은 추적하지 않음.

EditorView 인스턴스를 저장하기 위해 view를 사용.
EditorView는 CodeMirror 편집기의 핵심 객체로, 편집기 상태와 DOM을 관리.

view.value를 통해 EditorView 인스턴스에 접근할 수 있도록 
view.value에 payload.view를 할당.

v-model이 없으면 어디서 이벤트를 주지를 볼것.

컴포넌트가 나한테 값을 주는 방식
- vmodel, 이벤트


💎 Vue의 존재이유
** 결국은 HTML을 변형하는 게 목표.
그래서 state를 쓰는거고.
뷰가 그걸 자동으로 해주는거고. v-model -> DOM
'data가 이러면 이렇게 그려'
'state'가 전부임.

어떤 것이든 눈에 보이는건 HTML 태그로 정의할 수 있다.
그러나 그걸 직접 document.getElement로 변경하면 얼마나 귀찮겠는가? 이벤트며..

