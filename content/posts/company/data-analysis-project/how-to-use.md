+++
title = 'Review Analysis'
date = 2024-12-09T00:20:22+09:00
draft = true
+++
## Review Analysis 구조


기본적인 기능들.

1. history 클릭하면 query/id로 이동한다
- 해당 url에서 쿼리를 보여주고 요청결과를 그린다.

2. 신규 요청
- url 이동

history든 new 쿼리든 /query/:id로 이동하고
id param이 바뀔때마다 fetchQuery 함수가 실행됨.

---
📕 setInterval:
- 시작 전에 항상 clear 해주고, distroy 전에 clear.
- 관심사 분리. 실행할 내용은 따로.

const fetchQueryTestInner = (requestId: string)  => {
    console.log(new Date());
    console.log(retryCount.value, requestId);
};

const fetchQueryTest = async (requestId: string) => {
    console.log('test');
    clearTimer();
    timerId.value = setInterval(() => {
        fetchQueryTestInner(requestId);
        retryCount.value += 1;
        if (retryCount.value >= 5) {
            clearTimer();
        }
    }, 3000);
};

onBeforeUnmount(() => {
    clearTimer();
});
