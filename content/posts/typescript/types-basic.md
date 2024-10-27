+++
categories = ['Typescript']
series = ['Typescript']
date = 2024-09-23T22:50:44+09:00
draft = true
+++
## 타입 기초

#### 에러처리
- error를 axios 에러 인스턴스로 처리
- error.response.data로 에러코드/메시지 접근
```ts
try {
	const response = await axios.get(url, {
		params: params
	});
	console.log('response', response.status);
} catch (err) {
	if (axios.isAxiosError(err) && err.response) {
		console.log(err.response.data);
	} else {
		console.error(err);
	}
	throw new Error('Failed to get Cross Border Data');
}
```