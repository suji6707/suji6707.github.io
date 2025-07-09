+++
title = 'week5 과제'
date = 2025-07-07T00:20:22+09:00
draft = true
+++

WEEK5 🔴 워커 테스트 설정 배운점.
- js는 import시 그 파일이 한 번 실행된다. bootstrap이 실행되고,
그 안의 함수를 테스트에서 별도로 또 실행한 셈.
- redis client undefined 에러는 꺼질때 발생한 문제였음.
WorkerOption - autoRun은 기본값 true라서 worker.run이 불리고 돌아가는데,
테스트가 종료되면서 redis client도 같이 종료됐으니까.
그래서 테스트에서 autoRun: false로 해두고, 워커가 실행할 로직을 process() 밖에 함수로 빼놓고 그걸 수동 실행하면 됨.
>> 사실 종료를 중복 호출하는건 큰 문제가 아님.
문제는 종료된 커넥션에 대해 계속 retry하는 프로세스가 있다는거임.