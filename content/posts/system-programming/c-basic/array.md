+++
title = 'C 배열'
date = 2025-02-28T00:20:22+09:00
draft = true
+++

### `int (*arr)[3]` 은 "가로 길이가 3인 2차원 배열을 처리하기 위한 포인터"
int arr[3]: arr은 int형 요소 3개를 가지는 1차원 배열
- arr 타입은 int*

int (*arr)[3]: arr은 int형 요소 3개를 가지는 1차원 배열을 가리키는 포인터.
arr 포인터 변수가 가리키는 대상은 int[3]
arr 타입은 int (*)[3] -> 2차원 배열 자체가 아니라, 2차원 배열의 첫번째 행을 가리키는 포인터.
- int (*arr)[3];는 메모리 공간을 할당하지 않습니다. 단지 int[3] 타입의 배열(또는 2차원 배열의 첫 번째 행)을 가리킬 수 있는 포인터 변수를 선언할 뿐

```c
int matrix[2][3] = { {1, 2, 3}, {4, 5, 6} };
int (*ptr)[3] = matrix; // 🟡ptr은 matrix의 첫 번째 행을 가리킵니다.

// ptr을 사용한 접근
ptr[0][0] = 10;  // matrix[0][0]을 10으로 변경
ptr[1][2] = 20;  // matrix[1][2]를 20으로 변경
```

---
### 2차원 배열
🟡 넘겨줄 때는 그냥 2차원배열 포인터면 됨.
```c
int apart[NUM_FLOORS][NUM_LINES];
for (int i = 0; i < NUM_FLOORS; i++) {
	for (int j = 0; j < NUM_LINES; j++) {
		scanf("%d", &apart[i][j]);
	}
}

int sum = count_people_on_floor(apart, i); // 🟡🟡🟡 포인터로

// 컴파일러는 메모리 크기를 알기위해 열의 수가 필요**. 행은 따로 줄거라서.
int count_people_on_floor(int array[][NUM_LINES], int floor) {
int sum = 0;
for (int i = 0; i < NUM_LINES; i++) {
	sum += array[floor][i];
}
return sum;
}
```

---
### 파라미터
int array[][NUM_LINES]로 받는 이유는 C언어의 2차원 배열 전달 규칙과 관련이 있습니다.

1. 2차원 배열 매개변수 전달 규칙
```c
// 다음 세 선언은 함수 매개변수로서 동일
int array[][NUM_LINES]
int (*array)[NUM_LINES]
int array[ANY_SIZE][NUM_LINES]
```

2. 컴파일러가 필요로 하는 정보
- 🟡2차원 배열에서 컴파일러는 각 행의 크기(열의 개수)를 반드시 알아야 합니다.
- 이는 메모리 주소 계산을 위해 필요합니다.
- 행의 개수는 알 필요가 없습니다.

3. 예시로 설명
```c
int apart[3][4] = {
    {1, 2, 3, 4},    // 4개의 열
    {5, 6, 7, 8},    // 4개의 열
    {9, 10, 11, 12}  // 4개의 열
};

// array[i][j]에 접근할 때
// 메모리 위치 = 시작주소 + (i * 열개수 + j) * sizeof(int)
```

5. 실제 사용 예시
```c
#define NUM_LINES 4

void process_floor(int array[][NUM_LINES]) {
    // array[0][0] ~ array[0][NUM_LINES-1] 접근 가능
    // 첫 번째 차원(행)의 크기는 알 필요가 없음
}

int main() {
    int apart[3][NUM_LINES];  // 3행 4열 배열
    process_floor(&apart[0]); // 첫 번째 층 처리
    process_floor(&apart[1]); // 두 번째 층 처리
    process_floor(&apart[2]); // 세 번째 층 처리
    return 0;
}
```

이러한 방식으로 선언하는 것이 C언어에서 2차원 배열을 함수 매개변수로 전달하는 표준적인 방법입니다.

---

첫번째 차원(행)의 크기를 비워둔 형태 -> 컴파일러는 이를 포인터로 해석
array[i][j]는 값이 되어버리므로!!
```c
void func(int array[][NUM_LINES]) {
    // 🟡array는 NUM_LINES 크기의 int 배열을 가리키는 포인터로 취급
}
```

가장 명시적인 선언. *array는 배열 포인터임을 나타냄.
```c
void func(int (*array)[NUM_LINES]) {
    // array는 NUM_LINES 크기의 int 배열을 가리키는 포인터
}
```

### Q. `void func3(int array[5][NUM_LINES]) { } // 5는 무시됨
왜 5가 무시되지? 5행 아닌가?`

이 질문은 C 언어에서 2차원 배열을 함수 인자로 넘길 때 겪는 흔한 혼란에서 비롯됩니다. 핵심은 **함수 인자로 배열을 넘기면 배열 전체가 복사되는 것이 아니라, 배열의 첫 번째 요소의 주소(포인터)가 전달된다는 점**입니다. 2차원 배열의 경우, "첫 번째 요소"는 "첫 번째 행"입니다.

**1. 배열 인자, 실제로는 포인터**

C 언어에서 함수에 배열을 인자로 넘길 때, 컴파일러는 배열 이름을 배열의 첫 번째 요소에 대한 포인터로 암묵적으로 변환합니다. 즉, `array[5][NUM_LINES]`로 선언된 인자는 함수 내부에서 `int (*array)[NUM_LINES]`와 동일하게 취급됩니다. 이는 "크기가 `NUM_LINES`인 `int` 배열을 가리키는 포인터"입니다.

**2. `5`의 의미와 무시**

* **선언 시 `5`의 의미:** 함수 선언 `void func3(int array[5][NUM_LINES])`에서 `5`는 프로그래머에게 "이 함수는 5행 이상의 2차원 배열을 받을 것으로 예상된다"라는 **정보를 제공하는 역할**을 할 수 있습니다. 즉, 코드의 가독성을 높이고, 함수가 특정 크기의 배열을 처리하도록 의도되었음을 나타낼 수 있습니다.
* **컴파일러가 `5`를 무시하는 이유:**
    * 🟡🟡 **크기 정보 불필요:** 함수 내부에서 `array[i][j]`와 같이 2차원 배열 요소에 접근할 때, 컴파일러는 **행의 수(`5`) 정보는 필요하지 않습니다.** (🟡진짜임. 몇번째 행인지가 필요한거지, 전체 행의 개수가 필요한 게 아님!! 그래서 몇번째 행인지는 두번째 인자로 따로 주는거임.)
		컴파일러가 주소 계산에 필요한 정보는 다음 두 가지입니다.
        1. **배열 시작 주소 (`array` 포인터):** 함수 인자로 전달됨.
        2. **각 행의 크기 (`NUM_LINES`):** 배열 선언 시 명시적으로 지정되어야 함 (`int array[][NUM_LINES]` 또는 `int (*array)[NUM_LINES]`).
    * **포인터 연산:** `array[i][j]`는 포인터 연산으로 해석됩니다.  실제 메모리 주소는 다음과 같이 계산됩니다.
        ```
        주소 = 배열 시작 주소 + (i * NUM_LINES * sizeof(int)) + (j * sizeof(int))
        ```
        여기서 행의 수(`5`)는 계산에 사용되지 않습니다. 중요한 것은 각 행이 `NUM_LINES`개의 열을 가진다는 정보입니다.
    * **유연성:** 함수가 특정 크기의 배열에만 종속되지 않도록 합니다. `func3` 함수는 5행 배열뿐만 아니라, 3행, 10행 등 `NUM_LINES` 열을 가진 어떤 크기의 2차원 배열도 받을 수 있습니다. 이는 함수의 재사용성을 높여줍니다.
