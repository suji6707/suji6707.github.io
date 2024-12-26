+++
title = 'FE 시스템'
date = 2024-11-10T22:50:44+09:00
draft = true
+++

---
## FE 시스템 
DAN24 발표

맞습니다. 템플릿과 컴포넌트 모두 속성을 사용하여 UI를 구성합니다. 하지만 두 개념은 사용 목적과 확장성 측면에서 차이가 있습니다.

### 템플릿
- **목적**: 주로 정해진 UI 구조에 데이터를 채워 넣는 데 사용됩니다.
- **특징**: UI 표현이 고정되어 있고, 데이터만 변경됩니다. 따라서 생산성이 높지만 확장성은 제한적입니다.
- **사용 예**: 뉴스 기사, 블로그 포스트 등 정형화된 데이터 표현.

### 컴포넌트
- **목적**: 재사용 가능한 UI 조각을 만들고, 다양한 상황에 맞게 조합하여 사용합니다.
- **특징**: UI 표현과 로직을 함께 조립할 수 있어 확장성이 높습니다. 다양한 props와 상태 관리가 가능합니다.
- **사용 예**: 버튼, 입력 폼, 모달 등 다양한 UI 요소.

### 차이점 요약
- **생산성 vs 확장성**: 템플릿은 생산성이 높고, 컴포넌트는 확장성이 높습니다.
- **고정 vs 유연**: 템플릿은 고정된 구조, 컴포넌트는 유연한 구조를 가집니다.

결국, 템플릿은 특정한 형태의 데이터를 빠르게 표현하는 데 유리하고, 컴포넌트는 다양한 UI 요구사항에 맞춰 확장 가능하게 설계됩니다. 프로젝트의 요구사항에 따라 적절한 방식을 선택하는 것이 중요합니다.

---

```jsx
import React from 'react';

// 모듈 타입 정의
interface ModuleProps {
  moduleId: string;
  // ... 모듈에 필요한 다른 속성들
}

// 각 모듈 타입에 대한 컴포넌트 정의
const BasicTitleModule: React.FC<ModuleProps> = ({ moduleId, title, titleHref }) => (
  <div>
    <h3>{title}</h3>
    <a href={titleHref}>바로가기</a>
  </div>
);

const FocusImageModule: React.FC<ModuleProps> = ({ moduleId, text, thumbSrc }) => (
  <div>
    <img src={thumbSrc} alt={text} />
    <p>{text}</p>
  </div>
);

// 모듈 렌더링 함수
const renderModule = (moduleData: ModuleProps) => {
  switch (moduleData.moduleId) {
    case 'basicTitle':
      return <BasicTitleModule {...moduleData} />;
    case 'focusImage':
      return <FocusImageModule {...moduleData} />;
    default:
      return null;
  }
};

const App = () => {
  // API 또는 다른 곳에서 받아온 데이터
  const modulesData = [
    {
      moduleId: 'basicTitle',
      title: '네이버 인기 토시',
      titleHref: 'https://m.naver.com',
    },
    {
      moduleId: 'focusImage',
      text: '다낭 이미지',
      thumbSrc: 'https://search.img.net/img.jpg',
    },
  ];

  return (
    <div>
      {modulesData.map((moduleData) => (
        <div key={moduleData.moduleId}>
          {renderModule(moduleData)}
        </div>
      ))}
    </div>
  );
};

export default App;
```

**설명:**

1. **모듈 타입 정의:** 각 모듈에 필요한 속성들을 정의합니다.
2. **모듈 컴포넌트 정의:** 각 모듈 타입에 대한 React 컴포넌트를 생성합니다.
3. **모듈 렌더링 함수:** `moduleId` 값에 따라 해당하는 모듈 컴포넌트를 렌더링합니다.
4. **데이터 기반 렌더링:** API 또는 다른 곳에서 받아온 데이터를 기반으로 `renderModule` 함수를 사용하여 동적으로 모듈을 렌더링합니다.

**장점:**

- **재사용성:** 각 모듈은 독립적인 컴포넌트이므로 재사용이 용이합니다.
- **유지보수 용이성:** 코드 구조가 명확해져 유지보수가 쉬워집니다.
- **확장성:** 새로운 모듈 타입을 쉽게 추가할 수 있습니다.

**참고:**

- 위 코드는 예시이며, 실제 구현은 프로젝트 요구사항에 따라 달라질 수 있습니다.
- `className` 속성을 사용하여 각 모듈에 스타일을 적용할 수 있습니다.
- `key` 속성은 React에서 리스트를 렌더링할 때 필수적으로 사용해야 합니다.

---

이 Flexible Rendering 구조를 React로 구현하는 예시 코드를 제공해드리겠습니다:

```jsx
import React from 'react';

// 모듈 컴포넌트들
const BasicTitle = ({ title, titleHref }) => (
  <h2><a href={titleHref}>{title}</a></h2>
);

const FocusImage = ({ text, thumbSrc }) => (
  <div>
    <img src={thumbSrc} alt={text} />
    <p>{text}</p>
  </div>
);

// Renderer 컴포넌트
const Renderer = ({ template, props, children }) => {
  const renderModule = (module) => {
    switch (module.moduleId) {
      case 'basicTitle':
        return <BasicTitle {...module} />;
      case 'focusImage':
        return <FocusImage {...module} />;
      default:
        return null;
    }
  };

  return (
    <div className={template}>
      {children.map((child, index) => renderModule(child))}
    </div>
  );
};

// 사용 예시
const App = () => {
  const data = {
    template: "ugcImage",
    props: {},
    children: [
      {
        moduleId: "basicTitle",
        title: "네트남 맛집 도사",
        titleHref: "https://m.naver.com"
      },
      {
        moduleId: "focusImage",
        text: "다낭 이미지",
        thumbSrc: "https://search.img.net/img.jpg"
      }
    ]
  };

  return <Renderer {...data} />;
};

export default App;
```

이 코드에서:

1. 각 모듈(`BasicTitle`, `FocusImage`)은 개별 React 컴포넌트로 정의됩니다.

2. `Renderer` 컴포넌트는 JSON 데이터를 받아 적절한 모듈을 렌더링합니다.

3. `renderModule` 함수는 `moduleId`에 따라 적절한 컴포넌트를 반환합니다.

4. `App` 컴포넌트에서는 서버로부터 받은 JSON 데이터를 시뮬레이션하고, 이를 `Renderer`에 전달합니다.

이 구조를 통해 서버에서 전달받은 JSON 데이터에 따라 유연하게 UI를 구성할 수 있으며, 새로운 모듈을 추가하거나 기존 모듈을 수정하는 것이 용이합니다.

