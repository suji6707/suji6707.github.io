+++
title = '바닐라 자바스크립트'
date = 2024-09-04T00:20:22+09:00
draft = true
+++
### 바닐라 자바스크립트

자바스크립트로 html document에 접근할 수 있다.
js로 html doc을 변경하고 읽는다.

document.getElementById나 ByClassName과 달리,
querySelector는 CSS selector 처럼 가져올 수 있다.
#(id), .(class), ' ' child, > 등만 잘 구분해준다면.  

---
#### 이벤트
어떤 element를 가져와서! 거기에 이벤트를 등록한다.
<Component onClick= > 

이름앞에 on이 붙어있으면 이벤트 리스너다.


```js
// 특정 요소를 선택 (id가 'myButton'인 버튼)
const myButton = document.getElementById('myButton');

// 버튼에 'onclick' 이벤트 리스너 추가
myButton.onclick = function() {
    alert('Button was clicked!');
};

// 비교
function handleClick() { 
}
myButton.addEventListener('click', handleClick);
```

참고.
`loginForm.addEventListener('submit', onLoginSubmit);`
여기서 onLoginSubmit()을 하면 브라우저가 보자마자 함수를 실행시켜버림.
함수이름만 넣으면 지정한 이벤트가 발생했을 때만 함수를 실행.


