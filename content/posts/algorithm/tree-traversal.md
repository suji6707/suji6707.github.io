+++
title = 'My First Post'
date = 2024-02-25T00:20:22+09:00
draft = true
+++
## Tree Traversal

This is **bold** text, and this is *emphasized* text.

1. inorder 
말그대로 순서대로. Left < Data(root) < Right 순서
void로 됨.
root만 인자로 주면 (+순회대로 담을 배열)
참조배열에 차곡차곡 쌓임. (void)

2. preorder
루트먼저 찍음. Data - Left - Right

3. postorder
left, right 서브트리 전부 호출되고 나서
맨 마지막에 루트원소 찍힘
Left - Right - Data


**재귀 Tip**
리트코드에서 주어진 함수, 주어진 인자로 부족할 경우
helper 함수를 하나 더 만든다.

재귀함수 parameter의 경우
count를 세어서 쌓아올리는 경우 인자가 필요치 않음.
그때그때 합산해서 리턴해버리므로.
그러나 traversal 순서대로 배열에 담는 등 
최초 호출시점에서 준 인자를 계속 참조해야하는 경우 인자를 계속 넘기면서 재귀를 호출해야한다. 

즉, 참조를 해야하는 경우 인자가 필요하고 (보통 helper 함수 필요)
값복사 느낌으로 바로 연산해서 리턴하는경우 인자가 필요치 않다.

 
---



```typescript
const a: number = 1;
console.log(a) // 1
```

`a` is number
