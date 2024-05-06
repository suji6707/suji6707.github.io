+++
title = 'Chess Game'
date = 2024-05-04T00:20:22+09:00
draft = true
+++
## 체스 게임 만들기 + Attlasian Draggable

### 목표
서버에 체스게임을 띄운다
- 드래그 기능 붙이기
- DNS 붙이기
	- IP(L4)는 같아도 Nginx로 uri 기반 라우팅만 달리하면 여러 사이트 띄울 수 있음.(L7) 

---
#### React vs Angular

함수형 프로그래밍은 객체지향과 반대.
함수형은 내부 상태를 갖지 말아야.
데이터를 받아서 데이터를 반환할 뿐. 

함수형 프로그래밍에서 가장 큰 관심사는 바로 프로그램의 정확성입니다. 프로그램이 정확하다는 건 어떤 프로그램을 실행했을 때 아무런 부수 효과(side effect)없이 모든 입력에 대해 정확한 결과를 반환한다는 것이고, 부수효과를 발생시키지 않는 방법은 프로그램은 실행 중간에 변경될 수 있는 자료구조를 사용하지 않는 것입니다.

💎 useState로 막아놔야 
새로 생성하지 않고 기존 값들을 재활용하도록 할 수 있음
- new constructor로 정의되어있으면
상태가 변경될 때마다 컴포넌트 전체가 리랜더링되어서
원치않는 백지화가 됨.
- useEffect( , []) 일때는 mount할 때 한번만 불리는거라
이 안에 new 와 같은 식으로 해도 되겠지만..

💎
아래 상황을 생각.
- C.name 로 직접 값을 변경하면 리액트 입장에선 그것이 바뀌었는지 알기가 힘듦. 알려면 딥하게 확인해야하는데 그러기엔 너무 컴포넌트가 많음..
- 따라서 State를 사용할 때는 사본을 써야. 
주소값이 바뀌면 확실하게 알 수 있기 때문
- 변경이 아니라 '갈아끼워야'하고
setName으로 바꿔야 한다는 것.
```
const [cname, setCname] = useState(cname)
C.name = y
setCname(y)  
```
- 이미 C.name에는 y가 들어가있는데
setCname의 인자인 y와 비교해 값이 같으면 
바뀌었다고 판단하지 않음..
- 처음부터 setCname으로 해야. 

현재 체스게임을 생각해보면
```typescript
// 기존 컴포넌트
export default function ChessBoardComponent = () => {
  const chessBoard = new ChessBoard();

  const updateBoard = (
  ): void => {
    // chessBoard 상태 변경
    chessBoard.move(prevX, prevY, newX, newY);
    setChessBoard(chessBoard);
    setPlayerColor(chessBoard.playerColor);
    setSafeSquares(chessBoard.safeSquares);
  };
}

// 사본으로 하도록 변경
export default function ChessBoardComponent() {
  const [chessBoard, setChessBoard] = useState<ChessBoard>(new ChessBoard());

  const updateBoard = (
  ): void => {
    // chessBoard 사본 생성 후 각각 필요한 상태값들 변경
    const newChessBoard = ChessBoard.copy(chessBoard);
    newChessBoard.move(prevX, prevY, newX, newY);
    setChessBoard(newChessBoard);

    setChessBoardView(newChessBoard.chessBoardView);
    setPlayerColor(newChessBoard.playerColor);
    setSafeSquares(newChessBoard.safeSquares);

```
사실 setChessBoard(newChessBoard) 만으로도 
변경된 상태의 보드가 chessBoard에 들어가므로
chessBoard.view, 
chessBoard.playerColor,
chessBoard.safeSquares 
이런식으로 접근해도 되긴하지만 불편하니까?
각 속성들을 useState로 일일이 업뎃시켜놓고
복잡한 로직에 사용하는듯.




---
#### React Children
children이란?
- 호출자의 Sauare 컴포넌트에 {}로 들어가는 요소가 
Square 컴포넌트의 {children}에 대입됨
- SquareProps에 정의한 필드는 호출자에서도 반드시 넣어줘야 함.

```typescript
function renderSquares(pieces: PieceRecord[]) {
	// ...
	squares.push(
	<Square location={squareCoord}>
		{piece && pieceLookup[piece.type]()}
	</Square>
	);

export function Square({ location, children }: SquareProps) {
	// ...
  return (
    <div
      css={squareStyles}
      style={{ backgroundColor: getColor(isDraggedOver, isDark) }}
      ref={ref}
    >
      {children}
    </div>
  );
```

onDragEnter에서 인자 속성으로 들어오는 source는 
Piece에 props로 속성값을 주어야 정상 작동함.
```typescript
useEffect(() => {
const el = ref.current;
invariant(el, "Invalid Element");

return dropTargetForElements({
	element: el,
	// 타겟이 드래그될 때마다 불림
	onDragEnter: ({ source }) => {
	// source is the piece being dragged over the drop target
	// ...
})


function renderSquares(pieces: PieceRecord[]) {
	// ...
      squares.push(
        <Square location={squareCoord} pieces={pieces}>
          {piece && pieceLookup[piece.type](squareCoord)}
        </Square>
      );

export const pieceLookup: {
  [Key in PieceType]: (squareCoord: Coord) => ReactElement;
} = {
  king: (squareCoord: Coord) => (
    <King location={squareCoord} pieceType={"king"} />
  ),
  //...
}

export function King({ location, pieceType }: PieceProps) {
  return (
    <Piece
      image={pieceImagePaths["K"]}
      alt="King"
      location={location}
      pieceType={pieceType}
    />
  );
}
```

---
#### NextJS setting
모든 건 기본에 있다.
생각하는 과정.

Install Tailwind CSS with Next.js
- npx tailwindcss init -p 를 해보니 
tailwind.config.ts
postcss.js 가 생김.
- 기존에 있던건 postcss.mjs인데
생각해보면 msj는 ESM이라 package.json에 `"type": "module"` 이 있어야하고
- 이게 없어도 tsconfig.json에서 `"module": "esnext",`가 모듈을 지원하도록 해주는 것 같지만.
- 어쨌든 postcss.cjs + type: commonjs
postcss.mjs + type: module
조합이라는 건 기본임..

사실 css가 작동을 안하던건 (그냥 css든 tailwind든)
pages router 가 하라는 설정을 안해서였음.

<img width="1217" alt="image" src="https://github.com/suji6707/insightLINK/assets/111227732/8fe94b7c-8b67-4724-8529-ecec79e35ea6">

pages 라우팅 컨벤션에 따르면
pages 폴더내 _app.tsx 두어야 한다는 것.
- Next는 페이지를 초기화하기 위해 `App` 컴포넌트를 사용
_app.tsx에서 리턴하는 `Component`는 active `page`를 뜻하며, 어떤 라우터로 가든 `Component`가 새로운 `page`로 바뀜을 의미. (page가 그 자리에 뜬다)


현재 page 라우터를 쓰고있으므로 app 폴더는 app 라우터에서처럼 리액트 컴포넌트들을 들고있는 역할이 아니다.
사실 chess-logic 은 전부 services에 해당 (MVC 패턴)


export default면 {}가 붙지 않음.
```typescript
// export const 함수
import { ChessBoardComponent } from "@/components/chess/ChessBoard"; 
// export default function 
import ChessBoardComponent from "@/components/chess/ChessBoard"; 
```

---

### 도메인 설정
🍋 현재 상황
도메인은 namecheap에서 구매했고
네임서버만 cloudflare로 연결하기.
- 가비아에서 도메인 사고 AWS Route53 네임서버에 연결하는 것과도 같음

domain transfer는 원하던 기능이 아니었음.
- 이건 namecheap에서 cloudflare로 도메인 관리를 완전히 이전한다는거고,
그래서 돈을 내야 pending이 없어지는거였음.
- 네임칩에서 클플로 이전시 auth code가 필요하고 domain LOCK을 풀어놔야 함
<img width="1214" alt="image" src="https://github.com/suji6707/insightLINK/assets/111227732/a56f66e8-65af-4217-aceb-bc11ddb9fe96">


왜 main.suzee.xyz을 만들었을까?
- 외부에서 접속할 때 suzee.xyz로 접속하면 프록시를 타게 되는데
내가 이 도메인으로 ssh 접속을 하려하면 난독화가 되어서 읽을 수가 없다고 함.
- 그래서 ssh로 접속하기 위한 용도로 DNS only 서브도메인을 만들어두는 것.
`ssh -i ~/auth/my-server.pem jisu@main.suzee.xyz`
-> 한 IP에 여러 서브도메인을 만들 수가 있구나?
- 나중에 서비스할 때 api.suzzee.xyz 등 서버용 하위 도메인도 파야함. 
-> 이것도 proxy여야 하나?
![image](https://github.com/suji6707/insightLINK/assets/111227732/1ecfff35-7e3d-4afe-ac7b-a98352079ae6)


Namecheap에서 도메인을 구매하면 Namecheap의 네임서버가 기본적으로 설정됨
- 사실 네임칩 네임서버만으로도 도메인에 대한 DNS 질의를 처리하고, 이를 통해 사이트에 연결할 수 있지만
- 추가적인 기능들(사이트 성능향상, DDoS 보안 등)을 위해 클라우드플레어 네임서버를 쓰는거임



A 레코드, CNAME이 뭔가?
왜 지금 DNS 체크할 때는 필요없는건가?



suzee.xyz vscode 연결?
그냥 git clone 해서 pm2로 띄워도 됨.


---
#### 계산이 너무 많다?
계산 자체는 문제가 아님. 네트워크 타는 요청들이 문제지.

10년 전에도
CPU 1코어당 3GHz. 
초당 30억번 계산(덧셈뺄셈).
'주기'. 한번의 파동에 걸리는 시간이 1/30억 초 
(Hz는 주기 역수)

모니터 주사율은 60Hz.
1초에 60장을 보여주면 자연스럽다는 것.

CPU 코어 Hz
1/60 안에만 계산할 수 있으면 됨.

30억번 / 60 = 5천만번.
그런데 CPU 메모리 접근시간이 200 사이클이므로 -
대략 메모리 레이어마다 1000배씩 차이난다 치고
1000사이클이라 보면

1초에 최소 5만번 계산은 
사람이 화면에서 보기에 충분히 자연스럽다는 것. (5만~30만)


---
리액트:
컴포넌트 정의할 때 export (default) function
컴포넌트 안에서 메서드 정의는 const 