+++
title = '크레딧 충전 클라이언트 개발'
date = 2024-10-26T00:20:22+09:00
draft = true
+++

### 🚀 크레딧 충전 클라이언트 개발하면서 배운점들

🟢 로그인 라우팅 방법
* 결제버튼: 로그인모달
* 상세내역: /signin?redirectUrl=utility%2Ftranslate-image 
으로 라우팅
```
<Link href="/signin?redirectUrl=extension">
   <Button 
</Link>
```
이렇게 링크 걸어주거나
미들웨어에 라우터 전체 등록

---
해결
🟢 컴포넌트 리턴 전에 함부로 return null 하면 안됨.
아예 페이지가 안떠버리기 때문.
특히 로그인했을 때만 정보를 선택적으로 보여주고 
아니면 기본 페이지를 그려야하는 경우. 

if (!currentSetup) return null;

const { enabled = false, lowerBound: currentLowerBound = 0 } = currentSetup || {};
이렇게 빈객체로 두고 안의 필드에 기본값 넣어두고 처리하면 굿.

🔺로그인을 하지 않으면 그 API들이 'Error'로 온다는 것을 기억.
// 자동충전이 꺼져있거나 로그인하지 않은 경우 기본 화면 표시
{(!enabled || isSetupInfoError) && ( ...

🟢 클라에서 tinyint랑 boolean은 다름. 서버에서 boolean으로 바꿔서 줘야함
bool && (O) 
0 && (X) => '0'이 찍힘. 

🟢 enum은 Number 아니면 String 일관되게 
param 꺼내보니 '0' 스트링인데
'CREATED' enum string 이랑 비교하고있는 상황이었음

🟢 뭔가 useMutation 쓰면서 useState가 필요없어지는 느낌.
useMutation 응답 바뀌면 알아서 렌더링 하니까..

* 무한 렌더링 
- 컴포넌트가 렌더링될 때마다 setSelectedPlanId를 호출
- setState 호출은 새로운 렌더링을 트리거하므로, 다시 setSelectedPlanId 호출
  if (selectedIndex != null && creditPlans) {
    const planId = creditPlans[selectedIndex]?.id ?? null;
    setSelectedPlanId(planId);
  }

* 넘버형태 param 주의.
const paramIdx = Number(searchParams.get('index'));
Number(null) -> 이상하게 0으로 들어옴


🟢 이전 값에 대한 체크를 할 때는, 
변경하기 전에 체크해야한다. 메모리값 변경 전 tmp에 넣어두는 것 처럼..
get prevState 
check change
prevState.필드 변경

