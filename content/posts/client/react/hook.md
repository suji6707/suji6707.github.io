+++
title = 'Hook'
date = 2024-12-09T00:20:22+09:00
draft = true
+++
## Hook


리뷰 컴포넌트와 훅의 연동 예시를 보여드리겠습니다:

```typescript
// src/hooks/useProductReviewExtract.ts
export const useProductReviewExtract = () => {
  const [reviewData, setReviewData] = useState([]);
  const [isLoading, setIsLoading] = useState(false);

  const handleDownloadExcel = useCallback((event) => {
    if (event.data.m === 'productReviewExtractResponse') {
      setReviewData(event.data.data);
      setIsLoading(false);
    }
  }, []);

  const requestReviewExtract = useCallback((productNo, channelNo) => {
    setIsLoading(true);
    window.parent.postMessage(
      {
        m: 'productReviewExtract',
        data: { productNo, channelNo },
      },
      `https://${currentHost}`
    );
  }, []);

  useEffect(() => {
    window.addEventListener('message', handleDownloadExcel);
    return () => {
      window.removeEventListener('message', handleDownloadExcel);
    };
  }, [handleDownloadExcel]);

  return { reviewData, isLoading, requestReviewExtract };
};

// src/components/ReviewAnalysis.tsx
const ReviewAnalysis = ({ productNo, channelNo }) => {
  const { reviewData, isLoading, requestReviewExtract } = useProductReviewExtract();

  useEffect(() => {
    // 컴포넌트가 마운트되면 리뷰 데이터 요청
    requestReviewExtract(productNo, channelNo);
  }, [productNo, channelNo, requestReviewExtract]);

  return (
    <div>
      {isLoading ? (
        <p>Loading...</p>
      ) : (
        <div>
          {reviewData.map((review, index) => (
            <div key={index}>
              {/* 리뷰 데이터 표시 */}
            </div>
          ))}
        </div>
      )}
    </div>
  );
};

// src/pages/ProductAnalysis.tsx
const ProductAnalysis = () => {
  const [selectedTab, setSelectedTab] = useState(0);
  const { data } = useProductData(); // 상품 데이터를 가져오는 훅이 있다고 가정

  return (
    <>
      <Tabs activeTab={selectedTab} setActiveTabIndex={setSelectedTab}>
        <Tabs.Tab label="상품 개요" />
        <Tabs.Tab label="리뷰 분석" />
        <Tabs.Tab label="상품 키워드" />
      </Tabs>

      {selectedTab === 1 && (
        <ReviewAnalysis 
          productNo={data?.productNo}
          channelNo={data?.channel?.naverPaySellerNo}
        />
      )}
    </>
  );
};
```

작동 과정:
1. `ProductAnalysis` 페이지에서 리뷰 분석 탭(selectedTab === 1)이 선택되면 `ReviewAnalysis` 컴포넌트가 마운트됩니다.

2. `ReviewAnalysis` 컴포넌트는 `useProductReviewExtract` 훅을 사용합니다.

3. `useProductReviewExtract` 훅이 초기화되면서:
   - 메시지 이벤트 리스너를 등록합니다.
   - `reviewData` 상태와 `isLoading` 상태를 관리합니다.
   - `requestReviewExtract` 함수를 제공합니다.

4. `ReviewAnalysis` 컴포넌트가 마운트되면 `useEffect`에 의해 `requestReviewExtract`가 호출됩니다.

5. 스크립트가 작업을 완료하고 메시지를 보내면:
   - 훅의 이벤트 리스너가 메시지를 감지합니다.
   - `handleDownloadExcel` 함수가 실행되어 `reviewData`를 업데이트합니다.
   - 상태 업데이트로 인해 컴포넌트가 다시 렌더링됩니다.

이렇게 하면:
- 관심사 분리: 데이터 처리 로직은 훅에, UI 로직은 컴포넌트에 있습니다.
- 재사용성: 훅은 다른 컴포넌트에서도 사용할 수 있습니다.
- 유지보수성: 이벤트 처리 로직이 한 곳에 모여있어 관리가 쉽습니다.

---

## 중복 문제

이런 상황이 발생하는 이유는 메시지 이벤트의 동작 방식 때문입니다.

1. 우선, 메시지 이벤트는 window 객체에 등록된 모든 이벤트 리스너에게 전달됩니다.

2. 현재 두 곳에서 같은 메시지('productReviewExtract')를 리스닝하고 있습니다:
   - 기존 훅의 이벤트 리스너
   - 새로 추가한 `handleReviewExtractMessage` 이벤트 리스너

3. `postMessage`를 보내면, 동일한 메시지 타입('productReviewExtract')을 감시하는 모든 리스너가 활성화됩니다.

해결 방법으로는:

1. 메시지 타입을 구분하는 방법:
```typescript
// 요청 메시지
window.parent.postMessage(
  {
    m: 'productReviewExtractRequest', // 요청용 메시지 타입
    data: {
      productNo: data?.productNo,
      channelNo: data?.channel?.naverPaySellerNo,
    },
  },
  `https://${currentHost}`
);

// 응답 메시지
const handleReviewExtractMessage = (event: MessageEvent) => {
  if (event.data.m !== 'productReviewExtractResponse') { // 응답용 메시지 타입
    return;
  }
  // ... 처리 로직
};
```

2. 또는 이벤트 리스너를 하나로 통합:
```typescript
// hooks/useProductReviewExtract.ts
export const useProductReviewExtract = () => {
  const [reviewData, setReviewData] = useState([]);

  const handleMessage = useCallback((event: MessageEvent) => {
    if (event.data.m === 'productReviewExtract') {
      if (event.data.error) {
        alert('리뷰 데이터 추출에 실패했습니다.\n 잠시후 다시 시도해주세요.');
        return;
      }
      setReviewData(event.data.data);
    }
  }, []);

  const requestExtract = useCallback((productNo, channelNo) => {
    window.parent.postMessage(
      {
        m: 'productReviewExtract',
        data: { productNo, channelNo },
      },
      `https://${currentHost}`
    );
  }, []);

  useEffect(() => {
    window.addEventListener('message', handleMessage);
    return () => {
      window.removeEventListener('message', handleMessage);
    };
  }, [handleMessage]);

  return { reviewData, requestExtract };
};
```

이렇게 하면 메시지 처리를 한 곳에서 관리할 수 있고, 의도하지 않은 중복 처리를 방지할 수 있습니다.


---

## 이벤트훅 vs 컴포넌트 역할

이 상황에서 이벤트 훅과 컴포넌트는 다음과 같은 역할을 수행하는 것이 좋습니다:

### 이벤트 훅 (`useProductReviewAnalysis`)

1. **데이터 요청 및 수신 관리**:
   - `requestReviewExtract` 함수는 `window.parent.postMessage`를 통해 리뷰 데이터를 요청합니다.
   - `handleAggregateReviews` 함수는 메시지 이벤트를 수신하고, 데이터를 처리하여 상태를 업데이트합니다.

2. **상태 관리**:
   - `reviews`, `isExtracting`, `isSuccess` 등의 상태를 관리하여 데이터 요청 및 수신의 진행 상태를 추적합니다.

3. **데이터 처리**:
   - 수신된 데이터를 가공하거나 분석하는 로직을 포함할 수 있습니다.

4. **재사용 가능성**:
   - 이 훅은 다른 컴포넌트에서도 재사용할 수 있도록 설계되어야 합니다.

### 컴포넌트 (`ProductReviewAnalysis`)

1. **UI 렌더링**:
   - 컴포넌트는 UI를 렌더링하고, 훅에서 제공하는 데이터를 기반으로 그래프를 그립니다.

2. **훅 호출 및 데이터 요청**:
   - 컴포넌트가 마운트될 때 `requestReviewExtract`를 호출 데이터를 요청합니다.
   - `useEffect`를 사용하여 컴포넌트가 마운트될 때 데이터를 요청합니다.

3. **상태에 따른 UI 업데이트**:
   - `isExtracting` 상태에 따라 로딩 스피너를 표시하거나, `reviews` 상태에 따라 그래프를 업데이트합니다.

4. **에러 처리**:
   - 데이터 요청 중 에러가 발생하면 사용자에게 알림을 표시합니다.

### 구현 예시

```typescript
// useProductReviewAnalysis.ts
export default function useProductReviewAnalysis({ currentUrl, ... }: Props) {
  const [reviews, setReviews] = useState<Optional<any[]>>(null);
  const [isExtracting, setIsExtracting] = useState(false);

  const requestReviewExtract = useCallback(() => {
    if (!mallPid || !productNo || !channelNo || !currentUrl || !currentHost || isExtracting) {
      return;
    }

    setIsExtracting(true);
    window.parent.postMessage(
      {
        m: 'productReviewExtract',
        target: 'reviewAnalysis',
        data: { productNo, channelNo },
      },
      `https://${currentHost}`
    );
  }, [mallPid, productNo, channelNo, currentUrl, currentHost, isExtracting]);

  useEffect(() => {
    const handleAggregateReviews = (event: MessageEvent) => {
      if (event.data.m !== 'productReviewExtract' || event.data.target !== 'reviewAnalysis') {
        return;
      }

      if (event.data.error) {
        alert('리뷰 데이터 추출에 실패했습니다.\n 잠시후 다시 시도해주세요.');
        setIsExtracting(false);
        return;
      }

      setReviews(event.data.data);
      setIsExtracting(false);
    };

    window.addEventListener('message', handleAggregateReviews);
    return () => {
      window.removeEventListener('message', handleAggregateReviews);
    };
  }, []);

  return { reviews, isExtracting, requestReviewExtract };
}

// ProductReviewAnalysis.tsx
import React, { useEffect } from 'react';
import useProductReviewAnalysis from './useProductReviewAnalysis';

export default function ProductReviewAnalysis({ currentUrl, ... }: Props) {
  const { reviews, isExtracting, requestReviewExtract } = useProductReviewAnalysis({ currentUrl, ... });

  useEffect(() => {
    requestReviewExtract();
  }, [requestReviewExtract]);

  return (
    <div>
      <h1>Product Review Analysis</h1>
      {isExtracting ? (
        <p>Loading...</p>
      ) : (
        reviews && (
          <div>
            {/* 그래프 컴포넌트로 리뷰 전달하여 렌더링 */}
            <GraphComponent data={reviews} />
          </div>
        )
      )}
    </div>
  );
}
```

### 설명
- **훅**은 데이터 요청과 수신을 관리하고, 상태 업데이트합니다.
- **컴포넌트**는 UI를 렌더링하고, 훅을 사용하여 데이터를 요청하며, 상태에 따라 UI 업데이트합니다.


---



