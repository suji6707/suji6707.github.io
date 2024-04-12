+++
title = '카테고리 분석기'
date = 2024-02-25T00:20:22+09:00
draft = true
+++
## Introduction

This is **bold** text, and this is *emphasized* text.


##### 어제 변경한 내용들
에러 처리에 관한 것.
- 카테고리 매출 집계에 실패했다는 것의 기준이 무엇인가?
	- 기준 키워드 개수가 100개 (또는 전체 길이)에 못 미칠 때다.
- 이 경우 카테고리 매출을 저장하지 않고 다음 서브카테고리로 넘어간다.
- 그리고 DB에 저장된 서브카테고리 개수가 원래 개수에 못미치면
배치파일을 다시 돌리도록 시스템을 설정한다.  	

로그 및 throw Error 지점
- 카테고리 매출 저장에서 실패하면 throw Error.
이는 이 함수를 부른 곳으로 에러를 전파할거고,
서브 카테고리를 돌리는 for문에서 catch 블록으로 이동하면 
최종적으로 로그를 남기게 된다. (여기서 한번만 하면 됨: category id 명시하고)
다음 서브카테고리는 문제없이 진행해야하므로 맨 바깥에서 throw는 하지 않는다. 
-+ 슬랙 알림을 날린다.
- '저장'에 실패한 것과 별개로 그 매출집계가 유효하지 않을 때는?
각 키워드 매출을 저장하되, 그 전체 개수(keywordSales.length)가 인풋 키워드 갯수에 못 미치면 throw Error 를 던진다. 
- 그렇다면 이 카테고리 매출 로직에서 throw Error가 되는 경우는 두 가지인데,
저장에 실패했을 때 보다는 후자, '키워드 개수가 모자랄 때'일 것이다. 
> 키워드 개수가 모자랄 때는 어떤 카테고리에서 몇 개 키워드가 실패했는지 logger.error만 찍고,
> 에러가 상단으로 전파되었을 때 거기서 에러발생 카테고리 로그 및 슬랙 에러를 보내면 된다. 

🍎 중요한 건, save나 크롤링이 실패했다고 해서 계속 null을 리턴할 것이 아니라
적절한 시점에 throw Error를 하고 상위 함수로 전파시켜 로그를 남기게 하는 것이다. 

---
### 배포

Docker 커맨드 구성: 추출된 디렉토리는 Docker 컨테이너의 /LocalBuild/envFile/ 디렉토리로 볼륨 마운트되며, 환경 변수 파일의 이름은 컨테이너 내의 ENV_VAR_FILE 환경 변수로 설정됩니다. 이를 통해, 컨테이너 내부에서 해당 환경 변수 파일을 참조하거나 사용할 수 있게 됩니다.
```shell
if [ -n "$environment_variable_file" ]
then
    environment_variable_file_path=$(allOSRealPath "$environment_variable_file")
    environment_variable_file_dir=$(dirname "$environment_variable_file_path")
    environment_variable_file_basename=$(basename "$environment_variable_file")
    docker_command+=" -v \"$environment_variable_file_dir:/LocalBuild/envFile/\" -e \"ENV_VAR_FILE=$environment_variable_file_basename\""
fi
```


---
TODO
- 전체 에러처리. 지금은 쿼리만 잘못넣어도 500 에러인데
tracking, pre-pay 등 참조해서 에러처리 맨 바깥에서 또는 일괄적으로 어떻게 처리하는지 보기.


---
🍏우선 지금 해야 할 것.
- git pull(env.dev 어떻게 바꼈는지 보기)
- 프로젝트명은 바꿔주셨음 (분명 config 폴더가 있었는데 왜 안보이지)
	- /datalab/.git/config를 어떻게 수정하지?  
- develop 브랜치로 작업내용 전부 옮기기 -> code build에서 develop 브랜치명망 자동 빌드/실행하도록 되어있음
	- 👗 일단 main에서 완성하고 develop으로 merge 시키기.  
- auth 모듈 레거시용으로 따로 만들기!!
- 👗 스웨거 설정
	- 로그인 붙이기. 딱 스웨거 테스트용으로. 레거시 auth 모듈 붙여서 sessionChecker 기능 넣어야하므로
	- api-docs 설정은 수영님이 develop 브랜치로 nginx 등 해주셨음.
	look up path로 proxy_pass http://datalab-api.dev-itemscout-ns:3000; 
> 이 namespace?는 어디서 찾아볼 수 있지? 컨테이너 안에선 일종의 DNS인셈
- health check API 하나 만들어야할듯? 선정산이나 tracking-api 참조

---
(금요일) 오늘은 복습도 좋지만.
권한체크를 어떤식으로 하는지 기존 코드 + 다른 사례/글들을 한번 보는것도 좋은 방법이다.

##### Auth 유닛테스트?
- 핵심은
	- 로그인을 하면 액세스/리프레시 토큰이 모두 발급되고
	- 액세스 토큰은 보통 30분이면 만료, 리프레시는 admin(1일)/PC(7일)/그 외 90일로 훨씬 길다는 것.
	- 헤더에 항상 토큰을 실어서 요청하게 되는데, 액세스가 만료이면 리프레시를 주고 액세스/리프레시 갱신
	- 리프레시가 만료되면 무조건 로그아웃. 
- 테스트
	- 사용자의 토큰이 갖게 될 정보를 검증하고자 함.
	- 토큰에 포함된 사용자 ID, 권한, 만료 시간 등의 정보가 올바르게 인코딩되어 있는지 확인
		- 토큰 생성 인자로 들어간 payload: Map, 액세스/리프레시 만료일: Instant(time) 타입 체크
	- 토큰이 변조되지 않았는지 검증
	토큰이 유효한 기간 내에 있는지 확인
	토큰에 포함된 정보가 실제 사용자 정보와 일치하는지 검증
- 

- TokenProvider 클래스: 유저에 따라 인증토큰을 생성
	- userRepository: 유저정보(id/password 체크), 
	JwtProvider(액세스/리프레시 토큰 생성), 
	timeServer(만료시간 설정)
	- newToken(id, isPc, isAdmin): payload 맵을 만들어 액세스 토큰을 생성하고, 
	isPc/admin 정보에 따라 만료시간을 설정해 리프레시 토큰을 만듦
	- JwtProvider는 

---
##### CI/CD
지금 수영님은 code build 만드는 중.
- 여기에 $ENVIRONMENT 환경변수 넣게 되어있음
사실 서버 실행 설정은 buildspec.yml에 다 있음

🍏 사실 CI/CD란게 대단한 건 아님.
아마존에서 관리되는 ECR 레지스트리가 있고
로컬에서 git push를 하면 여기로 가고,
자동으로 이미지를 빌드, 컨테이너를 재실행? 시켜줄 뿐.
(작업정의에 (docker) image 해시가 계속 바뀜)

---
### Auth 모듈
기존 서버 -> Datalab으로 요청갈 때 
기존 서버단의 sessionChecker 를 쓰지않고
Datalab의, 기존 서버 auth와 호환되는 레거시 auth 모듈을 쓰도록 하자.

---
#### 질문
- 인기 키워드가 없는 서브카테고리의 경우 결과에서 제외해야하는데
for문을 돌 때 continue로 스킵하려면 그 아랫단에서 계속 null을 리턴하면서 최종 맨 바깥 for문에서 if (! ) continue를 해야하는 번거로움.. 여러 레이어를 타야해서 어쩔수없는건가.
- DB에서 keywordIds를 봤을때 빈배열이면 throw Error를 하면 젤 편하겠지만



#### 에러 처리
- 오히려 컨트롤러 말고 서비스단에서 throw Error하면 될듯

#### DB / 캐싱
- 크롤링 전에, categoryId가 category_sales_aggregate 테이블에 있고 year, month가 같으면 해당 항목 보내주는걸로 바꿔야함.

---
#### Nest CLI and scripts
왜 nest start를 해야 빌드 후 watch가 제대로 작동할까?
- Nest는 타입스크립트 기반이라 실행 전 JS로 먼저 컴파일되어야하는데
- Nest CLI를 보면 nest build시 ts 컴파일러가 JS로 변환하며(이때 tsconfig.json 참조)
- nest start를 함으로써 빌드된 JS를 실행한다. 
- 여기서 nest start는 빌드가 되어있음을 ensure(빌드과정도 포함)하므로, 애초에 pnpm dev: nest start --watch 로 넣어버리면 젤 편하다.


```typescript
const a: number = 1;
console.log(a) // 1
```

`a` is number
