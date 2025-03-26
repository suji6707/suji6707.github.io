+++
title = '도커 기본'
date = 2024-02-25T00:20:22+09:00
draft = true
+++

## 도커파일 구조

Dockerfile에서 여러 파일과 폴더를 복사하는 작업을 더 깔끔하게 처리하기 위한 폴더 구조와 Dockerfile 작성법을 개선할 수 있습니다. 특히, Lambda 함수에 필요한 파일들만 효율적으로 복사하도록 폴더 구조를 재구성하면 좋습니다.

아래는 몇 가지 제안입니다:

### 제안 1: src 디렉토리 활용
Lambda 함수에 필요한 모든 파일을 `src` 디렉토리 아래에 두고, `src` 디렉토리 전체를 복사하는 방법입니다. 이 방법은 파일을 개별적으로 복사하는 번거로움을 줄일 수 있습니다.

**폴더 구조:**
```
project/
│
├── src/
│   ├── handler-producer.py
│   ├── utils/
│   │   ├── some_util.py
│   │   └── another_util.py
│   └── other_module/
│       ├── module_file.py
│       └── ...
├── Dockerfile
└── requirements.txt
```

**Dockerfile:**
```Dockerfile
FROM public.ecr.aws/lambda/python:3.8

# Install dependencies
COPY requirements.txt ./
RUN pip install -r requirements.txt

# Copy the entire src directory
COPY src ${LAMBDA_TASK_ROOT}/

# Set the handler
CMD ["handler-producer.handler"]
```

이 방법은 `src` 디렉토리 아래에 있는 모든 파일과 폴더를 복사하여 Lambda 함수에서 사용할 수 있게 합니다.

### 제안 2: 필요 파일만 별도의 디렉토리에 모으기
Lambda 함수에 필요한 파일들만 별도의 디렉토리에 모아두고, 그 디렉토리만 복사하는 방법입니다. 이 방법은 불필요한 파일이 복사되는 것을 방지할 수 있습니다.

**폴더 구조:**
```
project/
│
├── lambda/
│   ├── handler-producer.py
│   ├── utils/
│   │   ├── some_util.py
│   │   └── another_util.py
│   └── other_needed_file.py
├── Dockerfile
└── requirements.txt
```

**Dockerfile:**
```Dockerfile
FROM public.ecr.aws/lambda/python:3.8

# Install dependencies
COPY requirements.txt ./
RUN pip install -r requirements.txt

# Copy only the lambda directory
COPY lambda ${LAMBDA_TASK_ROOT}/

# Set the handler
CMD ["handler-producer.handler"]
```

이 방법은 Lambda 함수에 필요한 파일들만 복사하여 Docker 이미지의 크기를 줄이고 복사 시간을 단축할 수 있습니다.

### 추가 검색어:
- "Dockerfile copy multiple files and directories"
- "Organize files for AWS Lambda deployment"
- "Efficient folder structure for Docker and AWS Lambda"
- "Best practices for Dockerfile COPY command"

이 두 가지 방법 중 하나를 선택하여 Dockerfile에서 파일 복사 작업을 더 효율적으로 관리할 수 있습니다.
