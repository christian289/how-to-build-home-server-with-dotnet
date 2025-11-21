# 부록 G: 추가로 고려할 고급 서비스

이 책에서는 홈서버와 소규모 조직에 필수적인 서비스들을 다뤘습니다. 이 부록에서는 더 복잡한 요구사항이나 대규모 환경에서 유용한 고급 서비스들을 소개합니다.

## G.1 서비스 디스커버리 및 설정 관리

### Consul

[HashiCorp Consul](https://www.consul.io/)은 서비스 메시, 서비스 디스커버리, 분산 설정 관리를 제공합니다.

**주요 기능**:
- 서비스 디스커버리 (DNS/HTTP 인터페이스)
- 헬스 체크 및 장애 감지
- Key/Value 스토어 (분산 설정)
- 멀티 데이터센터 지원
- Service Mesh (Consul Connect)

**Docker 설치**:
```yaml
version: '3.8'

services:
  consul:
    image: consul:latest
    container_name: consul
    restart: unless-stopped
    ports:
      - "8500:8500"  # UI
      - "8600:8600/udp"  # DNS
    command: agent -server -ui -bootstrap-expect=1 -client=0.0.0.0
    volumes:
      - consul-data:/consul/data

volumes:
  consul-data:
```

**사용 사례**:
- 마이크로서비스 아키텍처에서 서비스 간 통신
- 동적 설정 관리 (애플리케이션 재시작 없이 설정 변경)
- 여러 서버 간 설정 동기화

**공식 사이트**: [https://www.consul.io/](https://www.consul.io/)

---

## G.2 시크릿 관리

### Vault

[HashiCorp Vault](https://www.vaultproject.io/)는 API 키, 비밀번호, 인증서 등 민감한 정보를 안전하게 관리합니다.

**주요 기능**:
- 시크릿 암호화 저장
- 동적 시크릿 생성 (임시 DB 자격증명)
- 암호화 키 관리
- 토큰 및 리스 관리
- 감사 로그

**Docker 설치**:
```yaml
version: '3.8'

services:
  vault:
    image: vault:latest
    container_name: vault
    restart: unless-stopped
    ports:
      - "8200:8200"
    environment:
      - VAULT_DEV_ROOT_TOKEN_ID=myroot
      - VAULT_DEV_LISTEN_ADDRESS=0.0.0.0:8200
    cap_add:
      - IPC_LOCK
    volumes:
      - vault-data:/vault/data

volumes:
  vault-data:
```

**사용 사례**:
- 데이터베이스 자격증명 중앙 관리
- API 키 동적 생성 및 회전
- PKI 인증서 관리
- .NET 애플리케이션에서 `VaultSharp` 라이브러리로 통합

**공식 사이트**: [https://www.vaultproject.io/](https://www.vaultproject.io/)

---

## G.3 현대적인 리버스 프록시

### Traefik

[Traefik](https://traefik.io/)은 Docker, Kubernetes와 네이티브 통합되는 현대적인 리버스 프록시입니다.

**주요 기능**:
- 자동 서비스 디스커버리 (Docker 라벨 기반)
- Let's Encrypt 자동 SSL
- 로드 밸런싱
- 미들웨어 (인증, 레이트 리밋)
- 웹 대시보드

**Docker Compose 예제**:
```yaml
version: '3.8'

services:
  traefik:
    image: traefik:v3.0
    container_name: traefik
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
      - "8080:8080"  # Dashboard
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - ./traefik.yml:/traefik.yml:ro
      - ./acme.json:/acme.json
    labels:
      - "traefik.enable=true"

  whoami:
    image: traefik/whoami
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.whoami.rule=Host(`whoami.example.com`)"
      - "traefik.http.routers.whoami.entrypoints=websecure"
      - "traefik.http.routers.whoami.tls.certresolver=myresolver"
```

**사용 사례**:
- 마이크로서비스 환경에서 동적 라우팅
- 자동 SSL 인증서 관리
- 제13장의 Nginx 대안

**공식 사이트**: [https://traefik.io/](https://traefik.io/)

---

## G.4 메시지 큐

### RabbitMQ

[RabbitMQ](https://www.rabbitmq.com/)는 AMQP 프로토콜 기반 메시지 브로커입니다.

**주요 기능**:
- 비동기 메시지 처리
- 작업 큐 (Task Queue)
- Pub/Sub 패턴
- 높은 가용성 (클러스터링)
- 관리 UI

**Docker 설치**:
```yaml
version: '3.8'

services:
  rabbitmq:
    image: rabbitmq:3-management
    container_name: rabbitmq
    restart: unless-stopped
    ports:
      - "5672:5672"   # AMQP
      - "15672:15672" # Management UI
    environment:
      - RABBITMQ_DEFAULT_USER=admin
      - RABBITMQ_DEFAULT_PASS=admin
    volumes:
      - rabbitmq-data:/var/lib/rabbitmq

volumes:
  rabbitmq-data:
```

**.NET 통합**:
```bash
dotnet add package RabbitMQ.Client
```

**사용 사례**:
- 비동기 작업 처리 (이메일 발송, 이미지 처리)
- 마이크로서비스 간 통신
- 이벤트 기반 아키텍처

**공식 사이트**: [https://www.rabbitmq.com/](https://www.rabbitmq.com/)

---

### Apache Kafka

[Apache Kafka](https://kafka.apache.org/)는 대용량 실시간 데이터 스트리밍 플랫폼입니다.

**주요 기능**:
- 고처리량 (millions of messages/sec)
- 내구성 있는 메시지 저장
- 실시간 스트림 처리
- 분산 아키텍처

**Docker 설치 (Redpanda - Kafka 호환)**:
```yaml
version: '3.8'

services:
  redpanda:
    image: docker.redpanda.com/redpandadata/redpanda:latest
    container_name: redpanda
    restart: unless-stopped
    ports:
      - "9092:9092"   # Kafka API
      - "8082:8082"   # HTTP Proxy
    command:
      - redpanda start
      - --smp 1
      - --memory 1G
      - --reserve-memory 0M
      - --kafka-addr internal://0.0.0.0:9092,external://0.0.0.0:19092
    volumes:
      - redpanda-data:/var/lib/redpanda/data

volumes:
  redpanda-data:
```

**사용 사례**:
- 로그 수집 및 분석
- 실시간 데이터 파이프라인
- 이벤트 소싱 아키텍처

**공식 사이트**: [https://kafka.apache.org/](https://kafka.apache.org/)

---

## G.5 오브젝트 스토리지

### MinIO

[MinIO](https://min.io/)는 S3 호환 고성능 오브젝트 스토리지입니다.

**주요 기능**:
- AWS S3 API 완벽 호환
- 높은 성능 (읽기/쓰기)
- 버전 관리
- 암호화
- 이레이저 코딩 (Erasure Coding)

**Docker 설치**:
```yaml
version: '3.8'

services:
  minio:
    image: minio/minio:latest
    container_name: minio
    restart: unless-stopped
    ports:
      - "9000:9000"   # API
      - "9001:9001"   # Console
    environment:
      - MINIO_ROOT_USER=minioadmin
      - MINIO_ROOT_PASSWORD=minioadmin
    command: server /data --console-address ":9001"
    volumes:
      - minio-data:/data

volumes:
  minio-data:
```

**.NET 통합**:
```bash
dotnet add package Minio
```

```csharp
var minio = new MinioClient()
    .WithEndpoint("homeserver:9000")
    .WithCredentials("minioadmin", "minioadmin")
    .Build();

await minio.PutObjectAsync(new PutObjectArgs()
    .WithBucket("mybucket")
    .WithObject("myfile.pdf")
    .WithFileName("./myfile.pdf"));
```

**사용 사례**:
- 파일 업로드/다운로드 서비스
- AWS S3 대신 온프레미스 스토리지
- 백업 및 아카이빙
- 미디어 파일 저장 (이미지, 동영상)

**공식 사이트**: [https://min.io/](https://min.io/)

---

## G.6 로그 분석

### Elastic Stack (ELK)

[Elastic Stack](https://www.elastic.co/)은 로그 수집, 분석, 시각화를 위한 통합 솔루션입니다.

**구성 요소**:
- **Elasticsearch**: 검색 및 분석 엔진
- **Logstash**: 로그 수집 및 파싱
- **Kibana**: 데이터 시각화
- **Filebeat**: 경량 로그 수집기

**Docker Compose 예제**:
```yaml
version: '3.8'

services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.11.0
    container_name: elasticsearch
    restart: unless-stopped
    environment:
      - discovery.type=single-node
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
      - xpack.security.enabled=false
    ports:
      - "9200:9200"
    volumes:
      - es-data:/usr/share/elasticsearch/data

  kibana:
    image: docker.elastic.co/kibana/kibana:8.11.0
    container_name: kibana
    restart: unless-stopped
    ports:
      - "5601:5601"
    environment:
      - ELASTICSEARCH_HOSTS=http://elasticsearch:9200
    depends_on:
      - elasticsearch

volumes:
  es-data:
```

**사용 사례**:
- 중앙 로그 관리
- 애플리케이션 로그 분석
- 보안 이벤트 모니터링
- 비즈니스 인텔리전스

**공식 사이트**: [https://www.elastic.co/](https://www.elastic.co/)

---

## G.7 통합 인증 (SSO)

### Keycloak

[Keycloak](https://www.keycloak.org/)은 오픈소스 SSO(Single Sign-On) 및 ID 관리 솔루션입니다.

**주요 기능**:
- OAuth 2.0, OpenID Connect, SAML 2.0 지원
- 소셜 로그인 (Google, GitHub, Facebook)
- 사용자 관리
- 2FA (Two-Factor Authentication)
- 역할 기반 접근 제어 (RBAC)

**Docker 설치**:
```yaml
version: '3.8'

services:
  keycloak:
    image: quay.io/keycloak/keycloak:latest
    container_name: keycloak
    restart: unless-stopped
    ports:
      - "8080:8080"
    environment:
      - KEYCLOAK_ADMIN=admin
      - KEYCLOAK_ADMIN_PASSWORD=admin
    command: start-dev
    volumes:
      - keycloak-data:/opt/keycloak/data

volumes:
  keycloak-data:
```

**.NET 통합**:
```bash
dotnet add package Microsoft.AspNetCore.Authentication.OpenIdConnect
```

```csharp
builder.Services.AddAuthentication(options =>
{
    options.DefaultScheme = CookieAuthenticationDefaults.AuthenticationScheme;
    options.DefaultChallengeScheme = OpenIdConnectDefaults.AuthenticationScheme;
})
.AddCookie()
.AddOpenIdConnect(options =>
{
    options.Authority = "http://homeserver:8080/realms/myrealm";
    options.ClientId = "myclient";
    options.ClientSecret = "secret";
});
```

**사용 사례**:
- 여러 애플리케이션에 통합 로그인
- 소셜 로그인 제공
- 사용자 권한 중앙 관리

**공식 사이트**: [https://www.keycloak.org/](https://www.keycloak.org/)

---

## G.8 완전한 DevOps 플랫폼

### GitLab

[GitLab](https://about.gitlab.com/)은 Git 저장소, CI/CD, 이슈 트래킹을 통합한 플랫폼입니다.

**주요 기능**:
- Git 저장소 호스팅
- 내장 CI/CD (GitLab Runner)
- 이슈 및 프로젝트 관리
- 컨테이너 레지스트리
- 위키, 코드 리뷰

**Docker 설치**:
```yaml
version: '3.8'

services:
  gitlab:
    image: gitlab/gitlab-ce:latest
    container_name: gitlab
    restart: unless-stopped
    hostname: gitlab.homelab.local
    ports:
      - "80:80"
      - "443:443"
      - "22:22"
    environment:
      GITLAB_OMNIBUS_CONFIG: |
        external_url 'http://gitlab.homelab.local'
    volumes:
      - gitlab-config:/etc/gitlab
      - gitlab-logs:/var/log/gitlab
      - gitlab-data:/var/opt/gitlab

volumes:
  gitlab-config:
  gitlab-logs:
  gitlab-data:
```

**사용 사례**:
- Gitea + Gitea Actions + Nexus를 하나로 통합
- 대규모 팀 협업
- 엔터프라이즈급 DevOps 워크플로우

**공식 사이트**: [https://about.gitlab.com/](https://about.gitlab.com/)

**참고**: GitLab은 리소스 소모가 큽니다 (최소 RAM 4GB 권장). 소규모 환경에서는 Gitea + Gitea Actions 조합이 더 적합합니다.

---

## G.9 워크플로우 오케스트레이션

### Apache Airflow

[Apache Airflow](https://airflow.apache.org/)는 복잡한 데이터 파이프라인을 관리하는 워크플로우 플랫폼입니다.

**주요 기능**:
- Python 기반 워크플로우 정의 (DAG)
- 의존성 관리
- 스케줄링
- 재시도 및 에러 핸들링
- 웹 UI 모니터링

**사용 사례**:
- ETL 파이프라인
- 정기적인 데이터 처리
- ML 모델 학습 파이프라인
- 복잡한 배치 작업

**공식 사이트**: [https://airflow.apache.org/](https://airflow.apache.org/)

**참고**: Airflow는 학습 곡선이 높고 리소스를 많이 사용합니다. 간단한 스케줄링은 N8N (제11장)이 더 적합합니다.

---

## G.10 분산 추적

### Jaeger

[Jaeger](https://www.jaegertracing.io/)는 마이크로서비스 환경에서 분산 추적(Distributed Tracing)을 제공합니다.

**주요 기능**:
- 분산 트랜잭션 모니터링
- 성능 병목 식별
- 서비스 의존성 맵
- OpenTelemetry 통합

**Docker 설치**:
```yaml
version: '3.8'

services:
  jaeger:
    image: jaegertracing/all-in-one:latest
    container_name: jaeger
    restart: unless-stopped
    ports:
      - "5775:5775/udp"
      - "6831:6831/udp"
      - "6832:6832/udp"
      - "5778:5778"
      - "16686:16686"  # UI
      - "14268:14268"
      - "14250:14250"
    environment:
      - COLLECTOR_ZIPKIN_HTTP_PORT=9411
```

**사용 사례**:
- 마이크로서비스 간 요청 추적
- 성능 최적화
- 장애 원인 분석

**공식 사이트**: [https://www.jaegertracing.io/](https://www.jaegertracing.io/)

---

## G.11 서비스별 권장 환경

| 서비스 | 최소 RAM | 권장 대상 |
|--------|---------|----------|
| Consul | 512MB | 마이크로서비스 환경 |
| Vault | 512MB | 시크릿 관리 필요 시 |
| Traefik | 256MB | 동적 라우팅 필요 시 |
| RabbitMQ | 512MB | 비동기 작업 처리 |
| Kafka/Redpanda | 1GB+ | 대용량 스트리밍 |
| MinIO | 1GB | S3 호환 스토리지 필요 |
| ELK Stack | 4GB+ | 중앙 로그 관리 |
| Keycloak | 1GB | SSO 필요 시 |
| GitLab | 4GB+ | 대규모 팀 협업 |
| Airflow | 2GB+ | 복잡한 데이터 파이프라인 |
| Jaeger | 512MB | 마이크로서비스 추적 |

---

## G.12 선택 가이드

### 소규모 조직 (1-20명)에 권장

이미 책에서 다룬 서비스들로 충분합니다:
- Gitea + Gitea Actions (Git + CI/CD)
- Nexus Repository (패키지 관리)
- Sentry (에러 추적)
- PostgreSQL + Garnet (데이터베이스 + 캐시)

### 추가로 고려할 만한 서비스

**시크릿 관리가 필요하다면**: Vault
**S3 호환 스토리지가 필요하다면**: MinIO
**통합 로그인(SSO)이 필요하다면**: Keycloak
**메시지 큐가 필요하다면**: RabbitMQ

### 대규모 환경 (50명+)

- **GitLab** (통합 DevOps 플랫폼)
- **ELK Stack** (중앙 로그 관리)
- **Consul** (서비스 디스커버리)
- **Kafka** (대용량 데이터 스트리밍)
- **Jaeger** (분산 추적)

---

## 마무리

이 부록에서 소개한 서비스들은 복잡한 요구사항이나 대규모 환경에 적합합니다. 소규모 환경에서는 **과도한 엔지니어링(Over-engineering)**을 피하고, 실제 필요에 따라 점진적으로 도입하는 것이 좋습니다.

**권장 학습 순서**:
1. 책의 기본 서비스들을 먼저 익히세요
2. 실제 문제가 발생했을 때 이 부록의 서비스를 고려하세요
3. 각 서비스의 공식 문서를 참고하여 심화 학습하세요
