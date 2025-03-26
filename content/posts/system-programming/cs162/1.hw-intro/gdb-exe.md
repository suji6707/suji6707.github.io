+++
title = 'HW1'
date = 2025-01-05T00:20:22+09:00
draft = true
+++
# GDB Basic

i386은 첫 32bit x86 CPU로, 핀토스에서 쓰는 아키텍처다.
qemu-i386을 통해 ARM 머신에서도 x86 유저 스페이스를 에뮬레이션할 수 있도록 스크립트를 설치해두었다. 
어떠한 아키텍처에서도 가상 CPU를 'imitating' 흉내냄으로써 x86 exe를 실행할 수 있도록 한다.


(참고)
KVM은 "가상화" 이고 Qemu는 "에뮬레이션". 두 가지를 모두 하기 위해 같이 씀.
QEMU와 KVM은 수많은 Hypervisor 중 하나다. Hypervisor는 서로 다른 복수 개의 OS를 단일 물리 머신 위에서 스케줄링 할 수 있는 소프트웨어.
Qemu(Quick Emulator)는 가상머신이 명령어를 번역. 다른 하드웨어(ARM vs x86)를 binary transition으로 가상환경으로 하여금 그러한 물리적인 환경이 존재하는 것처럼 해준다. 물리 장치 동작을 소프트웨어로 동작시키므로 느릴 수밖에 없다.
반면 KVM(kernel-based virtual machine)은 하드웨어에 Ubuntu를 설치해 다른 OS를 실행하는 방식으로, 가상머신에서 명령어를 번역할 필요가 없어 훨씬 빠르다. 
-> KVM은 리눅스 커널 자체가 가상화 기능을 제공.


s step 함수 내부로
n next 함수 호출 건너뛰고
c continue 다음 break 까지 계속 실행

break funcName
continue -> 이미 들어온듯
l funcName 함수 기준 10줄


4. 0xffffdb3df748 (char**) : argv[]는 char* 들의 배열, 그 처음을 가리키면 char**
5. 0xffffdb3dfe95 (char*) "/home/workspace/code/personal/hw-intro/map"
argv는 실행파일부터 0 1 2
6. break recur, continue
7. print recur : {int (int)} 0xaaaaca2e0860 <recur>
8. info line / info line funcName 
disas 0xaaaaca2e0888, 0xaaaaca2e088c

함수 호출: ARM에선 call 대신 bl(Branch with link)
레지스터:
eax(Accumulator Register) -> x0/x0: 함수 반환값 저장. 
esp(Stack Pointer) -> sp: 스택 최상단 주소. 함수 호출시 매개변수, 로컬변수 관리. push/pop으로 스택 조작시 자동으로 조정됨
🟡eip(Instruction Pointer) -> pc(Program Counter): 다음에 실행할 명령어 주소. 프로그램 실행 흐름 제어.

mov eax, 42      # eax에 42 저장 (반환값 설정)
push eax         # eax 값을 스택에 저장 (esp가 자동으로 조정)
call func        # 함수 호출 (eip가 func의 주소로 변경)

12. 🔴si: 어셈블리 명령어 단위로 실행해서 call 까지 갈 수 있음$
n은 함수호출 건너뛰어버림 src 기준이라. s도 마찬가지.
x0             0x27                39
sp             0xffffdb3df510      0xffffdb3df510
pc             0xaaaaca2e086c      0xaaaaca2e086c <recur+12>

16. break recur if i == 0  조건부 
(gdb) backtrace
#0  recur (i=0) at recurse.c:6
#1  0x0000aaaaca2e08a0 in recur (i=1) at recurse.c:7
#2  0x0000aaaaca2e08a0 in recur (i=2) at recurse.c:7
#3  0x0000aaaaca2e08a0 in recur (i=3) at recurse.c:7
#4  0x0000aaaaca2e0854 in main (argc=1, argv=0xffffdb3df748) at map.c:25



---
•disas : 현재 함수 전체를 디스어셈블
•disas function : 특정 함수를 디스어셈블
•disas address1, address2 : 주어진 주소 범위를 디스어셈블

(참고)
각 어셈블리 명령어의 의미
- `stp`: Store Pair (두 레지스터 값을 메모리에 저장)
- `mov`: Move (값 복사)
- `str`: Store Register (레지스터 값을 메모리에 저장)
- `ldr`: Load Register (메모리에서 레지스터로 값 로드)
- `add`: Addition (덧셈)
- `bl`: Branch with Link (함수 호출)
- `cmp`: Compare (비교)
- `b.le`: Branch if Less or Equal (작거나 같으면 분기)
- `sub`: Subtract (뺄셈)
- `ret`: Return (함수 리턴)

---
Call stack
스택프레임이 있는 스택. (Stack of stack)
함수가 호출되면 스택프레임을 생성(컴퓨터로 하여금 함수 실행이 완료되면 어떻게 return control 할지 알려주는)
또한 스택프레임은 로컬변수와 함수 args가 'stored'되는 곳.
backtrace: 현재 스택프레임 below의 스택프레임 리스트를 가져오는것.(가장 최근에 호출된 함수부터 맨처음 호출된 것 순으로)

backtrace.
0,1,2... number
frame num 으로 다른 스택프레임 조회. 실제 실행위치는 변경하지 않음.
info frame

main 함수의 리턴 주소?
info frame
- saved pc = 0xffff8e4b73fc
pc: 현재 실행중인 명령어 주소
saved pc: 함수가 호출되기 전 저장된 리턴 주소. main 함수가 종료된 후 실행이 재개될 주소


layout asm
layout src

22. return 0 -> 🔴한 줄이 아님. 
mov 0 $eax  반환값에 0을 담고
기존의 context를 pop 해서 가져온다음
ret .




---
# 소스코드에서 실행파일로.
보통의 컴파일 과정: 소스코드 → 전처리 → 컴파일(어셈블리) → 어셈블 → 링크
-S 는 어셈블리까지만 생성하도록 함. 
.S는 어셈블리 확장자.

1. 
movl    8(%ebp), %eax      # eax에 매개변수 i 값을 로드
subl    $1, %eax           # eax에서 1을 뺌 (i-1)
subl    $12, %esp          # 스택 공간 확보
pushl   %eax               # i-1 값을 스택에 push (함수 인자)
call    recur              # recur 함수 호출
addl    $16, %esp          # 스택 정리

ebp(Extended Base Pointer)는 스택 프레임의 기준점 역할
함수의 스택 프레임 시작점 표시. 
지역 변수와 매개변수에 접근할 때의 기준점
🔴x로 끝나면 보통 변수/인자. eax는 반환이고. ~p는 고정된 느낌.$

이제 컴파일된 코드를 실행파일로 만들기 위해 오브젝트 파일을 만들어보자. 
-c 옵션: C 코드 → 어셈블리 → 오브젝트 파일(.o)까지. [컴파일], [어셈블] 한번에 실행함. 

[C 코드] → [전처리] → [컴파일] → [어셈블] → [링크] → [실행 파일]
   .c         .i         .s         .o        →     (executable)

어셈블러는 raw assemble 코드를 오브젝트 파일로 변환한다. 오브젝트 파일은 코드뿐만 아니라 실행에 필요한 메타데이터들 포함. 
이 OS에서는 ELF(executable linkable format)을 사용해서 .o를 만든다.
objdump 프로그램으로 .o 파일을 읽는다. 

3. ELF 심볼 테이블 보려면
objdump -t 

4. 
심볼 주소  플래그비트 섹션이름  정렬값/크기  심볼 이름
00000000 g O .data 00000004 stuff   # 전역 데이터 오브젝트 'stuff'
00000000 g F .text 00000060 main    # 전역 함수 'main'
00000000 *UND* 00000000 malloc      # 정의되지 않은 외부 함수 'malloc'
00000000 *UND* 00000000 recur       # 정의되지 않은 외부 함수 'recur'

5. link
a.c b.c -o A 처럼 c파일로 바로 링킹할 수도 있지만
컴파일 시간을 줄이기 위해 a.o b.o -o A 처럼 하기도. (변경된 파일만 리컴파일하기 위해)

링크된 전체 map의 심볼 테이블을 보자.
i386-objdump -x -d map
SYMBOL TABLE:
여러 세그먼트로 구성된걸 볼 수 있고, 함수와 변수명은 주소가 있는 라벨과 대응된다. 

6. .text 세그먼트에 recur 함수
7. stuff: data, foo: bss (O)
8. 스택과 힙은 런타임에 동적으로 생성되므로 심볼 테이블에 없음.
컴파일 타임에 결정되는 static 영역(.text 코드, .data 초기화된 전역변수, .bss 초기화되지 않은
9. 스택은 낮은쪽으로 자란다. 
char h[] = "hello"; 배열 - 스택에 저장. 한 요소씩 변경 가능.
char *h = "hello"; 포인터 - 불변 문자열 리터럴 - .text

10. preprocess c code
🟡전처리기: #include "printer.c"가 처리되면, printer.c의 내용이 그 위치에 그대로 복사됨. 
ifdef X : X가 정의되지 않았으면 else로 감.
11. CFLAGS: 컴파일러 옵션을 지정 -> 여기에 -D 매크로정의

(참고)
objdump -t 심볼

1. 플래그 비트가 필요한 이유:
- g (global): 다른 파일에서도 접근 가능한 심볼
- l (local): 현재 파일에서만 사용 가능
- w (weak): 약한 바인딩, 다른 정의가 있으면 override 가능
- F (function): 함수라는 것을 표시
- O (object): 데이터 객체라는 것을 표시

2. 주요 섹션들:
- `.text`: 실행 가능한 코드 영역
- `.data`: 초기화된 전역/정적 변수
- `.bss`: 초기화되지 않은 전역/정적 변수
- `.rodata`: 읽기 전용 데이터 (상수, 문자열 리터럴)
- `.stack`: 스택 영역 (지역 변수, 함수 호출 정보)
- `heap`: 동적 할당 영역 (malloc 등으로 할당)

3. 섹션이 필요한 이유:
```c
const char* str = "Hello";  // .rodata 섹션
int global = 42;           // .data 섹션
void func() { }           // .text 섹션
```
- 메모리 보호 (실행/읽기/쓰기 권한 구분)
- 메모리 효율성 (같은 특성의 데이터를 모아서 관리)
- 링커가 각 섹션을 적절히 배치하기 위해

따라서 이러한 정보들은 프로그램의 메모리 구조와 보안, 효율성을 위해 필요합니다.