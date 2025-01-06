+++
title = 'Boot.dev'
date = 2024-12-28T00:20:22+09:00
draft = true
+++
# Refcounting GC 
C는 직접 관리하지만, 자동으로 해주는 메커니즘을 만들어본다.


* '실제 구조체 크기만큼' 메모리를 할당하고, 그 메모리를 반환하는거임.  -> sizeof 포인터가 아니라 구조체 자체임.
- calloc이 할당된 메모리를 0으로 초기화할 뿐, 반환된 메모리주소는 NULL이 아닌 특정 값임. 
`snek_object_t *obj = (snek_object_t*)calloc(1, sizeof(snek_object_t));`

* refcount decrease란 무엇인가?
객체의 경우 자신을 free 하기 전에 객체 안에 참조된 객체들의 count-- 해야한다.
왜냐면 그 참조된 객체들이 다른 곳에서 또 참조되고 있을 수 있기에.
refcount_dec 는 count=0이 되면 해제한다.

* 객체 배열 원소에 특정 값 넣기💎
element는 포인터이기 때문에 여기에 다른 주소를 넣어봐야 배열 인덱스 원소는 변하지 않음.
```c
a,b,c,d,e -> 모두 포인터. e Idx에 직접 value 넣어줘야
        |
       / \
   prev   newVal
refCount--  

snek_object_t *element = snek_obj->data.v_array.elements[index];
if (element != NULL) {
    refcount_dec(element);
}
// element = value;
snek_obj->data.v_array.elements[index] = value;
```

# Mark and Sweep GC
first, second 배열이 서로를 참조하는 경우 
refcount_dec 만으로는 free가 일어나지 않음.

-> MaS GC는 순환 참조로 인한 메모리 누수 방지
MaS는 주기적으로 한 번에 처리하므로 실시간 관리 부담이 적음.

### vm_track_object 함수를 사용하는 것이 "type-safe" 하다?

* vm->object
VM이 관리하는 모든 객체를 명시적으로 추적할 수 
가비지 컬렉터가 실행될 때, VM은 vm->objects에 등록된 객체만 검사

* vm->frames
가상 머신(VM)에서 실행 중인 함수 호출이나 코드 블록의 컨텍스트를 관리.
각 frame은 함수 호출이나 코드 블록의 실행 컨텍스트를 나타냅니다. 이 컨텍스트는 지역 변수, 임시 객체, 함수 인자 등을 포함.
frames 스택은 현재 실행 중인 모든 컨텍스트를 추적하여 함수 호출이 중첩될 때 올바른 컨텍스트를 유지
-> 그래서 frames 스택의 각 data[i]는 또다른 스택을 가리키는 프레임이다(frame->reference가 스택)

vm이 모든 객체를 트래킹한다-
_new_snek_object()는 obj 생성 후 vm의 objects에 그 obj를 넣는다. 
vm_track_object(): stack_push(vm->objects, obj);

도달 가능한 모든 객체들을 돌면서 mark, 
메모리를 스캔하면서 unmarked 객체들을 가비지로서 수집한다.
각 스택프레임에서 참조되는 객체들을 트래킹

---

root objects에 마킹하는 다른 방법들.
objects는 스택프레임에 의해 직접적으로 참조되는데

mark 함수는 'vm->frames' 스택의 모든 참조되는 obj에 mark 표시를 한다. 

즉 처음 new obj 생성시엔 is_marked = false,
스택프레임에 들어오고 나면 true로. 

메모리를 한번 sweep 할 때 mark를 하게된다.

BUT 문제점.
- [a] -> list만 마킹하고 malloc obj 하지 않은 a는 마킹안함.
- 생성했지만 리턴하지 않은 객체. -> 마킹은 됐지만 쓰이지 않으므로 없애야 할 존재임

# Tracing
Tracing이 이걸 해결함.
tracing은 mark and sweep 중 mark 단계.
root object에 의해 참조되는 모든 객체를 마킹하는.
- vm->objects의 모든 marked된 객체들을 gray_objects라는 스택에 담아 하나씩 꺼내면서 마킹한다. 

int/float/string은 원시타입이므로 객체를 참조하지 않음. -> tracing 안함.
- string은 free할 때 char * 메모리는 해제하지만 GC 대상은 아니다?


## 마킹시 그래프 탐색 알고리즘
Q. GC 구현에서 vm->objects의 모든 marked된 객체들을 gray_objects라는 스택에 담아 하나씩 꺼내면서 마킹하도록 하는데, 왜 이미 마킹된것을 스택에 담고, 거기서 마킹이 안된걸 마킹하는거지? 애초에 마킹된것만 담은거 아닌가

##### 마킹되지 않은 것들을 마킹하는 과정
-> is_marked = false인게 있나?
```c
void trace(vm_t *vm) {
  stack_t *gray_objects = stack_new(8);

  // 마킹된 모든 objects를 gray 스택에 담는다(회색 상태로)
  for (int i = 0; i < vm->objects->count; i++) {
    snek_object_t *obj = vm->objects->data[i];
    if (obj->is_marked) {
      stack_push(gray_objects, obj);
    }
  }
  // gray 스택의 모든 객체가 참조하는 객체들도 마킹한다(회색 상태로)
  while (gray_objects->count > 0) {
    snek_object_t* obj = (snek_object_t*)stack_pop(gray_objects);
    trace_blacken_object(gray_objects, obj);
  }
  // gray 스택 객체들은 검은색 상태가 된다. (neighboring all marked) 
  stack_free(gray_objects);
}
```

이 코드에서 `gray_objects` 스택을 사용하는 이유는 Mark-and-Sweep 가비지 컬렉션의 "삼색 마킹" 알고리즘과 관련이 있습니다. 이 알고리즘은 객체를 세 가지 색으로 구분하여 처리합니다: 흰색, 회색, 검은색. 각 색은 객체의 상태를 나타냅니다.

1. **흰색 (White)**: 아직 방문되지 않은 객체.
2. **회색 (Gray)**: 방문은 되었지만, 그 객체가 참조하는 다른 객체들은 아직 모두 방문되지 않은 상태.
3. **검은색 (Black)**: 객체와 그 객체가 참조하는 모든 객체가 방문된 상태.

이 알고리즘의 핵심은 회색 객체를 사용하여 객체 그래프를 탐색하는 것입니다. 회색 객체는 아직 완전히 처리되지 않았기 때문에, 이들을 추적하여 모든 연결된 객체를 방문할 수 있습니다.

### 과정 설명

1. **초기 마킹**: `mark` 함수는 루트 집합에서 시작하여 직접 참조되는 객체들을 마킹합니다. 이 단계에서는 객체가 회색 상태가 됩니다.

2. **회색 객체 처리**: `trace` 함수는 회색 객체를 `gray_objects` 스택에 넣고, 이 스택을 통해 객체 그래프를 탐색합니다. `trace_blacken_object` 함수는 회색 객체를 처리하여 그 객체가 참조하는 다른 객체들을 마킹하고, 이들을 다시 회색으로 만들어 `gray_objects` 스택에 추가합니다.

3. **검은색으로 변환**: `trace_blacken_object`가 완료되면, 해당 객체는 검은색이 됩니다. 즉, 그 객체와 그 객체가 참조하는 모든 객체가 방문된 상태가 됩니다.

### 왜 이미 마킹된 객체를 스택에 넣는가?

- **객체 그래프 탐색**: 이미 마킹된 객체는 회색 상태로, 그 객체가 참조하는 다른 객체들을 탐색해야 합니다. 이 과정에서 새로운 객체가 발견되면, 그 객체를 회색으로 만들어 스택에 추가합니다.
- **재귀적 탐색 방지**: 스택을 사용하여 명시적으로 객체를 추적함으로써, 재귀적 호출 없이 객체 그래프를 탐색할 수 있습니다. 이는 스택 오버플로우를 방지하고, 탐색 과정을 명확하게 관리할 수 있게 합니다.

따라서, `gray_objects` 스택은 객체 그래프를 효율적으로 탐색하고, 모든 도달 가능한 객체를 올바르게 마킹하는 데 중요한 역할을 합니다.

---
# Sweeping

이제 모든 객체가 is_marked를 갖게 되었다. 
reachable 한지 판단할 수 있다.

단 마킹된 모든 객체는 is_marked = false로 리셋되어야
다음번 mark phase가 돌 때 - 다시 마킹되지 않으면 참조되지 않는 것으로 free할 수 있다. 

#### 전체 과정**
```c
void vm_collect_garbage(vm_t *vm) {
  // 현재 상태: 스택프레임에서 참조하는 모든 객체 마킹 -> gray
  mark(vm);
  // 그래프: 그 객체들이 참조하는 객체들도 마킹 -> black
  trace(vm);
  // 모든 objects 돌면서 마킹된 것은 초기화, 안된것은 제거
  sweep(vm);
}
```

---

`&obj->data.v_array` vs `obj->data.v_array`
1.obj->data.v_array: 구조체 값 자체
2.&(obj->data.v_array): 구조체의 주소

v_array가 *가 아닌 구조체로 저장되어있기 때문에,
포인터로 접근하려면 2번으로 해야.
```c
typedef union SnekObjectData {
  snek_array_t v_array;
} snek_object_data_t;

snek_array_t *array = &obj->data.v_array;
```
