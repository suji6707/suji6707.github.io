+++
title = '함수형 포인터'
date = 2024-12-28T00:20:22+09:00
draft = true
+++
# Function Pointer


* 함수 포인터.
함수를 직접 전달하는 것과 함수 포인터를 전달하는 것은 같은 의미.
code 영역에 있기에 메모리 주소를 전달하게 되는.

함수 포인터를 구조체의 멤버나 변수로 저장할 때는 반드시 포인터 형태로 선언
struct Callbacks {
    void (*onClick)(int, int);    // 반드시 포인터로 선언해야 함
    // void onClick(int, int);    // 이렇게는 불가능
};

void *aux로 온 함수포인터를 형변환 해줄 수 있음.
bool (*less_func)(const word_count_t *, const word_count_t *) = aux;
-> aux(wc1,wc2)보다는 less_func(wc1, wc2);


