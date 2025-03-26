+++
title = 'UseQuery'
date = 2024-12-09T00:20:22+09:00
draft = true
+++

## 장점
React Query 장점 vs useEffect 안에서 fetching
- 리액트 쿼리는 캐싱을 해주고 로딩/에러 자동 관리
- 동일한 쿼리에 대한 중복요청을 방지함

undefined가 뜨는 이유는 fetch data가 비동기 요청이라서. 

---
## UseQuery 에러처리

const {  currentSetup, isPending, isError, error } = useAutoChargeInfoQuery();

if (isPending) {
  return <div>자동 충전 정보를 불러오는 중...</div>;
}

if (isError) {
  return <div>자동 충전 정보를 불러오는 중 오류 발생: {error.message}</div>;
}

또는 안에서 캐치
useQuery({
  queryKey: ['todos', todoId],
  queryFn: async () => {
    const response = await fetch('/todos/' + todoId)
    if (!response.ok) {
      throw new Error('Network response was not ok')
    }
    return response.json()
  },
})

•렌더링 중 계산된 값이 곧바로 필요할 때: useMemo
•상태를 업데이트하거나 부수효과가 필요한 경우: useEffect


* useState 디폴트값 안에 함수를 넣을 수도 있다. param 값으로 바로 setState하는 경우.
const [activeTab, setActiveTab] = useState<TransactionFilterType>(() => {
  const filter = searchParams.get('filter');
  if (filter === 'charge') return TransactionFilterType.CHARGE;
  return TransactionFilterType.ALL;
});