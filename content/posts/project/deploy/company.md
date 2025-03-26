+++
title = '배포 관련 시행착오'
date = 2024-07-22T00:20:22+09:00
draft = true
+++

### pnpm frozen lockfile 에러
💎pnpm frozen lockfile 에러. 예전에 postinstall로도 에러 났었음.
도커 빌드시 ERROR: process "/bin/sh -c pnpm install --prod --frozen-lockfile" did not complete successfully: exit code: 1

이 에러는 `husky`라는 패키지가 `prepare` 스크립트에서 실행되도록 설정되어 있지만, 실제로 설치되어 있지 않아서 발생하는 문제입니다. `husky`는 일반적으로 개발 환경에서 사용되는 패키지로, `devDependencies`에 포함되어 있을 가능성이 높습니다. `pnpm install --prod --frozen-lockfile` 명령어는 `devDependencies`를 설치하지 않기 때문에 `husky`가 설치되지 않아 에러가 발생합니다.

이 문제를 해결하기 위한 몇 가지 방법은 다음과 같습니다:

1. **`husky`를 `dependencies`로 이동**: 만약 `husky`가 프로덕션 환경에서도 필요하다면, `package.json`에서 `husky`를 `dependencies`로 이동하세요.

2. **`prepare` 스크립트 수정**: `prepare` 스크립트가 프로덕션 환경에서 실행되지 않도록 조건을 추가하거나, 프로덕션 환경에서는 이 스크립트를 제거하세요.

3. **빌드 환경 분리**: 프로덕션 빌드에서는 `prepare` 스크립트를 실행하지 않도록 빌드 환경을 분리할 수 있습니다.

4. **`pnpm install` 명령어 수정**: 만약 개발 환경에서만 필요한 패키지라면, 프로덕션 빌드 시 `prepare` 스크립트를 실행하지 않도록 설정하거나, `pnpm install`을 통해 모든 패키지를 설치한 후 프로덕션 빌드를 진행할 수 있습니다.

이 문제는 `pnpm` 버전과는 직접적인 관련이 없으며, `pnpm`의 설치 옵션과 `package.json`의 스크립트 설정에 의해 발생하는 문제입니다.

---

# gulp-babel 중복호출 현상

1.로그 추가
2.server에서 호출하는건 watch에서 빼기

'./bin/www' 등은 실행 파일이라 nodemon으로 실행함.

watch -> /dist 일부 변경 -> /dist 전체를 감시하던 task가 재시작 (3개가 전부..)
근데 바벨이 다 안끝난 상태에서 자꾸 

```js
gulp.task('start-server', async () => {
    return nodemon({
        script: './bin/www',
        watch: DEST.SERVER,
		// exec: 'node -r ./tracing.js'
    });
});

// 해결 !!
gulp.task('start-server', async () => {
    return nodemon({
        script: './bin/www',
        watch: DEST.SERVER,
		delay: 1000,       // 변경 감지 후 재시작 전 지연시간 추가
        ignoreInitial: true // 초기 실행 시 재시작 방지
		// exec: 'node -r ./tracing.js'
    });
});

```
---

### nodemon delay
- 디바운싱 기능이라 보면 됨.
- 로컬 개발환경에서만 의미있고, 어차피 실서버는 pnpm build 후 dist app만 실행해서 상관없음.

debouncing은 연속적으로 발생하는 이벤트를 그룹화하여 마지막 이벤트만 처리하도록 하는 프로그래밍 기법입니다.

nodemon의 맥락에서 예를 들어 설명하면:

**debouncing 없을 때:**
```
파일 저장 -> 서버 재시작
파일 저장 -> 서버 재시작
파일 저장 -> 서버 재시작
파일 저장 -> 서버 재시작
```

**debouncing 있을 때:**
```
파일 저장
파일 저장
파일 저장
파일 저장 -> (일정 시간 대기) -> 서버 재시작 1번
```

nodemon에서는 `delay` 옵션을 통해 이러한 debouncing을 구현할 수 있습니다:

```javascript
nodemon({
    script: 'server.js',
    delay: 1000  // 1초 동안 추가 변경이 없을 때만 재시작
})
```

이렇게 하면 파일을 수정하는 동안 서버가 과도하게 재시작되는 것을 방지할 수 있습니다. 특히 여러 파일을 동시에 수정하거나, 코드를 타이핑하는 중에 유용합니다.

이전 문제에서도 이 `delay` 옵션을 추가하여 불필요한 재시작을 줄이도록 제안했었습니다.


---