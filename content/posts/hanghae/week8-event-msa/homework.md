+++
title = 'week8 과제'
date = 2025-07-07T00:20:22+09:00
draft = true
+++

🔵prisma null pointer 
:아키텍처 충돌로 발생한 문제

1. Node.js (x64, Rosetta) → Prisma CLI 명령 실행
2. Prisma CLI → Prisma 엔진 (arm64 네이티브) 호출
3. 데이터 전달 시 메모리 주소 포맷 불일치
4. Rust 엔진이 "인식할 수 없는 주소"라고 판단
5. assert 실패 → 프로세스 종료

메모리 주소 랜덤화
macOS의 ASLR(Address Space Layout Randomization)
매번 다른 메모리 주소 할당 → 간헐적 충돌

바이너리에서만 이 문제가 발생한 이유는 저수준 메모리 접근과 프로세스 간 통신에서 아키텍처 차이가 가장 극명하게 드러나기 때문

🔵EventListner - 테스트환경에선 AppModule, 
app = module.createNestApplication();
await app.init();
-> onApplicationBootstrap을 해줘야 모든 리스너가 등록이 됨

