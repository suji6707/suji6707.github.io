+++
categories = ['Web']
tags = ['Crawling']
title = 'Puppeteer 사용법'
date = 2024-04-06T22:50:44+09:00
draft = true
+++
## Puppeteer



### 퍼페티어로 유튜브 자막 크롤링하기

### 1. Puppeteer란?
헤드리스:
헤드풀:

### 스크롤 다운
1. selector에 해당하는 첫번째 요소를 찾아 section 변수에 할당.
2. 스크롤 동작을 비동기적으로 처리하기 위해 new Promise를 생성
3. 스크롤 실행: 100ms마다 100px만큼 스크롤

생각해봐.
autoScroll 함수는 스크롤만 할 뿐.
여기에 크롤링 로직을 넣어줘야 한다.
result = []

### The chromium binary is not available for arm64
puppeteer install arm


throw new Error(`Could not find Chrome (ver. ${this.puppeteer.browserRevision}

~./cache/puppeteer 에 chrome이 자동으로 설치되는데,
문제는 이미 arm64인데 x64 노드로 설치되어 로세타가 바이너리를 트랜스코딩하려 한다는 것.
이는 심한 성능문제를 일으킬 수 있으므로
자동설치 없이 brew로 설치한 chronium으로 실행하도록 하려는 것.

.env 또는 
To ~/.zshrc add:

export PUPPETEER_SKIP_CHROMIUM_DOWNLOAD=true
// -> It will skip trying to download chromium
export PUPPETEER_EXECUTABLE_PATH=`which chromium`
// -> 이 path내 바이너리가 실제 작동해야 할 크로니움


const browser = await puppeteer.launch({
    executablePath: process.env[‘PUPPETEER_EXECUTABLE_PATH’],
})


1. 스크롤도 필요없음. 버튼만 누른다음 자막 뜨면 쿼리셀렉터로.
``` javascript
await page.waitForSelector(#panels)

async function autoScroll(selector){
    await page.evaluate(async (selector) => {
      const section = document.querySelector(selector);
      
	  await new Promise((resolve, reject) => {
        let totalHeight = 0;
        const distance = 100; // 스크롤당 이동 px

        const timer = setInterval(() => {
          section.scrollBy(0, distance);
          totalHeight += distance;
          if(totalHeight >= section.scrollHeight){
            clearInterval(timer);
            resolve();
          }
        }, 100);
      });
    }, selector);
  }

await autoScroll()

const subtitles = await page.evaluate(() => {
	const segments = Array.from(document.querySelectorAll('.자막부분'))
	return segments.map(seg => seg.innerText)
})
```

2. 브라우저를 달고있으므로? 쿠키가 있는 상태 - fetch 요청을 날린다?
```javascript
    // POST 요청을 위해 page.evaluate() 사용하여 페이지 내에서 JavaScript 실행
    const transcriptData = await page.evaluate(async (videoId) => {
        const response = await fetch('https://www.youtube.com/youtubei/v1/get_transcript?prettyPrint=false', {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json',
                // 필요한 경우 추가 헤더 입력
            },
            body: JSON.stringify({
                // 요청 바디에 필요한 데이터 입력
                videoId: videoId,
                // YouTube API에 필요한 다른 매개변수 추가
            }),
        });
        return response.json(); // 응답을 JSON 형태로 변환
    }, videoId);
```




팁
pgrep chrome | wc -l : 실행중인 chrome 프로세스의 총 개수
- "process grep"의 약자로, 시스템에서 실행 중인 프로세스 중에서 특정 조건에 맞는 프로세스의 PID(Process ID)를 찾아 출력
- wc: word count. 
- -l: 줄의 수를 센다. pgrep으로부터 전달받은 각 PID를 한 줄로 카운트.


1. 페이지 상태 확인.
page.evaluate() 함수 안에서 console.log()를 호출하면, 그 로그는 Puppeteer를 실행하는 Node.js 환경의 콘솔이 아닌, 브라우저의 콘솔에 출력됨 ㅋㅋ

### 파싱
- 시간과 자막을 구분하거나,
- 시간의 형태를 띈 ':'를 regex로 replaceAll 하거나. 

클래스 형태
보다시피 segment, segment-timestamp, segment-text로 나뉘는 걸 알 수 있다. 
- 전체: segment style-scope ytd-transcript-segment-renderer
- 시간: segment-timestamp style-scope ytd-transcript-segment-renderer
- 자막: segment-text style-scope ytd-transcript-segment-renderer

"2:59\n걱정할 필요가 없어요",
  "2:59", "2:59", "", "걱정할 필요가 없어요", "",  

"3:01\n청주가 있으면 청주도 넣어주시면 좋아요",
  "3:01", "3:01", "", "청주가 있으면 청주도 넣어주시면 좋아요", "",  

"3:04\n그다음에~",
  "3:04", "3:04", "", "그다음에~", "",  


💎 Claude
필드명은 영어로 통일
`[\s\S]+` :  followed by one or more characters (which can be any character including whitespace
this part does correctly include newlines, but since there's nothing directly after the colon before the newline, the capture group ends up not matching anything right there.
:\s* accounts for any whitespace characters (including newlines) that might appear after the colon


mongoDB - duplicate : upsert로 변경. strsub
-> duplicate의 여부는 video 

```
db.collection("messages").updateOne(
{uuid:this.uuid, 
"key.id":new_message.key.id}, {$set: {uuid: this.uuid, ...new_message}}, {upsert:true})

```

---

💎 retry 로직을 잘 달아야함.

