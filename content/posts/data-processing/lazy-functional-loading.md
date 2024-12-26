+++
title = '함수형 프로그래밍'
date = 2024-12-16T22:50:44+09:00
draft = true
+++
## Lfi / pipelining

lfi의 파이프라인 처리 방식은 단일 for 루프처럼 동작하지만, 약간 다릅니다. 각 아이템이 파이프라인을 "흐르듯이" 처리되며, 각 단계가 독립적으로 병렬 처리됩니다.

작동 방식을 시각화해보면:

```javascript
// 입력 데이터 스트림
[log1] -> [log2] -> [log3] -> [log4] -> ...

// 파이프라인 처리
log1: filter -> map -> reduce
log2:    filter -> map -> reduce
log3:       filter -> map -> reduce
log4:          filter -> map -> reduce
```

**주요 특징:**

1. **스트리밍 처리:**
- 각 로그 항목이 도착하자마자 처리를 시작
- 전체 데이터를 기다리지 않음

```javascript
// 실제 동작 예시
const processLogs = await pipe(
  fromStream(logStream),
  filterConcur(async log => {
    // log1이 여기서 처리되는 동안
    // log2도 이미 스트림에서 읽혀서 처리 시작 가능
    return log.type === 'ERROR';
  }),
  mapConcur(async log => {
    // log1이 여기 도달했을 때
    // log2는 filter 단계에 있을 수 있음
    return transformLog(log);
  })
);
```

2. **병렬 처리:**
- 여러 항목이 동시에 다른 단계에서 처리될 수 있음
- 각 단계는 독립적으로 작동

```javascript
// 동시성 제어 예시
const processLogs = await pipe(
  fromStream(logStream),
  filterConcur(async log => log.type === 'ERROR', {
    concurrency: 3 // 최대 3개의 필터링 작업을 동시에 실행
  }),
  mapConcur(transformLog, {
    concurrency: 2 // 최대 2개의 변환 작업을 동시에 실행
  })
);
```

3. **메모리 효율성:**
- 각 항목은 파이프라인을 통과한 후 메모리에서 해제됨
- 중간 배열이 생성되지 않음

```javascript
// 기존 방식 (메모리에 중간 배열 저장)
const allLogs = await fetchLogs();        // [log1, log2, log3, ...]
const errorLogs = allLogs.filter(...);    // [error1, error2, ...]
const transformedLogs = errorLogs.map(...); // [transformed1, transformed2, ...]

// lfi 방식 (중간 배열 없음)
const result = await pipe(
  fromStream(logStream),
  filterConcur(...),  // 각 항목이 개별적으로 처리됨
  mapConcur(...)      // 필터링된 항목이 바로 변환됨
);
```

4. **백프레셔 처리:**
- 시스템 리소스에 따라 처리 속도 자동 조절
- 메모리 사용량 제어

```javascript
const processWithBackpressure = await pipe(
  fromStream(logStream),
  filterConcur(async log => {
    // 시스템 부하에 따라 처리 속도 자동 조절
    return log.type === 'ERROR';
  }, {
    concurrency: 5,    // 동시 처리 제한
    bufferSize: 1000  // 버퍼 크기 제한
  })
);
```

따라서, lfi는 단순한 for 루프보다 더 효율적인 방식으로 데이터를 처리합니다. 각 항목이 독립적으로 처리되면서도 병렬 처리의 이점을 활용하고, 메모리 사용을 최적화할 수 있습니다.

---

### 일반 js 실행과 비교

일반 JavaScript의 배열 메서드와 Promise.all을 사용할 때의 처리 방식을 시각화해보겠습니다:

**1. 일반 JS의 filter -> map -> reduce:**
```javascript
const result = logs
  .filter(log => log.type === 'ERROR')
  .map(transformLog)
  .reduce(accumulator);
```

시각화:
```
// 1단계: filter (전체 배열을 순회)
[log1, log2, log3, log4]
↓ ↓ ↓ ↓ (순차적으로 필터링)
[log2, log4] (필터링된 새 배열 생성)

// 2단계: map (필터링된 배열 전체를 순회)
[log2, log4]
↓ ↓ (순차적으로 변환)
[transformed2, transformed4] (변환된 새 배열 생성)

// 3단계: reduce (변환된 배열 전체를 순회)
[transformed2, transformed4]
↓ ↓ (순차적으로 누적)
final_result
```

**2. Promise.all을 사용한 병렬 처리:**
```javascript
// filter
const filteredLogs = await Promise.all(
  logs.map(async log => {
    const shouldKeep = await asyncFilter(log);
    return shouldKeep ? log : null;
  })
).then(results => results.filter(Boolean));

// map
const transformedLogs = await Promise.all(
  filteredLogs.map(async log => await transformLog(log))
);

// reduce
const result = transformedLogs.reduce(accumulator);
```

시각화:
```
// 1단계: Promise.all + filter
[log1, log2, log3, log4]
↓    ↓    ↓    ↓ (동시에 필터링 시작)
[    ↓    log2  ↓   log4] (필터링된 새 배열)

// 2단계: Promise.all + map
[log2, log4]
↓    ↓ (동시에 변환 시작)
[transformed2, transformed4] (변환된 새 배열)

// 3단계: reduce
[transformed2, transformed4]
↓ ↓ (순차적으로 누적)
final_result
```

**주요 차이점:**

1. **메모리 사용:**
   - 일반 JS: 각 단계마다 새로운 배열이 생성됨
   - Promise.all: 각 단계마다 새로운 배열이 생성됨
   - lfi: 중간 배열이 생성되지 않음

2. **처리 순서:**
   - 일반 JS: 완전히 순차적 처리
   - Promise.all: 각 단계 내에서는 병렬 처리, 단계 간에는 순차적🟡
   - lfi: 각 항목이 독립적으로 파이프라인을 통과🟡
-> 큰 차이!!

3. **성능 특성:**
```javascript
// 일반 JS - 순차 처리
console.time('regular');
const result1 = logs
  .filter(expensiveFilter)
  .map(expensiveTransform)
  .reduce(accumulator);
console.timeEnd('regular');

// Promise.all - 단계별 병렬 처리
console.time('promise.all');
const filtered = await Promise.all(logs.map(expensiveFilter));
const transformed = await Promise.all(filtered.map(expensiveTransform));
const result2 = transformed.reduce(accumulator);
console.timeEnd('promise.all');

// lfi - 완전 병렬 파이프라인
console.time('lfi');
const result3 = await pipe(
  asConcur(logs),
  filterConcur(expensiveFilter),
  mapConcur(expensiveTransform),
  reduceConcur(accumulator)
);
console.timeEnd('lfi');
```

4. **에러 처리:**
```javascript
// 일반 JS
try {
  const result = logs
    .filter(log => {
      // 하나라도 에러나면 전체 중단
      return someFilter(log);
    })
    .map(transformLog);
} catch (error) {
  // 전체 처리가 중단됨
}

// Promise.all
try {
  const results = await Promise.all(logs.map(async log => {
    // 하나라도 실패하면 전체 Promise.all이 실패
    return await processLog(log);
  }));
} catch (error) {
  // 전체 처리가 중단됨
}

// lfi
const results = await pipe(
  asConcur(logs),
  mapConcur(async log => {
    try {
      // 개별 항목의 실패가 전체에 영향을 주지 않음
      return await processLog(log);
    } catch (error) {
      return { error, log };
    }
  })
);
```

이러한 차이로 인해:
- 일반 JS: 간단한 동기 처리에 적합
- Promise.all: 단순한 비동기 병렬 처리에 적합
- lfi: 복잡한 데이터 파이프라인과 스트리밍 처리에 적합