+++
categories = ['Company']
series = ['Crul']
tags = ['node-libcurl']
title = "TypeOrm"
date = 2024-07-15T22:50:44+09:00
draft = true
+++
## Libcurl

어떤 과정을 거쳤고, 추가로 남은 것들은?
- 과정
  - curl 요청은 쿠키 없이도 잘 작동함을 확인, node-libcurl 설치
  - httpHeader, proxy 옵션으로 get요청
  - 리다이렉트 문제: 케이스 3가지에 따라 finalUrl 설정
    - 1. 스마트스토어내 리다이렉트, 2. 브랜드스토어로 리다이렉트, 3. 로그인화면으로 리다이렉트

- TODO
  - HTML 크롤링은 document (type all)이고,
이제 비공식 API로 해볼 것
  - 

curl -X GET http://127.0.0.1:8888

---
## 🟢목표
정확히 같은 요청인데 curl만 됨.
```shell
curl 'https://smartstore.naver.com/jungpumman/products/6986854546' \                           ─╯
  -H 'accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7' \
  -H 'accept-language: ko-KR,ko;q=0.9' \
  -H 'cache-control: max-age=0' \
  -H 'priority: u=0, i' \
  -H 'sec-ch-ua: "Not/A)Brand";v="8", "Chromium";v="126", "Google Chrome";v="126"' \
  -H 'sec-ch-ua-mobile: ?0' \
  -H 'sec-ch-ua-platform: "macOS"' \
  -H 'sec-fetch-dest: document' \
  -H 'sec-fetch-mode: navigate' \
  -H 'sec-fetch-site: same-origin' \
  -H 'sec-fetch-user: ?1' \
  -H 'upgrade-insecure-requests: 1' \
     -H "Connection: keep-alive" \
  -H 'user-agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/126.0.0.0 Safari/537.36' \
--compressed -H 'accept-encoding: gzip, deflate'

> node
fetch("http://localhost:8888/jungpumman/products/6986854546", {
  "headers": {
    "accept": "text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7",
    "accept-language": "ko-KR,ko;q=0.9",
    "cache-control": "max-age=0",
    "priority": "u=0, i",
    "sec-ch-ua": "\"Not/A)Brand\";v=\"8\", \"Chromium\";v=\"126\", \"Google Chrome\";v=\"126\"",
    "sec-ch-ua-mobile": "?0",
    "sec-ch-ua-platform": "\"macOS\"",
    "sec-fetch-dest": "document",
    "sec-fetch-mode": "navigate",
    "sec-fetch-site": "same-origin",
    "sec-fetch-user": "?1",
    "upgrade-insecure-requests": "1",
  'user-agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/126.0.0.0 Safari/537.36'
  },
  "referrerPolicy": "no-referrer-when-downgrade",
  "body": null,
  "method": "GET"
}).then(res => res.text()).then(console.log)
```