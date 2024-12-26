+++
title = 'React Component'
date = 2024-10-25T00:20:22+09:00
draft = true
+++
## React Component


propTypes를 사용하여 사용자 정의 검증 함수를 작성

### custom component
🟡(요약)
- React.createElement 대신 Capital Letter로 시작하는 함수를 사용 -> JSX 가능.
Babel로 하여금 그 함수를 그대로 reference 하도록 알려줌.
- React.createElement는 type에 함수가 오면, 그 함수 안으로 들어가 다른 React element가 있는지 보고 그걸 create 한다.


🍎 it's Babel that's responsible for taking our JSX and compiling it
to React.createElement calls 

JSX는 JavaScript XML의 약자로, **JavaScript 코드 안에서 HTML과 유사한 마크업**을 작성할 수 있게 해주는 React의 문법 확장입니다. JSX를 사용하면 React 요소를 더 직관적이고 선언적으로 작성할 수 있습니다.

1. 🍎 JSX를 사용하지 않은 방식:
   ```javascript
   React.createElement(message, {children: 'Hello World'})
   ```

2. 🍎 JSX를 사용한 방식:
   ```jsx
   <message>Hello World</message>
   ```

이 두 코드는 동일한 결과를 생성합니다. 하지만 JSX 버전이 더 읽기 쉽고 HTML과 유사하여 이해하기 쉽습니다.

JSX를 사용할 때 주의할 점:

1. 컴포넌트 이름은 대문자로 시작해야 합니다:
   ```jsx
   <Message>Hello World</Message>
   ```
   소문자로 시작하면 React는 이를 내장 HTML 요소로 취급합니다.

2. 🍎 props 전달:
   ```jsx
   <Message someProps="value">Hello World</Message>
   ```
   이는 `React.createElement(Message, {someProps: "value", children: "Hello World"})`와 동일합니다.

3. 자식 요소:
   🟢JSX에서 태그 사이의 내용은 자동으로 'children' prop으로 전달됩니다.

4. 표현식 사용:
   ```jsx
   <Message>{1 + 1}</Message>
   ```
   중괄호 {}를 사용하여 JavaScript 표현식을 삽입할 수 있습니다.

5. 셀프 클로징 태그:
   ```jsx
   <Message />
   ```
   자식이 없는 경우 이렇게 작성할 수 있습니다.

🍎 JSX는 결국 `React.createElement` 호출로 변환됩니다. 이 변환 과정은 일반적으로 Babel과 같은 도구에 의해 수행됩니다. 따라서 JSX는 React 사용을 더 편리하게 만들어주는 문법적 설탕(syntactic sugar)이라고 볼 수 있습니다.


---
### Key Props id
key prop에 인덱스를 넣으면 remove, add시 순서를 보장하지 않게된다..
고유 id를 넣어야 함. 

exercise를 보면 id, value에서 value는 1초마다 셔플링시키는데
key prop에 index가 들어가있으면 
포커스가 바뀌지 않음.
-> '상태'라는 건 키보드 포커스 위치도 포함하는건데
그 상태까지 고유하게 식별되게 하려면
key prop을 고유 id로 해야함.
(인덱스는 그 위치에 누가와도 똑같은 1번째니까..)

<img width="922" alt="image" src="https://github.com/suji6707/suji6707.github.io/assets/111227732/3b717784-7744-4350-b4eb-7db78de6a752">

---
여기서 말하는 "상태(state)"는 컴포넌트의 데이터 상태 뿐만 아니라 사용자 인터페이스 요소의 상태, 즉 키보드 포커스 위치나 선택된 텍스트 등도 포함됩니다. key 프롭이 이러한 UI 상태에 어떤 영향을 미칠 수 있는지 살펴보겠습니다.

key 프롭의 영향 예시
예시 1: 인덱스를 key로 사용할 때의 문제점
```
function MyComponent({ items }) {
  return (
    <ul>
      {items.map((item, index) => (
        <li key={index}>{item.text}</li>
      ))}
    </ul>
  );
}
```
이 경우에는 items 배열의 순서나 내용이 바뀌면 리액트가 DOM을 적절히 업데이트하지 못할 수 있습니다. 예를 들어, 리스트 중간에 항목이 추가되면, 모든 이후 항목의 key가 변경되어 불필요한 재렌더링이 발생할 수 있습니다. 이는 성능 저하를 초래하며, 입력 필드 같은 요소에서는 포커스나 선택된 텍스트가 잘못되게 유지되는 문제를 발생시킬 수 있습니다.

예시 2: 고유 ID를 key로 사용할 때
```
function MyComponent({ items }) {
  return (
    <ul>
      {items.map(item => (
        <li key={item.id}>{item.text}</li>
      ))}
    </ul>
  );
}
```
각 항목의 고유 ID를 key로 사용하면, 아이템이 추가, 삭제, 재정렬 되어도 각 li 요소의 정체성이 유지됩니다. 따라서 리액트는 효율적으로 DOM을 업데이트할 수 있으며, 사용자의 텍스트 선택이나 입력 필드의 포커스 상태가 올바르게 유지됩니다.

---
## React Hook

이제 새로고침을 해도 상태값이 남아있도록 하고싶다.
local storage도 가능은 한데.

**useEffect의 호출 시점:**
1. 초기 마운트 시: 컴포넌트가 DOM에 처음 마운트 되었을 때, useEffect 내부의 코드가 실행됩니다. 이때, localStorage에 name 값을 저장합니다.
2. 컴포넌트 업데이트 시: name 상태가 setName 함수를 통해 변경될 때마다, 컴포넌트가 재렌더링 됩니다. 그 후 useEffect가 다시 실행되어 업데이트된 name 값을 localStorage에 저장합니다.

리액트에서는 useState를 통해 관리되는 상태(state)의 값이 변경되면 해당 상태를 사용하는 컴포넌트를 다시 렌더링하도록 설계되어 있음.
UI가 항상 최신 상태를 반영하도록 하기 위해.
value를 타이핑으로 바꿀때마다 컴포는트 전체 리렌더링이 됨

**useState의 호출 시점:**
- 일반적으로 useState를 사용할 때 초기값을 직접 지정하게 되면, 컴포넌트의 모든 렌더링에서 이 초기값이 평가됩니다. 예를 들어, useState(window.localStorage.getItem('name'))와 같이 사용하면, 컴포넌트가 렌더링될 때마다 localStorage에서 'name' 값을 가져오게 됩니다.
- 그러나 useState에 함수를 전달하면, 이 함수는 컴포넌트가 최초로 렌더링될 때 단 한 번만 호출되고, 그 결과가 상태의 초기값으로 사용됩니다. 

lazy initializer
- useState에 함수를 전달하면
컴포넌트가 처음 렌더링될 때 한 번만 실행되어 성능 개선.

```javascript
const [name, setName] = React.useState(
	// window.localStorage.getItem('name') || initialName -> 로컬스토리지 읽기 작업이 매번
	() => window.localStorage.getItem('name') || initialName // 처음에 한번만 
)
```

Q. 🍎 useState에 익명함수로 넣어도
value 바뀔때마다 리렌더링 일어나는건 똑같은데 이걸 개선할 수는 없는건가

