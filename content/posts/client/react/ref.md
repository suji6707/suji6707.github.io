+++
title = 'React Ref'
date = 2024-10-25T00:20:22+09:00
draft = true
+++
## React Ref

### useRef
DOM에 접근하는 방법

useRef는 current 프로퍼티를 가진 객체를 생성한다.
그리고 current 속성은 mutable하다. 
```javascript
function Tilt({ children }) {
  const tiltRef = React.useRef()

  React.useEffect(() => {
    const tiltNode = tiltRef.current
  }, [])

  return (
    <div ref={tiltRef} className="tilt-root">
      <div className="tilt-child">{children}</div>
    </div>
  )
}

```
이렇게 하면 tiltRef가 이 div 요소에 연결됩니다. React는 이 요소가 DOM에 마운트될 때 tiltRef.current를 해당 DOM 노드로 설정합니다.

useEffect 훅 내에서 tiltRef.current를 사용하여 실제 DOM 노드에 접근하고, 이를 VanillaTilt 라이브러리의 init 함수에 전달하여 tilt 효과를 초기화합니다.

따라서 ref는 명시적으로 할당되지 않았지만, React의 ref 시스템을 통해 자동으로 DOM 요소와 연결되어 사용됩니다.

---