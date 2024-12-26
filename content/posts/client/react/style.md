+++
title = 'React Styling'
date = 2024-10-25T00:20:22+09:00
draft = true
+++
## React Styling

---
### Styling
JSX는 property 중심, HTML은 attribute 중심
- JSX에서 style에 객체를 넣어도 html에선 string으로 바뀜

<div {...props} />
여기서 props로 전달되는 속성
- clasName
- style

```js
function Box(props) {
  return <div {...props} />
}

const smallBox = (
  <Box 
    className="box box--small" 
    style={{fontStyle: 'italic', backgroundColor: 'lightblue'}}
  >
    small lightblue box
  </Box>
)
```
-> <Box ... 는 결국 <div ... 가 됨. 
