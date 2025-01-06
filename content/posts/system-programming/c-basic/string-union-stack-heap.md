+++
title = 'Boot.dev'
date = 2024-12-28T00:20:22+09:00
draft = true
+++
# String
- 포인터와 \0 null character가 중요.
시작과 끝만 알고 나머지는 모름
- 사이즈 정보를 저장하지 않으므로 버퍼 오버플로우를 조심해야. 
- strlen은 마지막 \0 직전까지 length를 센다.
- char buffer[64] 라고 했을 때, 63개까지만 넣을 수 있다. 

# Union Type
### 파이썬의 동적 변수. 
- 런타임에 타입 변경 되고 x = 10 후 x = 'hello' 
- 모든 것이 객체인 언어.
    - 파이썬과 JavaScript에서는 문자열이 객체이기 때문에 메서드를 가질 수 있으나
C는 단순히 메모리에 있는 char 배열이라 문자열 조작하려면 별도 함수들을 사용해야 함. 

union: 모든 필드가 같은 메모리를 씀. 마지막으로 설정한 필드 타입으로만 접근해야.
- int를 float나 store로 읽으려하면 이상한 값이 됨.

signed vs unsigned int
- signed: 최상위 비트는 부호비트, unsigned에서는 모든 비트를 값을 해석
- 최상위가 1인 음수를 읽을 때는, 나머지 31비트를 반전시킨 후 1을 더하고 앞에 -를 붙임.
이 방식은 CPU가 음수와 양수를 동일한 방식으로 처리할 수 있게 해준다. 

unsigned: 32비트를 전부 값으로 읽으면 1이 32개, 42억9천..
signed: 최상위 1이 부호, 나머지 0으로 반전, + 1 = -1.


int raw[8] 와 uint8_t raw[8]은 다름
앞은 4*8 * 8, 뒤는 8 * 8 ?

# Stack
스택은 로컬 변수가 저장되는 곳. 함수가 호출될 때마다 새로운 스택프레임(함수 params, local 변수들을 담은)이 생겨난다.
함수가 끝나지 않고 계속 스택프레임이 생성되는 상황
-> stack pointer가 계속 증가함. 

void printStackPointerDiff() {
  static void *last_sp = NULL;
  void *current_sp;
  current_sp = __builtin_frame_address(0);
  long diff = (char*)last_sp - (char*)current_sp;
  if (last_sp == NULL){
    last_sp = current_sp;
    diff = 0;
  }

last_sp는 첫 스택프레임 주소로 고정, curr_sp만 계속 낮은 주소로 이동함.
-> 함수 내부에 선언된 static 변수: 
함수가 여러 번 호출되더라도 해당 변수는 한 번만 초기화되며, 함수 호출 간에 값을 유지

* 스택에 저장되는 것.
return address - 반환되면 어디로갈지
stack pointer
argument - 메모리 복사가 일어남

* 스택의 장점
메모리가 연속적으로 배치됨 (힙은 그렇지 않음)
함수가 반환되면 없어지기에 자동으로 메모리 관리가 됨.
thread safety. 각 스레드는 자신의 스택을 가지므로 힙 처럼 동기화 메커니즘이 필요없음.
-> Go는 왠만하면 스택에 변수를 저장하지만 파이썬은 대부분의 객체를 힙에 저장한다
(파이썬은 모든것이 객체, 객체는 힙에 저장되어야 메모리 관리가 용이
파이썬의 함수 호출 스택은 스택 프레임을 사용하지만, 실제 데이터는 힙에 저장된 객체를 참조)

함수 반환값은 &c 같은 참조가 되면 안됨.
이미 스택프레임이 사라진 후 안에있던 변수를 참조하게 되므로 사실상 undefined인데 가장 최근 함수의 변수로 덮어쓰기 되는듯.
일반 구조체 c를 반환하면 메인함수로 복사가 일어나서 memory safe 접근 가능함.

c는 name, address, value 가 전부다.


* 스택 메모리에 할당된 변수를 가리키는 포인터를 반환하는 것은 위험
함수가 종료되면 이 함수가 반환하는 포인터는 더 이상 유효하지 않음.
-> 함수 종료후에도 써야하면 '힙'에 할당!
char *get_full_greeting(char *greeting, char *name, int size) {
  char full_greeting[100];
  snprintf(full_greeting, 100, "%s %s", greeting, name);
  return full_greeting;  // 위험! 스택 메모리의 주소를 반환
}


# 힙
동적 메모리. as needed에 따라 할당/해제.
직접 해제해줘야 하는경우 까먹을 위험.

컴파일 타임에 사이즈를 알 수 없는 경우 힙에 할당. (런타임에서야 정확한 크기 알 수)

malloc으로 할당하면 그 주소는 스택에 저장되나 실제 데이터는 힙에 있다. 그걸 가리키는 주소일뿐.

free를 해도 포인터는 남아있고, 해제된 메모리를 가리킨다.
- 즉 메모리에 저장된 값은 변하지 않으며, 단지 OS에 그 공간을 사용할 수 있다는것만 알려준다.

# 포인터
## 이중포인터 pointer-pointers
어떤 값을 가리키던 포인터를 다른 값을 가리키게 할 수 있음.

함수 인자로 &참조를 넘겼을때, 그 참조 array를 deep copy하려면 이중포인터 배열을 생성해서(malloc) 그 안에 각 원소를 malloc으로 할당해 넣어준다.
JS와 비교해보면 
JS의 경우 스프레드 연산자(...)는 shallow copy임. 최상위 속성만 복사하고 중첩된 객체나 배열은 여전히 원래 객체를 참조.
JSON.stringify(obj)를 해야 deep copy


## swap param
C의 함수 호출 방식(call by value) 
함수에 전달되는 포인터도 '값으로 전달'. 즉, 포인터의 복사본이 전달.
a = b; b = ptr; 이런식으로 포인터값만 스왑하는건 의미없음

void swap_ints(int *a, int *b) {
    int temp = *a;    // 포인터가 가리키는 실제 값을 조작
    *a = *b;
    *b = temp;
}

int a = 5;
int b = 6;
swap_ints(&a, &b);
-> Call by Reference: 이 함수 안에서 a, b의 값을 바꾸는거임. 포인터 인자는 콜스택 빠져나오면 없어지고.

### generic(void) pointer swap의 경우
*ptr1 = *ptr2 만으로는 안됨 (string은 **라서 그런가)
memcpy(dest, src, size)로 해야함.
-> size는 개수 * 각 sizeof

---
# 스택 자료구조
제너릭(void *)에 대해 더 깊이 알아보자.

any가 void*임.
```c
// any type == void *, any array is what?
// 1 element is void* type
void **ptr = malloc(capacity * sizeof(void*));
```
-> char 배열 (char **str)과 비슷. 

stack->data = any[];


* 아무거나 stack->data에 들어가는 상황
we can push both int and int * types into the same stack (again, a bad idea).
```c
void scary_double_push(stack_t *s) {
  // 원치않은 push. 1337이라는 숫자 자체가 포인터로 캐스팅되어 push됨.
  // void*가 any지만 그보다는 int*, char*이 나음.
  stack_push(s, (void*)1337);
  // 의도된 것. 1024라는 값을 가리키는 주소가 push됨.
  int *ptr = (int*)malloc(sizeof(int));
  *ptr = 1024;
  stack_push(s, ptr);
}
```

---
```c
void stack_push_multiple_types(stack_t *s) {
  float *float_ptr = (float*)malloc(sizeof(float));
  *float_ptr = 3.14;

  stack_push(s, float_ptr);

  //🔴 string 타입이 뭐지? char* 
  
  char *src = "Sneklang is blazingly slow!";
  char *char_ptr = (char*)malloc(sizeof(char) * (strlen(src) + 1));
  strcpy(char_ptr, src); //🔴 메모리에 넣는거라
  stack_push(s, char_ptr);
}

* memcpy를 쓰는 상황
- struct 복사시 (원시타입만 *ptr = 단순대입으로 가능)
- void*로 작업시 크기 정보가 없어서 직접 대입 안됨.

# Object
.h에 선언된 객체 초기화시
malloc 으로 할당 후 그 포인터를 반환하기에
주로 `obj->field = 초기화값` 으로 하게 됨. 
snek_object_t *new_obj = malloc(sizeof(snek_object_t));


typedef union SnekObjectData {
  int v_int;
  float v_float;
  char *v_string;
  snek_vector_t v_vector3; -> 포인터가 아니라 구조체! 🔴
} snek_object_data_t;

🔴malloc이 아니라 구조체를 생성해서 직접 대입.
  snek_vector_t vector = {
    .x = x,
    .y = y,
    .z = z,
  };
  obj->data.v_vector3 = vector;


* 리턴할거면 스택 변수여도 되고,
다만 그 안에 참조되는 값이 있다면 malloc이어야 함.
  snek_array_t array = {
    .size = size,
    .elements = arr_ptr // 참조이나 malloc
  };

  obj->data.v_array = array; // 참조 아님
  return obj;

* strcat은 dest 문자열이 초기화된 상태를 가정.
-> strcat 전 strcpy 또는 memcpy
- malloc은 가비지 값들이 들어있어서 string으로 판단해버림.. 그래서 \0 null terminator 다음에 넣어야하는데 찾을 수 없게됨
- calloc은 0으로 모든 비트를 초기화 -> 0 비트가 \0 임. 

char *temp_str = calloc(new_length, sizeof(char));
strcat(temp_str, a->data.v_string);
strcat(temp_str, b->data.v_string);
```