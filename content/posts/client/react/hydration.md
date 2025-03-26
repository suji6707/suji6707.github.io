+++
title = 'Hydration'
date = 2024-12-09T00:20:22+09:00
draft = true
+++
## React Hydration

•SSR: 레스토랑에서 미리 요리된 음식을 포장해서 집으로 가져오는 것과 같습니다. 음식을 바로 먹을 수 있지만(빠른 초기 로딩), 음식을 데우거나 추가 재료를 넣는 등의 조작은 할 수 없습니다(상호작용 불가).
•Hydration: 포장된 음식을 전자레인지에 데우고, 소스를 추가하고, 필요한 조리 도구를 사용하는 것과 같습니다. 이제 음식을 원하는 대로 조작하고 맛있게 먹을 수 있습니다(상호작용 가능).

* 서버가 html을 서빙한다 -> 
CSR (Client-Side Rendering)
<script src="bundle.js"></script>
-> bundle.js는 리액트 코드.
이 js 파일을 다운로드, 실행해서 페이지 전체 내용을 동적으로 생성.
js 코드는 DOM을 조작하여 HTML 요소를 생성하고 데이터를 가져와 화면에 표시.

SSR의 경우
- 서버렌더링: React 컴포넌트를 사용하여 HTML을 생성(완성된 형태)
생성된 HTML과 js 번들을 같이 전송
- 클라에서 js가 다운되고 실행되면 브라우저는 hydration 과정을 통해 정적인 HTML에 이벤트 리스너를 연결하고 React 컴포넌트를 활성화하여 동적인 페이지로 만듦.
-> hydration이 없으면 정적인 페이지만 보여짐.


---
useMutation의 onError 옵션은 사이드 이펙트 처리 목적일 뿐
에러 자체를 변경하거나 수정하지 않습니다.

try...catch 블록에서 throw 하는 에러가 useMutation의 error 상태가 됩니다. 이것이 컴포넌트에서 사용할 수 있는 에러 객체입니다.

** 나이스페이 체크
    const register = async () => {
      if (isChangePayment) {
        await changeCard(customerUid);
      } else {
        await updateCard(customerUid);
      }
      setIsRegistered(true);
    };
