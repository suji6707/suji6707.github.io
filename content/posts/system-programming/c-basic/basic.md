+++
title = 'C 베이직'
date = 2024-12-08T00:20:22+09:00
draft = true
+++
## Basics



---
size_t 타입: sizeof의 반환 타입.
%zu는 printf에서 size_t 타입의 값을 출력할 때 사용하는 형식 지정자

32비트 시스템에서 int의 크기는 일반적으로 4바이트인 반면 64비트 시스템에서는 일반적으로 8바이트

---
C에서 두 개의 정수를 나누면 결과는 항상 정수로 나온다.
float는 3.0으로 나눠줘야. 

void !== None

### string은 char[] 타입.
string은 char의 배열이고
char *var = "string";
으로 char 배열 스트링 첫주소를 var로 선언.
printf("%s\n", var) 로 하면 알아서 배열을 출력함.

---
C의 헤더 파일과 소스 파일의 관계

1. 파일 구조:
```
exercise.h  (헤더 파일) - 함수 선언(프로토타입)
exercise.c  (소스 파일) - 함수 구현
main.c      (메인 파일) - 메인 함수
```

2. 컴파일 과정:
- 컴파일러는 모든 .c 파일들을 함께 컴파일합니다.
- 링커가 컴파일된 객체 파일들을 하나의 실행 파일로 연결합니다.

3. 예시:
```c
// exercise.h
int add(int a, int b);  // 함수 선언만 있음

// exercise.c
#include "exercise.h"
int add(int a, int b) { // 실제 구현
    return a + b;
}

// main.c
#include "exercise.h"
int main() {
    int result = add(2, 3); // exercise.c의 구현을 사용
}
```

4. 작동 방식:
- main.c는 exercise.h를 include하여 함수 선언을 알게 됨
- 컴파일 시 exercise.c의 구현과 main.c가 함께 링크됨
- 실행 파일에는 모든 코드가 포함됨

즉, .h 파일을 include하는 것만으로도 충분한 이유는:
1. .h 파일이 함수의 인터페이스(선언)를 제공
2. 컴파일러가 모든 .c 파일을 함께 처리
3. 링커가 실제 구현과 호출을 연결 -> 실제 구현 위치 get ⚪️⚪️⚪️  

이것이 C의 분할 컴파일(separate compilation) 시스템입니다.

---
