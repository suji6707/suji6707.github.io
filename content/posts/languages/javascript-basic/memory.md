+++
title = 'Memory'
date = 2024-10-21T00:20:22+09:00
draft = true
+++

🍎 얕은 복사 [... spread]
새로운 배열은 새로운 key-value 객체들을 참조하지만, key-value 객체 내부의 값들은 여전히 원본 Map 객체의 값을 참조.
- 즉, Map이 아닌 새로운 배열에 대한 포인터가 생긴 건 맞음. 
그런데 그 배열에는 복사된 값이 아니라 기존 Map에 있는 값들에 대한 참조가 들어있다고 보면 됨.
pointer array..
- 완전히 새로운 메모리 공간에 새로운 배열을 생성하고 싶다면 깊은 복사(Deep Copy)를 수행해야 합니다. 깊은 복사는 JSON.parse(JSON.stringify(attrCountMap))
-> 이렇게 쓰는 경우는 잘 없고(파라미터로 넘기는건 복사임), 그말은 즉슨 대부분 객체 다루는 일들이 얉은 복사, 참조라는 뜻임!
- 참고로 sort는 원본배열을 변경하고 그 배열에 대한 메모리 참조를 반환함.

const attrCountMap = new Map([['a', { count: 1 }], ['b', { count: 2 }]]);

// spread 연산자를 사용하여 새로운 배열 생성
const sorted = [...attrCountMap].sort((a, b) => 
  a[1].count > b[1].count ? -1 : 1 
);

// sorted 배열의 요소 값 변경
sorted[0][1].count = 100;

// 원본 Map 객체에도 변경 사항이 반영됨
console.log(attrCountMap.get('a').count); // 100

---