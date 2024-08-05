+++
title = '카테고리 분석기'
date = 2024-02-25T00:20:22+09:00
draft = true
+++

## 배포 및 CI/CD

🍋 배포 하면 젤 먼저 .env.development가 먼저 떠올라야 한다.
그리고 기존 플젝 보면서
아, build.yml이 필요하구나
레거시 서버에는 MSI-API 설정이 필요하구나 등.




### ⛑️ Too many connection 원인?
conn limit은 5로 해도 문제없음. 느리긴 한데 .. 반환이 되고있다는 뜻임.

궁금: 
- 실서버 max connection은 얼마?
조치:
- app에서 커넥션 갯수 제한(DB_CONN_LIMIT)
    - max connection이 100개면 app A, B, C 앱 단에서의 커넥션 수를 다 합한게 100개 넘으면 안된다는 말임.
- db엔진에서 sleep 상태의 세션 종료 시간 조정(wait_connection)
    - 💎 conf.d에서 해당 옵션 찾아보기! 이걸 검색해낼 수 있어야함..

#### 💎 변경값  
- 1번은 .env 설정 변경했구요(어제처럼 컨테이너에 직접 주입) = 5
- 2번은 rds 파라미터 그룹 변경 입니다 = wait_timeout 60(s)
sleep 된 세션이 8시간동안 사라지지 않아서 1분으로 변경
    - 설정방법: RDS > 해당 DB > DB instance parameter group
    - 현재 dev는 micro라서 max_conn 60개고 
    - prd는 R3.xlarge: 2540 -> 수백개는 거뜬하고. 앱단의 limit은 32개.
        - 근데 이것도 100명이 한번에 요청하기에는 빡세보임. 빨리 반환해주지 않으면..
#### 🍎 랭킹추적 배포 prd 적정수치에 대한 생각
- dev 10, prd 32

클라이언트 안켜지면
- conf.d를 수정하고 재시작하거나
- mysql restart 후 접속해서 SET 명령어로 설정한다.


---
#### 🍋 TODO

---

### buildspec.yml
**소스코드를 가지고 실행하고, PUSH까지 하게 됨(ECR 배포)**

스테이지 단계별로
- 빌드 단계에선 buildspec.yml
- deploy(배포) 단계(post-build?)에선 imagedefinition.json이 필요 
	- imagedefinition.json 경로를 다이나믹하게 만드려고 buildspec에서 `"imageUri":"%s"`로 동적 생성함.

artifacts: files를 보고 어떤 컨테이너를 올리고 내릴지 결정.
클러스터가 바꿔치기 되는 것. 

메인 브랜치를 배포할 때는 파이프라인에 prd를 추가하면 됨.
지금은 CI/CD 파이프라인이 dev 브랜치로만 설정되어있는거고.
- 생각해보면 이 파이프라인이라는게 
코드를 push하면 깃랩 -> AWS code commit으로 가고,
이게 build - ECR push - deploy(컨테이너) - ECS 클러스터, 서비스까지 가는건데.
각 단계마다 중요한 설정파일(커맨드 정의, json/yml)과 환경변수가 있는거고.
내가 buildspec을 직접 scratch부터 만들어 봄으로써 실제 운영환경에 어떻게 반영되는지 배워야.

(IaC 프로젝트 참고)
- Docker 커맨드 구성: 추출된 디렉토리는 Docker 컨테이너의 /LocalBuild/envFile/ 디렉토리로 볼륨 마운트되며, 환경 변수 파일의 이름은 컨테이너 내의 ENV_VAR_FILE 환경 변수로 설정됩니다. 이를 통해, 컨테이너 내부에서 해당 환경 변수 파일을 참조하거나 사용할 수 있게 됩니다.
```shell
if [ -n "$environment_variable_file" ]
then
    environment_variable_file_path=$(allOSRealPath "$environment_variable_file")
    environment_variable_file_dir=$(dirname "$environment_variable_file_path")
    environment_variable_file_basename=$(basename "$environment_variable_file")
    docker_command+=" -v \"$environment_variable_file_dir:/LocalBuild/envFile/\" -e \"ENV_VAR_FILE=$environment_variable_file_basename\""
fi
```

### AWS code build
SourceArtifact -> BuildArtifact 와 같이 트리거를 계속 보내면서
소스코드부터 최종적으로 ECS 배포까지 가는 것. HOW??

소스코드 레포지토리는 깃랩을 쓰고있고 
운영서버 구성 aws를 쓰고있어서.
서로가 모르는 상황이고.
aws가 이 소스코드를 알아야 배포를 할텐데.
이 코드를 aws에도 - code commit임.
서버 레포지토리 깡통을 만든다음
깃랩에서 액션이 일어나면 저기로 미러링이 됨.
Mirroring repositoties가 있음. 
코드커밋 주소가 있고 

로컬에서 push하면 깃랩 -> aws로 가게되있음.

aws 코드가 업뎃됨. 
그다음 파이프라인에서 흐름.

source - build - deploy 스테이지가 있는데
순차적으로 
source 스테이지에서는 우리가 걸어둔 develop 브랜치. 
트리거가 되어서 zip파일로 소스코드를 묶어서 S3로 업로드를 할거고

그 올린걸로 build 스테이지에서
방금 생성한 SourceArtifact zip 파일을 가져와서
환경변수를 적용(적절한 변수를 대입)
**코드 컨테이너 하나 내부적으로 띄워서 S3 다운받아서 그 소스코드로 실행하는게 buildspec.yml**

#### 💎 정리
> 1. 소스코드를 가지고 AWS에서 미리 설정한 .env.OOO를 바인딩
> 2. 그 zip 소스코드를 이미지로 빌드해 ECR에 push해야하는데, 
이걸 datalab 프로젝트에 정의된 buildspec.yml이 IaC로 다 실행하게 되어있음.
> 3. buildspec.yml로 이미지 빌드, 태그, 푸쉬까지 진행하고
> 4. AWS deploy에서 필요로하는 imagedefinitions URL은 동적으로 넣어서 배포함

(참고)
- (빌드 단계에서?) $ENVIRONMENT 환경변수 넣게 되어있음
사실 서버 실행 설정은 buildspec.yml에 다 있음

- 사실 CI/CD란게 대단한 건 아님.
아마존에서 관리되는 ECR 레지스트리가 있고
로컬에서 git push를 하면 여기로 가고,
자동으로 이미지를 빌드, 컨테이너를 재실행? 시켜줄 뿐.
(작업정의에 (docker) image 해시가 계속 바뀜)

---
### API-DOCS 프로젝트
api-docs용 컨테이너 따로 있음.
로드밸런서에서 api-docs로 트래픽을 보내면
Nginx 서버가 받아서 80번 포트로 API DOCS 연결
> Q. 왜 api-docs 앞단에 Nginx를 붙여놨을까? 모든 프로젝트의 dev용 api-docs를 묶는 하나의 dev-api-docs 서버가 있고, 로드밸런서로부터 트래픽을 받았을 때 이것을 적절한 프로젝트 docs로 라우팅시키는게 Nginx 역할인듯.

``` shell
# nginx.development.conf
# datalab
location ^~ /api-docs/datalab {
    proxy_pass http://datalab-api.dev-itemscout-ns:3000;
	# 컨테이너간 통신 위한 lookup path로 보내줌
}
```

### Swagger
- nestjs 프레임워크 레벨에서 swagger로 PATH를 설정해야 스웨거가 요청을 받음.
- 컨트롤러 테스트 가능.
- 로그인 api: tracking-api의 경우 레거시를 통해서만 가므로 가상으로 로그인할 수 있게 위에 붙여놓은 것. 쿠키를 달아서 요청보내게 해줌. 

---
### 🟣 카테고리 분석기 배포 프로세스

1. 새 인스턴 생성. 
2. dev 서버 -> 새로운 서버로 요청하기 위한 도메인 설정
- dev 서버에 DNS A 레코드 추가해서 새 서버 IP 주소 가리킴
- 보안 설정. SSH 및 필요한 포트만 열기
3. 도커
- docker run -p 80:3000 -d <이미지 이름> 
컨테이너 3000포트를 호스트 80 포트에 바인딩
4. 로드밸런서 설정
- 대상 서버에 새로운 서버 추가
5. 이미지 빌드 후 컨테이너 실행



---
4번 설정 ㄴㄴ.
3번도 80으로 붙일피요 없음. 인스턴스끼리 통신은 3000 : 3000 에 바인딩
포워딩도 안해도됨.

1  인스턴스 - 

ECS 클러스터.
풀을 전체 관리하는 네임스페이스같은거.
개발,운영 네임스페이스.
안에 들어오면 

쿠버네티스에서 서비스 - pod. 묶어주는 역할.
특정 논리적으로 여러개의 파드를 하나로 묶어서 하나의 엔드포인트를 밖으로 노출하기 위한.
api-svc 등을 하나의 서비스로 묶어준다.

🟣 클러스터와 컨테이너 개념. 
prd-item svc cluster 보면
gateway에 5개가 떠있는데 컨테이너. 독립적으로 떠있는데.
직접 바라볼 수 없는데
밖에서 바라볼때 내부에서 바라볼때 
(💎클러스터는 일종의) 로드밸런서라 보면 됨(컨테이너 내/외부 통신을 돕는). gateway 같은 서비스는.
안에 다섯개의 컨테이너가 떠있음. 
해당 컨테이너가 정의를 보고 띄움. 태스크 정의.

컨테이너 정의서를 작성 - 태스크 정의.
tdef task definitoin
프로덕션 환경 네이밍. 

🟣 왜 task def가 계속 생성되나? 
빌드할 때마다 이미지 경로가 들어감.실제 AWS 프라이빗 이미지 저장소 주소인거고. 
CI/CD에 의해 이미지 새로 생성되고 빌드이미지 번호를 가지게 됨.
배포할 때마다 definition이 업뎃이 되고 이걸로 컨테이너가 올라가는.

0.1 이렇게 로컬에서 배포를 하고 ECR로 푸쉬,
그 이미지를 가지고 컨테이너를 만듦.

0.1로 하드코딩해놓고 함.

포트매핑은 따로 ㄴ
3000 HTTP 
🟣 포트이름 server-3000-tcp 비즈니스 검색이름.
컨테이너간 통신위한 수단으로써 (이러한 주소로 가라 . 포트이름. )
내부적으로 통신하는 서비스인데 Cloud Map ECS Service Connect같은. 
DNS를 룩업하듯이. 컨테이너를 찾으려고 질의를 함.

2-4-

--------
1. ECR 서비스 정의서 작성. 
- 이미지. 
띄우기 위해 서비스로 생성해야하는데.
🟣-> 서비스는 (컨테이너와 그 안의 task를) 묶기위한 수단으로서. 단일 엔드포인트를 내보내기 위한 수단.
컨테이너끼리 통신할 수있는 수단.
🟣 실제로 배치를 돌릴때는 서비스를 생성하지 않음. 

2. 작업정의. 클러스터
그다음 .sh 마지막줄 CLI 명령
- terraform-admin 사용자 계정. 

3. 태깅 -  태깅기준 검색 위해 . 

-> GUI보다는 CLI 버전으로 작성해둔게 있음.
IAC 프로젝트. 
itemscout- aws-cli 에 보면
otel collector 
