+++
title = 'Langraph Persistence'
date = 2025-03-26T22:50:44+09:00
draft = true
+++

# CUDA (hugo)

same program is executed on many data elements in parallel
ex) GPU는 이미지의 수백만 픽셀 각각에 대해 "밝기 증가"라는 동일한 연산을 병렬로 수행.

 
GPU는 수많은 코어를 가지고 있기 때문에, 특정 코어가 메모리에서 데이터를 기다리는 동안 (latency 발생), 다른 코어들은 계속해서 계산 작업을 수행할 수.

스레드는 GPU에서 독립적으로 명령어를 실행할 수 있는 가장 작은 단위.
요즘 수만개 코어.

Host: CPU와 DRAM을 포함하는 CPU 시스템. GPU 연산에 필요한 데이터 준비, GPU에서 실행될 커널함수 호출(런칭) + 그리드/블록 구조 설정.
Device: GPU와 GPU 메모리를 포함하는 GPU 시스템. 커널 런칭 시 설정된 그리드, 블록, 스레드 구조를 기반으로 병렬 스레드 실행을 관리

__global__ kernal 함수 -> device에서 실행할 로직.
call by reference 필수.

threadIdx x,y,z -> 위치.


Thread는 각자 로컬 메모리 및 레지스터를 가짐.
Blocks는 각 Shared memory를 가짐. 그 블록내 스레드들이 공유.

'local' : scope and lifetime of this memory space

SM (Streaming Multiprocessor), 스트리밍 멀티프로세서는 실제 연산 수행하는 엔진.


malloc -> global memory로 감. stored in off chip 
매우 크지만 넘 느려서 트래픽 최소화.



