+++
title = 'Vue'
date = 2024-09-26T00:20:22+09:00
draft = true
+++
## Vue 베이직

🔮CSS 선택자
- 부모 자식: 띄어쓰기 ( > 는 바로 아래 자식만)
- button.copy : 버튼 태그중 copy 클래스 요소만

---
### ㅇ



---
### 파일업로드
꼭 파일을 어딘가에 저장해둬야하는 건 아니다.
1. 사용자가 화면을 캡처 (Blob, File 객체 생성)
2. 클라이언트로 이미지를 multipart/form으로 전송
3. 클라이언트는 FileReader 인스턴스로 파일을 읽고 즉시 base64 string으로 변환(`readAsDataURL`)

(참고)
#### FileReader: readAsDataURL() method
파일 읽기가 완료되면 `e.target.result` attribute에 base64가 담긴다.
- data: URL representing the file's data as a base64 encoded string.
`data:[<mediatype>][;base64],<data>`
, 이후가 데이터라고 보면 됨.


---
### @click vs @change
맞습니다. Vue.js에서 `@` 기호는 `v-on` 디렉티브의 축약형으로, 컴포넌트에 이벤트 리스너를 연결하는 데 사용됩니다. 그리고 이벤트 리스너에 사용되는 이벤트 이름들은 DOM 이벤트 명세에 따라 정의되어 있으며, 각각 특정 상황에서 발생하도록 설계되었습니다.

`<input type="file">` 요소의 경우, `@click` 이벤트는 사용자가 파일 선택 버튼을 **클릭할 때** 발생하지만, 파일 선택 대화상자가 열리는 시점까지만 관여합니다. 실제로 **파일을 선택하고 해당 내용을 브라우저가 읽어 들이는 시점은 `@change` 이벤트가 발생하는 시점**입니다.

따라서 파일 입력 요소에서 선택된 파일 데이터를 다루려면 `@change` 이벤트를 사용해야 합니다. `@click` 이벤트는 파일 선택 대화상자를 여는 것을 제어하는 데 사용할 수는 있지만, 파일 선택 자체를 다루지는 못합니다.

**요약:**

- `@click`: 파일 선택 버튼을 클릭했을 때 발생. 파일 선택 대화상자가 열리는 시점까지만 관여.
- `@change`: 파일을 선택하고 브라우저가 해당 내용을 읽어 들였을 때 발생. 실제 파일 데이터를 다루려면 이 이벤트를 사용해야 함.

Vue.js에서 `@` 기호는 단순한 임의의 이름이 아니라, DOM 이벤트를 효과적으로 처리하고 컴포넌트 로직과 연결하기 위한 중요한 부분입니다. 

---
## 이미지 리사이징 결과
**참고:** Divisor는 원본 이미지 파일의 가로, 세로 길이를 얼마로 나누었는지 의미. 

| Divisor | 가로 크기 | 세로 크기 | Base64 문자열 길이 | 파일 크기 | 결과 |
|---|---|---|---|---|---|
| 1.0 | 5000 | 3333 | 1,528,284 | 7,101 KB | X |
| 1.2 | 4166 | 2777 | 1,121,188 | 840 KB | X |
| 1.3 | 3846 | 2563 | 987,668 | 741 KB | O |
| 1.4 | 3571 | 2380 | 878,956 | 659 KB | O |
| 1.5 | 3333 | 2222 | 789,932 | 592 KB | O |
| 2 | 2500 | 1666 | 511,704 | 384 KB | O |
| 5 | 1000 | 666  | 133,496 | 100 KB | O |
| 10 | 500  | 333  | 48,172  | 36 KB  | O | 


---

