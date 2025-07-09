+++
title = 'week7 시행착오, 배운점'
date = 2025-07-07T00:20:22+09:00
draft = true
+++

WEEK7 과제

대기열 sorted set

ZADD scheduleId:1 timestamp queue-token -> 먼저 온 순서대로 정렬됨. 
대기인원 zcard
순번 zrank
활성 예약 가능자 추출 : 스코어가 가장 낮은 N명의 사용자 추출.
-> 이렇게 추출된 사용자들에게는 예약 페이지로 진입할 수 있는 링크나 알림을 보냄.

예약 타임아웃 관리 (비활성 유저 제거)


* 예매율
인터파크 생각. 
🟣방법
1. 레디스에 예매율 실시간 기록해놓고 db로 스냅샷 뜨거나
2. db에 집계해다가 업데이트해서 레디스로 분석한걸 주기적으로 넣어줘도 됨.
-> 1이 더 맞음.
실시간성: 예약이 발생하는 즉시 랭킹 반영하는게. 
데이터 손실 위험이 있어보이지만, db에 예약데이터는 다 있어서. 
하이브리드: 차이가 5% 이상이면 DB 데이터로 동기화
** 그래도 nestjs cron 알아봐
1-3. 주기적 스냅샷 배치🔺
CREATE TABLE concert_sales_snapshots

예매 발생(또는 취소) 시점마다
Redis의 Sorted Set에 예매율(score) 갱신


엥? 
그래서 대기열토큰은 sorted set이 아니라 그냥 LIST로 한다고?
중복방지용 SET이면 순서보장 안되잖아. 그럼 Sorted set 아닌가 ㅋㅋ
