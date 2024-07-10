+++
categories = ['Database']
series = ['Database-System']
tags = ['데이터베이스']
title = "B트리"
date = 2024-05-09T22:50:44+09:00
draft = true
+++
## B+트리 
공부 방법
- 삽입/삭제는 규칙 딱 적어놓고 (노드개수- 리프 ~n-1 vs nonleaf ~n 등)
직접 카드를 옮겨가면서 + 수도코드를 이해한다.
- 지금은 일단 개념이 우선이라서. B트리 확장부터?

### B+트리 인덱스 파일
인덱스 순차파일의 주요 단점: 파일이 커질수록 인덱스를 찾아서 그 데이터를 연속으로 스캔하는 성능이 감소하는 것.
B+트리 인덱스 구조는 삽입/삭제에도 성능을 유지한다.
트리 루트에서 leaf까지 모든 경로의 길이가 같은 balanced tree.
tree에서 non-leaf노드는 |n/2|(올림)와 n 사이의 자식을 갖고 있고
루트는 2~n개 사이의 자식을 갖는다.

#### B+트리의 구조
다계층 인덱스이지만 다계층 인덱스 순차파일과는 다름.
고유키를 가정.

🔹 Leaf Node
n-1개의 검색 키값과 n개의 포인터.
검색키값은 정렬된 순서.
Pointer는 최종 row를 가리킴.

🔹 NonLeaf Node
다계층 Sparse Index를 형성. 
all pointers are pointers to tree nodes 라는 점이 리프노드와 차이점.
- P1이 가리키는 모든 검색키는 K1보다 작다.
- Pi가 가리키는 검색키는 Ki-1보다 같거나 크다
Ki-1 <= Pi < Ki

#### B+트리 쿼리
수도 코드
- 루트노드에서 시작해 일부 리프노드까지 트리 경로 탐색
- 실제 구현은 iterator. next를 통해 연속적인 레코드를 가져옴. 엔트리를 하나씩 검사하는 건 똑같은듯.

#### B+ 쿼리 비용
- 한 노드는 보통 disk block 사이즈와 같음.(4KB)
- n 은 대략 100개 (40 bytes per index entry) >> 노드당 100개 인덱스 엔트리 저장. 실제 레코드 row가 아니라 엔트리(주소, 오프셋, 검색키값을 포함)
- 1 million search key values and n = 100
	- at most log50(1M) = 4 노드만 거치면 찾을 수 있음.
		- 여기서 n/2 = 50 -> 한 노드가 최소 50개~100개 엔트리.
	- 반면 binary tree는 2^20은 해야 1M임. vs 50^3~4 = 1M
- 이러한 차이는(노드개수?) 실로 엄청나다. 왜냐하면 모든 노드 액세스는 disk I/O를 요하며 20ms 정도의 시간이 들기 때문이다..

```
n은 한 노드가 가질 수 있는 최대 자식 노드 수 또는 최대 값의 수를 의미합니다. 전체 레코드 개수가 아닙니다.
 B+ 트리의 특성에 따르면:
 •루트 노드나 리프 노드가 아닌 노드는 n/2개에서 n개 사이의 자식 노드를 가집니다.
 •리프 노드는 (n-1)/2개에서 n-1개 사이의 값을 가집니다.
 
앞서 예시에서 n = 100이었던 것은 한 노드가 최대 100개의 엔트리(자식 노드 포인터 + 값)를 가질 수 있다는 뜻입니다. 그래서 리프 노드는 최소 (100-1)/2 = 49개, 최대 99개의 값을 가지게 되는 것입니다.
```

```groovy
function findRange(lb, ub)
/* Return all records with search key value B such that lb <= V <= ub */
	Set resultSet = {};
	Set C = root node
	while (C is not a leaf node) begin
		Let i = smallest number such that lb <= C.Ki
		if there is no such number i then begin
			Let Pm = last non-null pointer in the node
			Set C = C.Pm
		end
		else if (lb = C.Ki) then Set C = C.Pi+1
		else Set C = C.Pi   // lb < C.Ki
	/* C is leaf node */
	Let i be the least value such that Ki >= lb
	if there is no such i
		then Set i = 1 + number of keys in C; // 다음 리프로 이동
	Set done = false;
	while (not done) begin
		Let n = number of keys in C
		if (i <= n and C.Ki <= ub) then begin  // 해당 노드 안에서의 i = 1~n ?
			Add C.Pi to resultSet
			Set i = i + 1 // 반복문
		end
		else if (i <= n and C.Ki > ub) 
			then Set done = true; // while 종료
		else if (i > n and C.Pn+1 is not null)
			then Set C = C.Pn+1, and i = 1 // 다음 리프노드까지 살펴봐야 함.
		else Set done = true;  // no more leaves to the right
	end
	return resultSet;
```

### Non-Unique Keys
복합 인덱스를 생성.
name, ID를 생각.
kim, yoon 등은 비고유 키이므로 ID를 붙여서 인덱스를 생성하는데.
kim을 가진 모든 튜플을 검색하려면,
findRange(lb, ub)
- lb = (kim, -INF)
- ub = (kim, +INF)

키 비교 연산
- 단일 키: lb <= C.Ki와 C.Ki <= ub 비교
- 복합 키: (kim, -∞) <= C.Ki와 C.Ki <= (kim, +∞) 비교

위 방식에 대한 대안?
- buckets on seperate block (별로.)
	- 장점: 검색 키에 대한 모든 레코드를 한 번에 가져올 수
	- 단점: "별도의 블록"에 저장된 버켓에서 레코드를 가져오는 데 추가적인 I/O 작업이 필요
```c
데이터: [(kim, 1), (kim, 2), (lee, 1), (park, 1), (kim, 3)]
버켓 구조: kim이라는 키를 가진 레코드들을 하나의 버켓에 모은다.

검색키: "kim"
버켓: [ (kim, 1), (kim, 2), (kim, 3) ]
```
- 각 키에 튜플 포인터 리스트를 사용
	- 각 검색 키에 대해 해당 키를 가진 레코드들의 포인터 리스트를 유지
	- 장점: 공간 오버헤드가 적고, 검색 시 추가 비용이 들지 않음
	- 단점: 긴 리스트를 처리하는 추가 코드가 필요. 삭제 작업 시 리스트에서 포인터를 제거하는 것이 복잡할 수.
```c
검색 키: "kim"
// kim"이라는 키를 가진 레코드들의 포인터 리스트를 유지
포인터 리스트: [ pointer to (kim, 1), pointer to (kim, 2), pointer to (kim, 3) ]
```

- 레코드 식별자를 추가해 검색키를 고유하게 만들기
```c
검색 키: (kim, 1), (kim, 2), (kim, 3)
```
\
 
### Update om B+Trees: Insertion
pr: pointer to record
v: search key value of the record

find the leaf node in which search key value would apear.
- if there is room in leaf node, insert (v, pr) pair
- otherwise, split the node (along with new (v,pr) entry), and propagate update to parent nodes.

노드 분할
- take  the n(search key value, ptr) pairs(삽입할 엔트리 포함) in sorted order.
- 첫 |n/2| 오리지널 노드를 앞에 두고 나머지는 new node에. 
- new node가 p, p에서 가장 작은 키값을 k라고 하자. 
	- (k,p)를 부모 노드로 올리는데, not full 부모노드가 나올때까지.
	- 최악의 경우 루트노드가 반으로 쪼개지고 찐 루트가 하나 더 생겨서 height += 1이 된다.
