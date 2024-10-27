+++
title = 'router'
date = 2024-09-4T00:20:22+09:00
draft = true
+++
## Vue 기초

🏞️ 추가 사항
- status failed일 때 재시도 로직


---
페이지 왼쪽과 오른쪽을 나누자.
- 사이드바 쿼리를 누르면 그 requestId로 dynamo에서 쿼리 및 결과를 가져온다.

### Passing props to Component?
defineProps, props: true
이건 좀 심화버전임.
url의 /:reqId를 props로 넘기는 방법 중 하나.

### Vue Router, route (기본)
router: 어디를 갈 건지
route: 현재 url 관련 정보를 가져오기

#### 🍀전체 구조
Vue Router는 URL을 담당한다. 싱글페이지에서 어느 컴포넌트로 이동할지.

App.vue를 보면 router-view가 전체 템플릿을 감싸고 있고,
그 안에 routes에서 정의한 컴포넌트들을 그리게 된다.

```js
<template>
  <router-view />
</template>

const routes = [
    {
        path: '/:requestId?',
        name: 'DashboardPage',
        component: DashboardPage
    }

const router = createRouter({
    history: createWebHistory(),
    routes
});
export default router;
```
createRouter로 router를 생성한 후 외부에서 사용할 수 있게.
그리고 그걸 express처럼 main.js에서 
최상단 app 컴포넌트에 적용한다.
```js
const app = createApp(App);
app.use(router);
```


---
### 현재 상황 정의
(클릭시 url이 변경되고,)
그 url에 담긴 requestId 정보를 컴포넌트에 전달해야한다.

동일한 컴포넌트이면 mounted 이벤트가 발생하지 않으므로 
Watch 이벤트를 등록해서 변경을 감지하도록 한다. 

그리고 watch는 처음 마운트가 아닌 다음부터 발동하게 되어있기 때문에(값이 변경될 때)
`immediate: true` 를 넣어서 마운트 즉시 반영하게 할 수 있음.

```js
import { useRoute } from 'vue-router';

const route = useRoute();

watch(() => route.params.requestId, (newRequestId) => {
    console.log(newRequestId);
    if (newRequestId) {
        // fetchQueryResult({ requestId: newRequestId });
    }
}, { immediate: true });
```
route를 생성해서 현재 url에 대한 정보를 얻는다.

---
### Watch
watch: 어떤 것에 반응할건지?
- (new, old) => { new value를 어떻게 쓸건지 정의 }


---
### router Link
네비게이션을 가능하게 하는 컴포넌트.
기본적으로 href를 갖는 `<a>` 태그로 렌더링됨.

어떻게 url을 변경하게 할 것인가?
- a태그
- button
  - button의 경우 당장 어떤 requestId를 담아 보낼지 알 수 없기 때문에 동적으로 구성해야함.
  - -> useRouter.push `"프로그래밍 방식 네비게이션"`
  - 뒤로가기 앞으로가기 다 스택이나 다름없음. 브라우저에 저장된 정보. 여기에 push를 해서 다음에 여기로 가라- 필요한 정보를 받으면.


---
#### 파라미터를 사용한 동적라우트 매칭

주어진 패턴에 따라 컴포넌트를 라우터로 매핑.
모든 유저에게 렌더링되어야 하는 User 컴포넌트가 있지만 각 유저별로 ID 경로가 다른 경우.

```js
routes = path: '/users/:id'
```
이제 ID가 달라도 같은 라우트에 매핑된다.
템플릿에선 {{ $route.params.id }}로 접근할 수 있다.


파라미터와 함께 라우트를 사용할 때 주의할 점은 
동일한 컴포넌트 인스턴스가 재사용된다는 것.
이는 생명주기 훅이 호출되지 않음을 뜻하므로,
Watch로 route.params와 같은 route 객체의 프로퍼티를 감시하면 된다.

```js
watch(
  () => route.params.id,
  (new, old) => {
    // 라우트 변경에 반응
  }
)
```


내 경우 
- 사이드바 쿼리를 클릭한다
- 해당 requestId로 url이 변경된다


---
### Props
🔺부모에서 값을 변경하면 자식컴포넌트에 반영.

부모에 inputText 설정해두고
그 값이 바뀌면 
QueryInput 컴포넌트에서 props로 inputText를 watch하고 있다가 감지해서
로컬 값(ref)을 바꿔 반영/렌더링한다.

결과창을 별도 컴포넌트로 분리하고 props, 이벤트 등을 설정해줄래?
Run 버튼을 누르면 inputText가 함수로 전달되고
그 값을 받아와서 자식컴포넌트(현재 query-result 클래스)로 전달.



---
### 이벤트

```js
// 이벤트 발생 (Component2.vue)
const c2method = () => {
  emitter.emit('component1')
}

// 이벤트 수신 (Component1.vue)
onMounted(() => {
  emitter.on('component1', c1method)
})
```