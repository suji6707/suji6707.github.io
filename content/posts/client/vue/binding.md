+++
title = 'Vue'
date = 2024-09-26T00:20:22+09:00
draft = true
+++

# Binding


\<script setup> 블록 안에 있는 변수들은 바로  \<template> 에서 참조할 수 있기 때문에 우리가 JS를 HTML에 다이나믹하게 반영할 수 있다. 

script 블록에선 ref 값에 접근하려면 .value를,
템플릿에선 바로 참조 가능.

{{}}은 텍스트 대치에만 사용되고 < >{{here!}} < >,
속성을 바인딩하려면(객체형태) v-bind를 써야한다.

v-bind:id="dynamicId"
id 속성은 컴포넌트 상태의 dynamicId와 동기화된다.
- id, class 등 기본 속성은 물론 임의로 정한 변수도 된다.

v-on : DOM 이벤트를 리스닝한다.
- 원래 단방향 데이터 바인딩. 일방통행.
컴포넌트에서 템플릿으로는 데이터가 자동으로 가지만,
반대방향으로(html user interaction -> component/script) 가려면
다른 길(@input 이벤트 리스너)를 통해야 한다.

=> v-bind(script의 변수를 html에 동기화시킨다), v-on(html의 이벤트 타겟의 값을 script에 반영 - e.target.value)을 함께 사용함으로써 
양방향 데이터 바인딩을 한다.

이걸 한번에 하는게
v-model="변수명" 
- <input> value를 자동으로 state에 반영해줘서 이벤트 핸들러가 필요없다.

<li>는 key 속성이 바인딩되어있어야 한다.
<li>가 어떤 요소에 매칭되는지 파악하기 위해.
