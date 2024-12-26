+++
title = 'C 메모리 포인터'
date = 2024-12-18T00:20:22+09:00
draft = true
+++
## 메모리 포인터

---
sizeof(struct)는 각 필드 size 합과 다를 수 있다.
4byte or 8byte에 align하게 접근하게 하는 padding 때문에. 

필드 순서를 largest to smallest로 하는 것이 padding을 최소화할 수 있다.

포인터(char *name)는 8바이트(64bit)

printf("The address of age is: %p\n", &age);
- &로 변수의 주소에 접근할 수 있다. %p는 포인터 formatter.

long long : 8 byte (32/64 bit)
-> 주소 &는 4 byte 반환

### 가상메모리
프로그램을 실행하면 프로세스가 되고, OS로부터 일정 크기의 가상메모리를 부여받는다.
가상메모리 체제 덕분에
프로세스간 isolation이 되고, 개발자 입장에선 물리 메모리를 관리할 필요가 없다. 

인자로 struct를 넘기면 값복사가 일어나므로 원본이 안바뀜
-> 포인터를 넘겨야 참조해서 원본 값을 바꿈.

초기화
coordinate_t new_coord = {new_x, coord.y, coord.z};

pointer->field 로 참조.

"Dereferencing"은 간접 참조를 풀어서 실제 값을 얻는 것.
int x = 42;
int *p = &x;  // p는 x의 주소를 참조(reference)
*p;           // 참조를 해제(de-reference)하여 실제 값 42를 가져옴


배열의 이름은 배열 첫번째 원소를 가리키는 포인터로 역할한다.
int value = *(numbers + 2); -> 포인터에서 두 번째 인덱스에 대한 역참조(값)
포인터에 정수를 더하면 offset * sizeof type 의 위치로 간다. 이 역시 포인터.


---
## 💎Array Decay

배열이 크기 정보를 잃는 주요 상황들을 설명해드리겠습니다:

1. 함수에 매개변수로 전달될 때:
```c
void func(int arr[]) {  // 여기서 배열은 포인터로 decay
    sizeof(arr);        // 포인터 크기만 반환
}
```

2. 포인터에 할당될 때:
```c
int array[5] = {1,2,3,4,5};
int* ptr = array;           // 여기서 decay 발생
sizeof(ptr);               // 포인터 크기만 반환
```

3. 배열이 표현식에서 사용될 때:
```c
int array[5];
int value = *(array + 2);  // 배열이 포인터로 decay
```

반면, 배열 크기가 유지되는 경우:
```c
int array[5];
sizeof(array);              // 전체 배열 크기 유지
int (*ptr)[5] = &array;    // 배열에 대한 포인터
```

귀하의 코드에서:
```c
int main() {
    int core_utilization[] = {43, 67, 89, 92, 71, 43, 56, 12};
    sizeof(core_utilization);    // 여기서는 실제 배열 크기 (32)

    core_utils_func(core_utilization);  // 여기서 decay 발생
}

void core_utils_func(int core_utilization[]) {
    sizeof(core_utilization);    // 여기서는 포인터 크기 (8)
}
```

즉, 배열이 선언된 범위 내에서는 크기 정보를 유지하지만, 다른 컨텍스트로 전달되면 크기 정보를 잃게 됩니다.

---

