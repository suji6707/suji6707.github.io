+++
title = '실전 CSS'
date = 2024-12-24T00:20:22+09:00
draft = true
+++
### 실전 CSS
서비스 만들면서 느낀 점들.

---
보이는 테두리 기준으로는 padding,
안의 것들 기준으로는 margin임.

---
Progressive bar
- 안의 내용물이 없으면(%), 부모의 width가 0이 되어버림.
부모 너비를 직접 지정해주고 자식의 h-full도. 

h, w 직접 지정하는 건 왠만하면 삼가
대부분 flex + padding/margin으로.
div 들의 너비는, 전체 영역 width - margin으로 자동 계산되는것임.

---
gap은 해당 부모 div에 flex 속성이 있어야만 적용됨!
flex flex-col 같이!

5줄 자르기
<p className="line-clamp-5">{getLongestReview(data.reviews)}</p>


---
## 클래스 컴포넌트 This vs 함수형 컴포넌트 UseRef
Chart.js의 ticks callback에서 전달되는 매개변수의 구조를 설명하자면:

```typescript
callback: (value: any, index: number, ticks: any) => {
  // ticks 객체의 주요 구조
  ticks = {
    $context: {
      index: number,      // 현재 tick의 인덱스
      tick: {             // 현재 tick 정보
        label: string,
        value: number
      },
      scale: CategoryScale,  // 스케일 정보
      chart: Chart,         // 차트 인스턴스
      ctx: CanvasRenderingContext2D  // 캔버스 컨텍스트
    },
    [[Prototype]]: Object   // 프로토타입 체인
  }
}
```

이미지에서 보이는 구조를 분석하면:
1. `$context`는 현재 tick의 컨텍스트 정보를 담고 있는 객체
2. 프로토타입 체인을 통해 scale, chart, ctx 등의 속성에 접근 가능
3. 계층 구조:
   - ticks
     - $context (tick별 고유 정보)
     - [[Prototype]] (공유되는 메서드와 속성)
       - scale
       - chart
       - ctx
       - options 등

이는 JavaScript의 프로토타입 상속을 활용한 구조로, 각 tick이 공통 기능은 프로토타입을 통해 공유하면서 개별적인 컨텍스트 정보는 `$context`에 저장하는 방식입니다.


this는 클래스 컴포넌트에서 사용되는 개념입니다. 함수형 컴포넌트는 클래스 컴포넌트와 달리 인스턴스를 생성하지 않기 때문에 this가 존재하지 않습니다.
-> useMemo와 별개로 걍 this 안되는듯?
🔥 Q. 매번 useRef로 참조해야하는가?

---
## 이벤트 버블링(Event Bubbling)
Chart.js에서 pointer-events: none이 flickering을 해결한 이유는 이벤트 버블링(Event Bubbling)과 관련이 있습니다.

**문제의 원인:**
1. Chart.js는 캔버스 위에 여러 레이어를 그립니다
2. 툴팁이나 호버 효과가 있을 때, 마우스 이벤트에 반응하여 캔버스를 계속 다시 그립니다
3. 이 과정에서 마우스 위치와 요소들 간의 충돌 감지(hit testing)가 계속 발생합니다
4. 특히 바의 경계선 부근에서는 마우스가 미세하게 움직일 때마다 이벤트가 발생하여 깜빡임이 생깁니다

### pointer-events: none의 역할
1. 해당 요소가 포인터 이벤트를 받지 않게 됩니다
2. 이벤트가 해당 요소를 "통과"하여 아래 레이어로 직접 전달됩니다
3. 결과적으로 불필요한 이벤트 처리와 리렌더링이 줄어듭니다

```javascript
// 예시: 문제가 발생할 수 있는 구조
<div class="chart-container">
  <canvas> // 실제 차트가 그려지는 캔버스
    // 여러 레이어들 (바, 축, 그리드 등)
    // 각 레이어마다 이벤트 처리가 발생할 수 있음
  </canvas>
  <div class="tooltip"> // 툴팁 레이어
</div>
```

**대안적인 해결 방법들:**
1. 이벤트 위임(Event Delegation) 사용:
```javascript
chartContainer.addEventListener('mousemove', (e) => {
  // 상위 레벨에서 한 번만 이벤트 처리
});
```

2. 이벤트 처리 최적화:
```javascript
const throttledHandler = throttle((e) => {
  // 이벤트 처리 로직
}, 16); // 약 60fps
```

3. 히트 영역(hit area) 조정:
```javascript
options: {
  elements: {
    bar: {
      borderWidth: 0, // 경계선 제거로 히트 테스트 단순화
    }
  }
}
```

pointer-events: none은 간단하고 효과적인 해결책이지만, 특정 상황에서는 원하는 이벤트 처리가 불가능할 수 있습니다. 이런 경우 위의 대안적인 방법들을 고려해볼 수 있습니다.


---
