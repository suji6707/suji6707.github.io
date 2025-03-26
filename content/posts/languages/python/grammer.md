+++
title = 'Python Grammer'
date = 2024-06-30T00:20:22+09:00
draft = true
+++

## 🎆 파이썬 문법

**는 JS의 spread와 비슷
여러 파라미터가 들어갈때, 그 순서를 정확히 지키지 않고도
객체로 만들어두고 ** 로 보내면 알아서 그에 맞는 인자로 할당되나봄.

query = { 'TableName': 'test', 'KeyCond': 'pk = :pk AND sk = :sk' }
response = dynamodb.query(**query)


---

💎 if __name__ == "__main__": 구문을 사용하면 해당 파일이 직접 실행될 때만 main() 함수가 실행됨.!

