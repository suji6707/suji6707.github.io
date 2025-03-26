
+++
title = 'Lecture'
date = 2025-02-16T00:20:22+09:00
draft = true
+++
# 중간고사 서술형

---
🟢 서술식 공부법
- 문제를 읽고 5분 생각. 나름의 가설을 세운다.
- 이때 최대한 검색을 한다.
- 답을 보고 정리한다.

------------------------------------------------------------------------------
P3.

3c.
void acquire(int *mylock) {
	while (!CAS(mylock, 0, 1)); // mylock != 0 인 동안 기다림
}  

3d.
코어가 남아돌면 스핀락 시키는게 나을 수.
CAS는 실제로 락을 획득할 수 있을 때만 쓰기 작업을 수행해서 캐시 동작이 더 좋음.
이런 상황에선 스핀락 while(!CAS){}가 효율적일 수.

3e. 주소 값 값!
void push(node_t **rootp, node_t *newnode) {
	do {
		node_t *oldfront = *rootp;
		>>> load, store 전후로 끼어들면 연산 실패, 다시 함.
		newnode->next = oldfront;
	} while (!CAS(rootp, oldfront, newnode)); // rootp 첫 노드 != oldfront 인 동안 반복
}

CAS가 rootp를 newnode로 갱신하는거고

3f.
TSET, CAS는 유저레벨 원자적 인스트럭션이라 커널단에서 하는 blocking(sleep)을 시킬 수 없다. 
-> Futex는 가능. 

TSET(&address) {
	r1 = M[address]
	M[address] = 1
	return r1 // 이전 값 반환
}

void acquire(int *thelock) {
	while (TSET(thelock)) { // 이전 값이 1(busy)인 동안

		futex(thelock, FUTEX_WAIT, 1); // 깨어나도 while TSET 체크
	}
}

3g.
언제 futex lock이 최적이 아닌가?
- release 할 때마다 커널을 통해서 wake up 시켜야함. waiting이 없을지라도.
=> uncontested인데(스레드끼리 경합할 일이 없는) 공통 변수 접근이 굉장히 잦은 경우엔 커널-유저 스위칭 오버헤드 높을듯.

3h. 
typedef enum { UNLOCKED, LOCKED, CONTESTED } Lock;
Lock mylock = UNLOCKED;

각 상태의 의미: unlocked- 아무도 공유변수에 접근하지 않은 상태, locked- 하나의 스레드가 사용 중, contested-두 개 이상 스레드가 동시 접근.(하나 외엔 기다림)

contested 상태가 어떤 상황에서 최적화를 해주는지.
- tset 대신 swap 또는 cas.
lock의 이전 상태값을 읽어 CONTESTED 일 때만 wake up. 
- 두 스레드만의 동시접근이 빈번하지 않은 경우 wakeup 오버헤드가 적을 수.

if lock = LOCKED | CONTESTED
	while swap(lock, CONTESTED) 이전 값 != UNLOCKED:
		-> 깨어났을 때 wating이 0이면 swap(lock, LOCKED), 1 이상이면 swap(lock, CONTESTED)임.

3i.
code

------------------------------------------------------------------------------
P4. short answer
2문장 이내로 서술.

4a. 인터럽트 발생시
- 인터럽트는 external source로 부터 발생. not from within processor's normal instruction stream.
원래 실행하던 흐름이 아니라 외부 소스(하드웨어 인터럽트, software exception, timer scheduling)에서 인터럽트가 발생. 
- 인터럽트 컨트롤러: CPU가 주변 장치(peripheral devices)로부터의 인터럽트 요청을 관리하고 처리하는 데 사용되는 하드웨어 구성 요소. 
	- 여러 장치로부터 인터럽트 요청을 받고
	- 동시 발생시 우선순위 결정
	- 가장 높은 우선순위 인터럽트를 CPU에 전달
- 인터럽트 마스킹: OS가 인터럽트를 개별적으로 enable/disable 할 수 있음

- 과정
1. 프로세서는 하던 일을 멈추고 유저 registers를 new stack에 저장한 후 커널 스택으로 스위칭.
2. 인터럽트ID에 따라 핸들러를 실행

4b. 
실제 값은 어디에 저장?
- list_elem 구조체를 멤버로 갖고있는 elem 구조체. 
C는 object-oriented 언어는 아니지만 객체지향 메커니즘을 제공하기 위해,
제너릭 타입으로 함수들을 사용하기 위해 필요.
노드가 어떤 타입인지 전혀 상관없이 linked list가 작동할 수 있게. 

4c. (강의 Lecture 7 14분)
1. 같은 프로세스 안에서 두 스레드간 스위칭할 때 저장해야할 것.
- 프로세스A -> A의 커널스택으로 현재 진행중이던 eip, esp를 저장해둠 
- 커널스택의 eip, esp가 CPU에 올라와 작업을 함? 
- 스케줄링에 의해 프로세스B의 커널 스택에 저장되어있던 eip, esp를 CPU에 올려서 user code, user stack 실행. 

2. 다른 프로세스 안에서 두 스레드간 스위칭할 때 저장해야할 것.
** memory protection state가 스위칭 되어야 함. active page table 등.
프로세스에 특정되는 PCB, file descriptor table 등은 주소공간/레지스터를 바꾸면서 자동으로 스위칭 되어야. 
-> 프로세스마다 독립된 주소공간이 있고 PCB, fdt 등 저옵는 커널 공간제 존재.
물리 메모리의 특정 위치에 존재하기 때문에, 프로세스 전환 시 이러한 정보를 별도로 저장하고 복원할 필요가 없습니다. 단순히 활성화되는 주소 공간이 변경됨에 따라, CPU가 접근할 수 있는 PCB와 파일 디스크립터 테이블이 자동으로 변경되는 것.

4d. race condition에 의한 방사선 의료사고
소프트웨어는 기계부품 설정이 완료될 때까지 방사선 조사를 시작하지 않도록 동기화 되었어야.
사고는
설정이 완료되지 않은 상태에서 방사선이 조사되었고
아마 기계부품단에선 아직 방사선이 조사되기 전으로 인식하여 또다시 방사선을 조사하게 했을듯.

4e. 
하이퍼스레딩: 한 코어에서 동시에 두 스레드를 돌리는 것. concurrent 이면서 parellel.
-  하나의 CPU 코어가 마치 두 개의 코어처럼 동작
각 스레드는 CPU 내부의 자원(functional unit, 캐시 등)을 공유하지만, 각자의 레지스터를 가지고 있어 서로의 작업에 간섭받지 않고 독립적으로 실행
- 비순차적 파이프라인(Out-of-Order Execution Pipeline)은 CPU가 명령어를 프로그램에 작성된 순서대로 실행하는 것이 아니라, CPU 내부적으로 명령어의 실행 순서를 재배열하여 성능을 향상시키는 기술

4f.
fork()를 하면 부모와 별도의 메모리 공간에서 동일한 내용을 가진 자식 프로세스가 생성된다.
fork는 OS가 parent의 모든 메모리를 복사하도록 하는가? 
- no. copy on write 로 효율화. 프로세스가 변경하려 한 데이터 일부만 복사됨. 실제 변경되는 데이터만 복사하는거라, 변경 전까지는 같은 메모리 공간을 공유하는 셈. 
- int i가 있고 fork 후 동시에 변경하면 서로다른 i 값을 가짐!

답)
fork 시점에 부모 page table entries를 read-only로 변경. child를 위한 page table을 복사하기 전까지.
child나 부모 둘 중 하나라도 변경하려하면, page fault를 내고 해당 페이지의 two copies를 만든다. 
마치 부모 공간 전체를 copy한 것 처럼 착각하게 만들면서. 


4g. 
핀토스 userprog/syscall.c
유저가 호출한 syscall이 뭔지 어떻게 알 수 있나? syscall handler function은 하나뿐인데.

```c
static void syscall_handler(struct intr_frame* f UNUSED) {
  uint32_t* args = ((uint32_t*)f->esp);
  /* printf("System call number: %d\n", args[0]); */

  if (args[0] == SYS_EXIT) {
    f->eax = args[1];
    printf("%s: exit(%d)\n", thread_current()->pcb->process_name, args[1]);
    process_exit();
  }
}

int wait(pid_t pid) { return syscall1(SYS_WAIT, pid); }
```
- intr_frame: 인터럽트 발생시 CPU 상태를 저장
- esp 스택포인터를 args 포인터에 저장해둠.
•args[0] : 시스템 콜 번호 (SYS_EXIT)
•args[1] : exit() 함수의 인자 (종료 코드)
-> 즉, intr_frame 스택포인터의 첫번째 요소가 시스템콜 번호.

hw-memory/pintos/src/threads/interrupt.h

인터럽트가 발생하면 CPU는 커널 모드로 전환하고, intr_frame 구조체를 커널 스택에 푸시합니다.  
intr_frame 자체는 커널 스택에 존재하지만, 그 안에 저장된 esp 값은 유저 스택을 가리키고 있습니다.
-> intr_frame 에 vec_no, error_code 다 있음.


4h. code

4i. 
high-level File I/O 인터페이스는 왜 streaming I/O 인터페이스인가?
- 스트리밍 통신이란 byte 단위로 읽고 쓰는 것. 형식 관계없이.
- fread는 컴퓨터 buffer size만큼 한번에 읽어 유저 공간에 가져지만 
지정한 바이트수만큼 읽어온 것처럼 행동한다. 

답)
스트리밍 통신이란: 연속적인 연산을 통해 큰 연속적인 스트림 바이트를 파일로부터 읽거나 쓰는.
low level과의 핵심 차이는, user-level buffer의 존재여부.
high-level I/O 연산은 여러번의 fread, fwrite 요청을 단 몇 번의 read, write 연산(to kernel)로 합칠 수 있다. 

** 왜 High-level I/O는 User Buffer를 두는가? 에 대한 예시
ex) fwrite를 하면 디스크가 아닌 FILE's buffer에 쓴다. (FILE 구조체에는 fd, Buffer array, Lock 등이 있다.)
- FILE's buffer가 가득 차면 flushed 되어 fd로 쓰여진다. 
- function call보다 syscall이 훨씬 비싸므로.
ex) getline: 실제로는 커널에서 한줄씩 읽는 게 아니라 한번의 big read syscall로 user space로 가져온 후 first new line을 가져옴. 커널은 format이란 개념이 없기에..

단, 유저레벨 버퍼를 잘 활용할 수 있을 때만 유용.
random access (여기저기 흩어진 위치에 접근) 패턴이라면, 버퍼가 효율적으로 사용되지 못합니다.  버퍼에 데이터를 채우기 전에 다른 위치로 이동하게 되므로, 작은 요청들이 묶이지 않고 각각 커널에 전달되어 오히려 성능 저하


------------------------------------------------------------------------------
P5. 

5a. personal에 code로 작성함.

