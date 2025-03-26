+++
title = 'python'
date = 2024-08-22T00:20:22+09:00
draft = true
+++

### UV
uv venv 명령어는 현재 디렉토리에 .venv라는 이름의 폴더를 생성합니다. 이 폴더가 바로 가상 환경
- UV(UltraFast Python Dependency Resolver)
- .파이썬 인터프리터 복사: 시스템에 설치된 파이썬 인터프리터(예: python3.12)의 복사본을 .venv 폴더 안에 생성
-  pip와 같은 도구들을 .venv 폴더 안에 설치

source .venv/bin/activate: 가상 환경 활성화
- 셸에서 python이나 pip 명령어를 실행할 때, 시스템 전역 파이썬 인터프리터나 pip 대신 가상 환경 내의 것을 사용
- 셸의 PATH 환경 변수가 일시적으로 변경
    - PATH의 맨 앞에 가상 환경의 bin 디렉토리가 추가됨.
    - 터미널에서 명령어를 입력했을 때 시스템 전체 경로보다 가상 환경의 bin 디렉토리를 먼저 검색

---
### 🍊🍊🍊 파이썬 실행 방법 기억.
우리는 poetry 쓰고있음

1. source .venv/bin/activate
python main.py

2. poetry shell
-> 알아서 .venv/bin/activate 실행됨
python main.py

3. poetry run python main.py

4. .venv/bin/python3 main.py
-> pm2에서 이렇게 실행하고 있음. 직접 파이썬 실행

---

#### 🍋 파이썬 실행환경
- subprocess -> shell 환경으로 바로 실행시 
python으로 하면 문제가 생김. 전역 python으로 실행해버려서 해당 가상환경내 설치한 모듈들(패키지)를 찾을 수 없음
- 이 경우 두 가지가 있는데
	- pm2 run.json에서 
	`"script"      : "source .venv/bin/activate && 
	python ./src/database-backup/main.py",`
	- activate를 해줘야 실질적으로 가상환경내 파이썬이 활성화되기 때문에.
- 또는 shell로 실행시에도 `.venv/bin/python`으로 실행하도록 하는것. 환경변수에 path 넣어두고

---
##### pyenv 설정
나중에 깃에 템플릿 올려두고 쓰자.

🟥🟥 (정리!!)
1. pyenv local 3.11 -> .python-version 생심됨
2. poetry.toml 생성
3. poetry init -> .venv 및 pyproject.toml 생성됨
4. pyproject.toml 안에 `[tool.pyright]` 부분 넣기
5. poetry shell -> which python이 .venv/bin/python 로 바뀜
- or 5만 해도 알아서 3 결과가 생성됨


🟥 (기본)
```
> poetry init
poetry.toml 생성
pyproject.toml 수정 -> 파이썬 버전 체크
> pyenv local 3.12.3
> poetry shell
> which python : /.venv/bin/python인지 확인
> poetry add : 라이브러리 설치후
좌측 익스텐션 .venv `Export Environment` 버튼

```

poetry는 npm, pnpm 같은 패키지 매니저(종속성 관리),
pyenv는 파이썬 버전관리 -> nvm (node version manager에 대응)
- 그래서 `pyenv local 3.12.3` = nvm use 버전과 비슷함(`.python-version` 파일 생성)
-> 버전을 수정한 경우 다시 poetry shell로 .venv를 생성해야함. 특정 버전에 종속된거라서. 

🟦 poetry
- **pyproject.toml 파일 통해서 파이썬 프로젝트 종속성 관리.**
- **프로젝트별 가상 환경을 자동으로 생성. `poetry shell`**
- 그 외
poetry build 명령어를 사용하여 배포 가능한 패키지를 생성
poetry publish 명령어를 사용하여 패키지를 PyPI(Python Package Index) 또는 다른 패키지 저장소에 배포

🟦 프로젝트 초기화
1. `poetry init`
필요시 pyenv 초기화 경로설정 하기
```
eval "$(pyenv init --path)"
which python
```

poetry.toml 파일 만들기
```
[virtualenvs]
in-project = true
```

2. 가상환경 설정
```
// poetry.toml
[virtualenvs]
in-project = true

// pyproject.toml
[tool.pyright]
venvPath = "."
venv = ".venv"

[virtualenvs]
in-project = true
```
- 두 가지 추가한 후 `poetry shell` 실행하면 프로젝트내 .venv 폴더가 생성됨.
	- 그렇지않으면 글로벌한 곳에 저장되어버림
	- shell 실행 후에는 which python이 /{현재 프로젝트}/.venv/bin/python 로 바뀜!

3. 라이브러리 import하기 전에
python environment 익스텐션 좌측에서
.venv `Export Environment` 버튼 눌러주기.
-> 수많은 파이썬 버전중에 어떤 것의 라이브러리를 쓸 지 정해줌

4. git ignore 설정

/Users/heojisu/.pyenv/versions/3.12.3/bin/python
poetry env use /Users/heojisu/.pyenv/versions/3.12.3/bin/python 

---
##### pyenv 버전 문제
pyenv 삭제하고 다시 설치했음.
기본경로 파이썬 버전에 맞춰서 pyenv가 설치되는데,
pyenv install python 3.12.3 하려니까 문제생긴듯?

pyenv global 3.11 -> 글로벌 버전은 성능 업글된 3.11로 맞추고
which python 했을 때 pyenv local로 설정한 버전의 파이썬이 실행되도록 
.zshrc의 PATH를 변경함. (13 -> 12)
```
 12 export PATH="/Users/heojisu/.pyenv/shims:$PATH"
 13 #export PATH=/opt/homebrew/opt/python@3.10/libexec/bin:$PATH
```

🟡 참고
Pyenv 버전관리 도구는 여러 버전의 Python을 쉽게 설치/전환하도록 해주는데
- global 버전: 기본 버전 (`pyenv global ~`)
- local: 특정 디렉토리에서 사용할 버전 (`pyenv local ~`)

Pyenv Shims 디렉토리
- 경량 wrappers. pyenv가 관리하는 모든 python 버전의 실행파일에 대한 심볼릭 링크를 포함.
- 사용자가 실행하는 모든 Python 명령을 가로채서, 현재 설정된 Python 버전 (global, local 또는 shell)에 따라 올바른 버전의 Python을 실행하도록 함.

---
##### pyenv install python 에러
(결론) openssl@3을 시스템의 기본 OpenSSL 버전으로 설정함으로써 문제가 해결되었을 가능성이 큼.
일부 파이썬 버전은 openssl 3버전이랑 호환될 수도.

Python을 컴파일할 때, 특히 HTTPS 요청을 처리하는 라이브러리와 같은 기능을 위해 OpenSSL 라이브러리가 필요합니다. Pyenv는 Python을 소스 코드에서 빌드하는 방식으로 설치합니다. 이 과정에서 OpenSSL 라이브러리를 찾을 수 없거나 호환되지 않는 버전을 찾는 경우 문제가 발생할 수 있습니다.구체적인 문제는 다음과 같습니다:
1.OpenSSL 버전 불일치: Pyenv가 사용하는 OpenSSL 버전과 시스템에 설치된 OpenSSL 버전이 일치하지 않거나 호환되지 않을 수 있습니다.
2.경로 문제: Pyenv가 Python을 컴파일할 때 올바른 OpenSSL 라이브러리 경로를 찾지 못할 수 있습니다.

1번 방식 (현재)
```
export CPPFLAGS="-I$(brew --prefix)/include"
export LDFLAGS="-L$(brew --prefix)/lib"
>> brew link openssl@3
```
- Homebrew의 기본 설치 경로: usr/local (/Users/heojisu)
- LDFLAGS와 CPPFLAGS는 컴파일러와 링커가 소프트웨어를 빌드할 때 사용하는 환경 변수
	- CPPFLAGS는 C/C++ 전처리기가 헤더 파일을 찾을 수 있도록 포함 경로를 지정
	- LDFLAGS는 링커가 라이브러리를 찾을 수 있도록 라이브러리 경로를 설정
- brew link openssl@3
	- openssl@3의 바이너리와 라이브러리를 Homebrew의 기본 경로에 심볼릭 링크로 연결. 이는 openssl@3을 시스템의 기본 OpenSSL 버전으로 설정하는 효과

2번 방식
```
export PATH="/usr/local/opt/openssl@3/bin:$PATH"
export LDFLAGS="-L/usr/local/opt/openssl@3/lib"
export CPPFLAGS="-I/usr/local/opt/openssl@3/include"
```
- 2번의 경우 OpenSSL 버전의 경로를 명시적으로 설정한것이고
- 1번의 경우 Homebrew 기본경로를 사용하고 brew link 를 통해 @3 version을 openSSL 기본 버전으로 설정한 것.