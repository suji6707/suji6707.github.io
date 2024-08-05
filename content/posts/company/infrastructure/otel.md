+++
categories = ['Company']
series = ['OTEL']
tags = ['Node 및 Prisma Otel']
title = "TypeOrm"
date = 2024-07-10T22:50:44+09:00
draft = true
+++
## 🔵 현재 시그노즈 서버 구조 및 TODO

<img width="2477" alt="image" src="https://github.com/user-attachments/assets/0edc59e5-0c72-442c-95bd-224bae7ef16a">
<img width="2515" alt="image" src="https://github.com/user-attachments/assets/b408a63e-b755-4fd2-babb-ddb00d555be2">


---
### TODO
1. ncs-rank에 설정 추가하기. 시그노즈 서버 IP랑.
-> 수영님께 grpc IP 허용해달라고 부탁. 
2. prometheus/server/prometheus.yml 수정 후 docker compose restart.



### ncs 서버에서 signoz 서버로 데이터 전송
🔴 시그노즈 서버에 ncs 추가.
com-apm 172.31.103.160:4317 로 데이터를 보내면 됨.
-> grpc라서??
Q. new OTLPTraceExporter에 안넣고 어떻게 전송하고있지??
- 일단 dev에만 배포해보자. 

Q. conf 파일이 어딨지??
ssh -i ~/auth/aws-itemscout-key.pem ubuntu@172.31.103.160
-> 막혀있음.

우선 IDC dev로 들어가서 구경. 
main.itemscout.io
dev.itemscout.io

ssh -i ~/auth/itemscout_dev.pem ubuntu@dev.itemscout.io
ubuntu@c0.itemscout.io


### 그라파나, 프로메테우스에서 인프라 모니터링하기

1. 프로메테우스 exporter들을 docker 컨테이너로 실행하는 설정
- prometheus-node-exporter
- prometheus-nginx-exporter

```yaml
# prometheus/exporter/docker-compose.yml
version: "3.9"
services:
  # 호스트 시스템의 메트릭 수집. CPU 사용량 등. 9100 포트
  prometheus-node-exporter:
    networks:
      - backend
    image: prom/node-exporter
    ports:
      - "9100:9100"
  # Nginx 서버의 메트릭을 수집합니다. 요청 수, 응답 시간 등 Nginx 관련 메트릭
  prometheus-nginx-exporter:
    networks:
      - backend
    image: nginx/nginx-prometheus-exporter
    ports:
      - "9113:9113"
    environment:
      - SCRAPE_URI=http://172.17.0.1:8088/metrics
networks:
  backend:
```

2. 프로메테우스 서버 설정 정의. 어떤 메트릭을 어디서 수집할지 저장.
- 이 파일만 수정하면 됨! 💎
- targets에 본인 호스트 바라보게 하고. (node, redis, nginx 다 넣으면 될듯?)
- 그다음 docker-compose restart 하면 됨.

🔴 c13~28 까지 지우고 ncs test 서버.



```yaml
# prometheus/server/prometheus.yml
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: 'prometheus'

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    static_configs:
      - targets: ['localhost:9090']

        #  - job_name: 'pushgateway'
        #    honor_labels: true
        #    static_configs:
        #      - targets: ['localhost:9091']

  - job_name: squid
    static_configs:
      - targets: ['localhost:9301']

  # Node exporter
  - job_name: 'node-exporter'
    scrape_interval: 5s
    static_configs:
      - targets:
        - 'main.example.io:9100'
        labels:
          instance: main
      - targets:
        - 'mainapi.example.io:9100'
        labels:
          instance: mainapi
      - targets:
        - 'client.example.io:9100'
        labels:
          instance: client

 # Nginx exporter
  - job_name: 'nginx-exporter'
    scrape_interval: 5s
    static_configs:
      - targets:
        - 'main.example.io:9113'
        labels:
          instance: main
      - targets:
        - 'mainapi.example.io:9113'
        labels:
          instance: mainapi
      - targets:
        - 'client.example.io:9113'
        labels:
          instance: client
      - targets:
              #- 'dev.example.io:9113'
        - 'server-prometheus-nginx-exporter-1:9113'
        labels:
          instance: dev

```

3. 프로메테우스 서버 띄울때 volumes- /prometheus.yml 참고해서 띄운다
```yaml
# prometheus/server/docker-compose.yml
version: "3.9"
services:
  # local server
  prometheus:
    networks:
      - backend
    image: prom/prometheus:latest
    user: "$UID:$GID"
    volumes:
      - ./prometheus-data:/prometheus
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
    ports:
      - "9090:9090"
  grafana:
    networks:
      - backend
    image: grafana/grafana:latest
    user: "$UID:$GID"
    volumes:
      - ./grafana-data:/var/lib/grafana
      - ./grafana.ini:/etc/grafana/grafana.ini
      - ./datasource.yml:/etc/grafana/provisioning/datasources/datasource.yaml
    environment:
      - GF_AUTH_ANONYMOUS_ENABLED=true
      - GF_AUTH_ANONYMOUS_ORG_ROLE=Admin
      - GF_AUTH_DISABLE_LOGIN_FORM=true
    ports:
      - "3030:3000"

  # local exporter
  prometheus-node-exporter:
    networks:
      - backend
    image: prom/node-exporter
    ports:
      - "9100:9100"
  prometheus-nginx-exporter:
    networks:
      - backend
    image: nginx/nginx-prometheus-exporter
    ports:
      - "9113:9113"
    environment:
      - SCRAPE_URI=http://172.17.0.1:8088/metrics

        #  pushgateway:
        #    networks:
        #      - backend
        #    image: prom/pushgateway:latest
        #    ports:
        #      - "9091:9091"
networks:
  backend:

```



---
## 🟢 로그는 서드파티를 써야한다

---
### 🟢 Opentelemetry 설정들
> 추후 aws tracer 커스터마이징해서 쓰면 될듯

@opentelemetry/sdk-trace-base

데이터 수집은 opentelemetry로 하고 
데이터 전송/시각화는 Signoz.

ConsoleSpanExporter: 콘솔에 스팬을 출력하나
Signoz와 같은 백엔드로 데이터를 전송하려면 다른 익스포터 사용해야 함.

npm install @opentelemetry/exporter-otlp-grpc
-> Signoz에 데이터 전송


옵션
- instrumentations: 애플리케이션 코드에서 자동으로 계측(instrumentation)을 설정하기 위한 플러그인 목록을 지정하는 역할을 합니다. -> Node로 설정
- resource 옵션은 수집된 추적 데이터에 대한 메타데이터를 정의

---
# 구성요소
- Signoz는 Apache Kafka를 로그와 추적 데이터를 수집하고 처리하는 데 사용합니다. Zookeeper는 Signoz에서 Kafka 클러스터의 원활한 운영을 보장하기 위해 사용

# 설정
- 기본 보존 기간: 기본적으로 Signoz는 로그와 추적 데이터를 7일 동안 유지하며, 메트릭 데이터를 30일 동안 유지합니다.
- 샘플 애플리케이션 설치: docker-compose.yaml 파일을 실행하면 HotR.O.D라는 샘플 애플리케이션이 설치됩니다. 이 애플리케이션은 추적 데이터를 생성합니다.


---
## 🟢목표

1. trace: 모든 아웃포트가 trace 에 나왔으면 좋겠고.
https://signoz.io/docs/instrumentation/nestjs/#mysql-instrumentation
https://signoz.io/blog/opentelemetry-mysql-metrics-monitoring/#preparing-mysql-database-setup
@opentelemetry/instrumentation-mysql
@opentelemetry/instrumentation-redis

2. 기본 옵저빌리티 기능들- 
각 PC별로 CPU, RAM, Disk 사용률을 알고싶다. 


signoz/deploy/docker/clickhouse-setup/
1. otel-collector-config.yaml
2. docker-compose.yaml

```
receivers:
service:
    pipelines:
```
오픈텔레메트리 콜렉터는 리시버를 통해 데이터를 수집한다.
YAML 파일의 top-level에 있는 receivers 태그를 보고.

리시버가 설정되면 이제 데이터 플로우를 활성화.
service내 파이프라인은 다음과 같이 구성된다
- 리시버: 콜렉터의 데이터 엔트리포인트
- 프로세서: 데이터를 받고나면 필터/가공
- exporters: 프로세싱 후 데이터의 목적지를 정한다(외부 모니터링 시스템이든 스토리지든)

* 수집된 데이터를 보내 모니터링/시각화할 백엔드가 필요하다.

* Service 섹션
일련의 데이터 포인트들을 매분 수집하고 그래픽 형태로 나타낸다.


* prisma opentelemetry
- @prisma/instrumentation@latest
- npm install @opentelemetry/semantic-conventions @opentelemetry/exporter-trace-otlp-http @opentelemetry/instrumentation @opentelemetry/sdk-trace-base @opentelemetry/sdk-trace-node @opentelemetry/resources

Prisma의 OpenTelemetry 라이브러리를 사용하여 계측을 설정하는 경우, MySQL 리시버를 추가할 필요가 없습니다. 
Prisma OpenTelemetry 라이브러리는 Prisma 클라이언트를 통해 MySQL 데이터베이스와의 상호작용을 계측하고, 이 데이터를 OpenTelemetry Collector로 전송합니다.

- docker compose -f /Users/heojisu/signoz/deploy/docker/clickhouse-setup/docker-compose.yaml stop
- docker compose -f /Users/heojisu/signoz/deploy/docker/clickhouse-setup/docker-compose.yaml up -d
- docker compose -f /Users/heojisu/signoz/deploy/docker/clickhouse-setup/docker-compose.yaml restart
- docker compose -f /Users/heojisu/signoz/deploy/docker/clickhouse-setup/docker-compose.yaml restart signoz-otel-collector


docker compose -f docker/clickhouse-setup/docker-compose.yaml down -v

---
### HTTP logs 설정
결론은, 8082에서 수신하는건 별로 의미가 없음. 이미 4138에서 http 로그를 수신하고 있어서..
-> 모든 http 로그를 수신한다? 
https://signoz.io/docs/userguide/send-logs-http/

Expose Port
- docker-compose.yaml: Docker Compose 설정 파일에서 8082 포트를 노출
- OpenTelemetry Collector 설정 파일(otel-collector-config.yaml)에서 8082 포트에서 HTTP 로그를 수집하도록 설정

리시버
- protocols 섹션: OTLP 리시버가 어떤 프로토콜을 사용할지 정의
    - http: HTTP를 통해 데이터를 수신합니다. 기본적으로 0.0.0.0:4318에서 수신 대기합니다.
- 8082 포트에서 HTTP 로그를 수집하고자 한다면, OTLP 리시버의 HTTP 설정과 별도로 특정 HTTP 리시버를 설정

---
(참고)
docker-compose: 컨테이너의 포트를 호스트 시스템에 노출하는 방법을 정의
```
ports:
  - "호스트 포트:컨테이너 포트"
```
호스트 시스템의 포트 8082을 컨테이너의 포트 8082에 매핑합니다. 
이는 OpenTelemetry Collector가 이 포트에서 HTTP 로그를 수집하도록 설정되어 있는 경우, 해당 포트를 통해 로그를 수집할 수 있도록 합니다.

네트워크와 포트 매핑의 필요성
1. 기본 네트워크 설정기본적으로 Docker는 각 컨테이너를 자체 네트워크 네임스페이스에서 실행합니다. 이는 컨테이너가 호스트 시스템과는 독립된 네트워크 스택을 갖는다는 의미입니다. 따라서, 컨테이너 내부에서 실행 중인 서비스는 외부(호스트 시스템)에서 직접 접근할 수 없습니다.
2. 포트 매핑의 역할포트 매핑을 통해 호스트 시스템의 특정 포트를 컨테이너 내부의 포트에 연결할 수 있습니다. 이렇게 하면 외부에서 해당 포트로 접속할 때, 요청이 컨테이너 내부의 지정된 포트로 전달됩니다.Copy codeports:
  - "8082:8082"위 설정은 호스트의 포트 8082을 컨테이너의 포트 8082에 매핑합니다. 즉, 호스트에서 localhost:8082으로 요청을 보내면, 이 요청이 컨테이너 내부의 포트 8082으로 전달됩니다.

---
### 로컬 바이너리 파일 직접 실행
설정
1.리시버:•hostmetrics: 시스템 메트릭을 수집하는 리시버입니다.•collection_interval: 메트릭을 수집하는 간격을 지정합니다 (예: 30s).•scrapers: 수집할 메트릭의 종류를 지정합니다 (예: cpu, load, memory, disk, filesystem, network).
2.프로세서:•batch: 메트릭을 배치로 처리하여 성능을 최적화합니다.
3.익스포터:•logging: 수집된 메트릭을 로그로 출력합니다.•otlp: 수집된 메트릭을 OTLP 엔드포인트로 전송합니다.
4.서비스 파이프라인:•receivers: hostmetrics 리시버를 사용하여 메트릭을 수집합니다.•processors: batch 프로세서를 사용하여 메트릭을 배치로 처리합니다.•exporters: logging과 otlp 익스포터를 사용하여 메트릭을 출력 및 전송합니다.

설정 파일 적용
1.설정 파일 저장: 수정된 설정 파일을 otel-collector-config.yaml로 저장합니다.
2.OpenTelemetry Collector 실행: Collector를 설정 파일과 함께 실행합니다.Copy code./otelcol-contrib --config=otel-collector-config.yaml이 설정을 통해 OpenTelemetry Collector가 시스템 메트릭을 수집하고 이를 로그로 출력하거나 OTLP 엔드포인트로 전송하게 됩니다. 이를 통해 hostmetrics를 모니터링할 수 있습니다.


---
## Prisma Tracking
#### 트레이싱이란
- 트레이싱을 활성화하면
    - Prisma Client가 각 쿼리에 대한 트레이스를 만든다
    - 각 트레이스는 하나 이상의 span을 갖는데. 각 span은 걸린 시간을 보여주고. 트리구조로서 child span은 parent span에서 일어나는 실행을 나타낸다. 
- 트레이싱 결과를 콘솔로 보낼수도 있고, 다른 오픈텔레메트리 트레이싱 시스템(재거, 시그노즈? 등)에서 분석할 수도 있다.
    - 여기서는 재거로 어떻게 트레이싱 결과를 보낼수있는지 살펴보겠다.

#### Trace output
각 트레이스에서 일련의 스팬이 생기는데. 그 수와 타입은 Prisma Client 오퍼레이션에 따라 달라진다.



---


---
# Signoz

### 두 서버에 설정하기

opentelemetry-config.yaml
```
receivers:
  otlp:
    protocols:
      grpc:
      http:

exporters:
  otlp:
    endpoint: "http://<signoz-server>:4317" # SigNoz 서버의 URL
```
서버A, 서버B가 서로다른 IDC에서 Signoz에 데이터 보내려면.

Docker compose
- signoze 

---
### View Traces in Signoz
분산 트레이싱으로 텔레메트리 데이터를 얻는다. 

분산 트레이싱이란?
애플리케이션의 여러 컴포넌트에 걸쳐 요청을 모니터링.
어떤 인스턴스가 slow 또는 fail인지 알 수 있도록 각 리퀘스트는 유니크 ID를 갖는다.

분산 아키텍처에서, 서비스는 여러 VM/컨테이너에 걸쳐 돌아가고.
분산 트레이싱은 여러 컨테이너에 걸쳐 모니터링할 수 있다.

유저 요청이 시작되면 parent span이 할당되고, 그에 딸린 쿼리, API 콜 등 각 스텝이 child span으로 달린다. 

Tag/Attributes로 필터링
- HTTP headers, DB systems, and Messaging destinations 등 다양한 특성으로 분류 가능하다.


---
### Instrumentation
Instrument는 opentelemetry에서 애플리케이션의 특정 부분을 모니터링하는 기능.
Tracking, Metric(성능지표), Log 가 주요 작업.
특정 프레임워크에 대한 자동계측 - DB 쿼리, HTTP 요청, 메시지 큐 등을 자동으로 추적한다.


send traces to self-hosted signoz

노드 앱을 Opentelementry에 instrumenting하는 데는 두 가지 방법이 있다.
1. all in one
- @opentelemetry/auto-instrumentations-node
2. specific auto-instrument library


---
### 앱 실행
중요. 
Signoz 백엔드로 사용할 머신에 포트 4318를 열어둬야함을 잊지 말 것.
exporter -> url: 4318

Opentelemetry를 사용할 때, 트레이싱 설정을 앱보다 먼저 실행해야함.
그러나 pm2의 클러스터모드는 node 명령어에 인자를 전달할 수 없으므로,
tracer.ts를 먼저 로드하는 방법을 생각.


```
// main.ts
import './tracer'; // 트레이싱 설정을 가장 먼저 로드
```












---
NestJS 애플리케이션에서 OpenTelemetry를 설정하고, 환경 변수를 사용하여 자동 계측(instrumentation)을 구성하는 예제를 보여드리겠습니다. 이를 위해 다음 단계를 따릅니다:

1. **OpenTelemetry 및 관련 패키지 설치**
2. **환경 변수 설정**
3. **NestJS 애플리케이션에서 OpenTelemetry 설정**

### 1. OpenTelemetry 및 관련 패키지 설치

먼저 필요한 패키지를 설치합니다:

```sh
npm install @opentelemetry/api @opentelemetry/node @opentelemetry/tracing @opentelemetry/exporter-otlp-grpc @opentelemetry/auto-instrumentations-node
```

### 2. 환경 변수 설정

환경 변수를 `.env` 파일에 설정합니다. 예를 들어:

```env
OTEL_TRACES_EXPORTER=otlp
OTEL_EXPORTER_OTLP_PROTOCOL=http/protobuf
OTEL_EXPORTER_OTLP_COMPRESSION=gzip
OTEL_EXPORTER_OTLP_TRACES_ENDPOINT=https://your-endpoint
OTEL_EXPORTER_OTLP_HEADERS=x-api-key=your-api-key
OTEL_EXPORTER_OTLP_TRACES_HEADERS=x-api-key=your-api-key
OTEL_RESOURCE_ATTRIBUTES=service.namespace=my-namespace
OTEL_NODE_RESOURCE_DETECTORS=env,host,os,serviceinstance
OTEL_SERVICE_NAME=nestjs-app
NODE_OPTIONS=--require @opentelemetry/auto-instrumentations-node/register
```

### 3. NestJS 애플리케이션에서 OpenTelemetry 설정

NestJS 애플리케이션의 진입점 파일(`main.ts`)에서 OpenTelemetry 설정을 추가합니다.

#### `main.ts`

```typescript
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';
import { NodeTracerProvider } from '@opentelemetry/node';
import { Resource } from '@opentelemetry/resources';
import { BatchSpanProcessor } from '@opentelemetry/tracing';
import { OTLPTraceExporter } from '@opentelemetry/exporter-otlp-grpc';
import { registerInstrumentations } from '@opentelemetry/instrumentation';
import { getNodeAutoInstrumentations } from '@opentelemetry/auto-instrumentations-node';
import * as dotenv from 'dotenv';

dotenv.config();

async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  // OpenTelemetry 설정
  const otlpTraceExporter = new OTLPTraceExporter({
    url: process.env.OTEL_EXPORTER_OTLP_TRACES_ENDPOINT,
    headers: {
      'x-api-key': process.env.OTEL_EXPORTER_OTLP_HEADERS,
    },
  });

  const provider = new NodeTracerProvider({
    resource: new Resource({
      'service.name': process.env.OTEL_SERVICE_NAME,
      'service.namespace': process.env.OTEL_RESOURCE_ATTRIBUTES,
    }),
  });

  provider.addSpanProcessor(new BatchSpanProcessor(otlpTraceExporter));
  provider.register();

  registerInstrumentations({
    tracerProvider: provider,
    instrumentations: [getNodeAutoInstrumentations()],
  });

  await app.listen(3000);
}

bootstrap();
```

### 요약

1. **패키지 설치**: OpenTelemetry와 관련 패키지를 설치합니다.
2. **환경 변수 설정**: `.env` 파일에 환경 변수를 설정합니다.
3. **NestJS 설정**: `main.ts` 파일에서 OpenTelemetry를 설정하고, 환경 변수를 사용하여 구성합니다.

이 설정을 통해 NestJS 애플리케이션에서 OpenTelemetry를 사용하여 자동 계측을 설정할 수 있습니다. 환경 변수를 사용하여 설정을 유연하게 변경할 수 있습니다.


---
#### 리버스 프록시

1. **각 서비스에 트레이싱 라이브러리 설치**: 클라이언트, 리버스 프록시, 그리고 백엔드 서비스(prod-v1, prod-v2)에 트레이싱 라이브러리를 설치합니다.

2. **트레이싱 헤더 전달**: 리버스 프록시가 클라이언트로부터 받은 트레이싱 헤더를 백엔드 서비스로 전달하도록 설정합니다. 이를 통해 전체 요청 체인이 하나의 트레이스로 연결됩니다.

3. **스팬 생성 및 보고**: 각 서비스는 요청을 받을 때마다 새로운 스팬을 생성하고, 이를 트레이싱 시스템에 보고합니다. 리버스 프록시는 클라이언트로부터 받은 요청에 대해 스팬을 생성하고, 백엔드 서비스로 전달할 때도 스팬을 생성합니다.

이렇게 설정하면 Signoz와 같은 분산 트레이싱 도구는 클라이언트에서 시작하여 리버스 프록시를 거쳐 백엔드 서비스까지의 전체 요청 흐름을 추적하고 시각화할 수 있습니다. 이를 통해 각 서비스 간의 지연 시간, 오류 등을 파악할 수 있습니다.