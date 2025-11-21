# 제6장: 모니터링 시스템 구축

## 왜 모니터링이 필요한가?

### 서비스가 늘어나면서 겪는 장애 관리의 악몽

5장에서 리버스 프록시로 접속 문제를 해결했습니다. 이제 새로운 문제: **서비스가 언제 죽었는지 어떻게 알 것인가?**

#### 1단계: 모니터링 없이 운영 (서비스 1-3개)

처음에는 직접 확인으로 충분합니다:

```
아침 출근:
1. 브라우저 열기
2. Portainer 접속 확인
3. Gitea 접속 확인
4. PostgreSQL 상태 확인

문제 발생 시:
- 직접 눈으로 확인하고 재시작
- 로그는... 직접 SSH 접속해서 cat으로 확인
```

**이 단계에서는 문제없습니다**:
- 서비스가 3개밖에 없어 관리 가능
- 개인 프로젝트라 다운타임이 크게 문제되지 않음
- 문제 생기면 그때 해결하면 됨

#### 2단계: 문제 발견 지연 (서비스 5-10개)

**실제로 겪는 문제**:

**상황 1: 오전 11시, 팀원의 Slack 메시지**
```
팀원A: "Nexus 접속이 안되는데요?"
당신:  (황급히 서버 접속)
      "어... 메모리 부족으로 OOM Killed 됐네요"
      "언제부터 죽어있었지?"

$ docker logs nexus --tail 50
...
2025-01-20 02:37:42 java.lang.OutOfMemoryError: Java heap space
...

당신: "새벽 2시 37분부터 죽어있었네요... 9시간 동안..."
팀원: "그동안 패키지 다운로드 안됐는데 왜 모르셨어요?"
당신: "...죄송합니다"
```

**상황 2: 디스크 풀로 전체 서비스 다운**
```
금요일 저녁 6시:
팀원B: "모든 서비스가 접속이 안돼요!"
당신:  (주말 약속을 취소하고 노트북 켬)

SSH 접속:
$ df -h
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda1       100G  100G     0 100% /

당신: "디스크가 꽉 찼네... 도커 로그가 50GB를 차지하고 있어"

$ docker system df
Images          15GB
Containers      5GB
Local Volumes   10GB
Build Cache     20GB

당신: (하나씩 정리하며 2시간 소요)
     "이제 복구됐습니다"

대표: "2시간 동안 모든 서비스가 멈춰있었다는 건가?
      협력사에서 Git 접속 안된다고 연락 왔는데?"
당신: "..."
```

**상황 3: 메모리 누수를 모르고 지나감**
```
월요일:
팀원C: "서버가 왜 이렇게 느려요?"

$ free -h
              total        used        free
Mem:           16Gi        15Gi       500Mi
Swap:           8Gi         8Gi          0

당신: "메모리가 거의 꽉 찼네요. 재부팅해야겠어요"
팀원: "재부팅하면 모든 서비스 다운되는 거 아니에요?"
당신: "다른 방법이 없어요..."

재부팅 후 30분:
- 모든 서비스 재시작
- 팀원들 작업 중단
- 협력사에 공지 발송

화요일:
팀원: "또 느려졌어요"
당신: "...어제 재부팅했는데?"

$ docker stats
CONTAINER       MEM USAGE
gitea           2GB (어제는 500MB였는데)
harbor          4GB (어제는 2GB였는데)
```

💼 **소규모 조직 관점 - 신뢰도 하락의 시작**:
- **다운타임 비용**: 2시간 서비스 중단 × 20명 × $40/시간 = $1,600 손실
- **협력사 신뢰 하락**: "너네 서버 자주 죽더라" → 계약 재논의
- **개발자 스트레스**: 주말/야간에도 장애 대응
- **근본 원인 미파악**: 재부팅으로만 해결 → 문제 반복

#### 3단계: 장애 대응의 악몽 (서비스 15개+)

**상황 1: 어떤 서비스가 문제인지 모름**
```
오전 10시, 대표실:
대표: "고객이 우리 서비스 느리다는데 확인 좀"
당신: (서비스 15개를 하나씩 체크)

1. Portainer 접속... 정상
2. Gitea 접속... 정상
3. Harbor 접속... 느림! (하지만 왜?)
4. Nexus 접속... 정상
5. Grafana 접속... 정상
...

Harbor 로그 확인:
$ docker logs harbor --tail 1000 | grep error
(수백 줄의 로그가 쏟아짐)
당신: "이 중에 뭐가 문제인지..."

30분 후:
팀원: "PostgreSQL CPU가 100%인 것 같아요"
당신: "어떻게 알았어요?"
팀원: "top 명령어로 확인했어요"
당신: "..."
```

**상황 2: 장애 발생 시점을 알 수 없음**
```
고객사: "어제 오후 3시부터 API가 느렸는데 뭐 했나요?"
당신:  "...어제 오후에 뭔가 했나?"

로그 확인:
$ grep "2025-01-19 15:" /var/log/* (수천 줄의 로그)

당신: "로그가 너무 많아서..."
고객사: "그럼 원인 분석이 안된다는 건가요?"
당신: "..."
```

**상황 3: 선제적 대응 불가**
```
화요일 오전:
(아무 문제 없어 보이는 평화로운 하루)

화요일 오후 3시:
전체 서비스 다운!

원인 확인:
- 디스크 사용량이 월요일부터 계속 증가
- 월요일: 70% (경고 없음)
- 화요일 오전: 85% (경고 없음)
- 화요일 오후 2시: 95% (경고 없음)
- 화요일 오후 3시: 100% (서비스 다운)

당신: "그래프로 보니까 월요일부터 디스크가 계속 찼네요.
      어제 정리했으면 오늘 장애 없었을 텐데..."
대표: "왜 어제는 몰랐어요?"
당신: "모니터링이 없어서..."
```

💼 **소규모 조직 관점 - 비즈니스 리스크 발생**:
- **고객 신뢰 상실**: 장애 원인도 설명 못하는 회사
- **SLA 위반**: 고객사와의 가동률 약속(99%) 미달
- **기회비용**: 장애 대응에 주 10시간 소요 → 개발 시간 손실
- **개발자 번아웃**: 24시간 긴장 상태 유지 필요
- **확장 불가**: 이런 상태로는 서비스 추가 불가능

#### 4단계: Grafana + Prometheus - 문제를 사전에 차단

**도입 후 변화**:

| 항목 | 도입 전 | 도입 후 | 개선 효과 |
|------|---------|---------|-----------|
| 장애 발견 시간 | 팀원 제보 후 (평균 2-4시간) | 즉시 알림 (30초 이내) | **99% 단축** |
| 다운타임 | 연간 48시간 | 연간 2시간 | **96% 감소** |
| 장애 원인 파악 | 30분-2시간 (추측) | 5분 (그래프로 확인) | **90% 단축** |
| 디스크 풀 장애 | 월 1-2회 | 0회 (80%에서 경고) | **100% 예방** |
| 메모리 누수 발견 | 불가능 (재부팅으로 해결) | 즉시 발견 (트렌드 분석) | **근본 해결** |
| 야간/주말 장애 대응 | 다음날 출근 후 발견 | 즉시 알림 받고 원격 대응 | **비즈니스 연속성** |

**비용/시간 계산**:

```
💰 다운타임 비용 절감:
   도입 전 연간 다운타임: 48시간
   20명 × 48시간 × $40/시간 = $38,400 손실

   도입 후 연간 다운타임: 2시간
   20명 × 2시간 × $40/시간 = $1,600 손실

   절감: $36,800/년

💰 장애 대응 시간 절감:
   도입 전: 주 10시간 × 52주 = 520시간/년
   도입 후: 주 1시간 × 52주 = 52시간/년
   절감: 468시간/년 × $40/시간 = $18,720/년

💰 총 절감: $55,520/년
```

**실제 사용 예시**:

```
도입 전 - 문제가 터진 후 대응:
1. 팀원: "서비스 안돼요" (2시간 경과)
2. 원인 파악: SSH 접속 → 로그 확인 → 추측 (30분)
3. 해결: 재부팅 또는 Docker 재시작 (10분)
4. 총 피해: 2.5시간 다운타임

도입 후 - 문제 발생 전 대응:
1. Slack 알림: "디스크 사용률 80% 도달" (예측: 2시간 후 100%)
2. 대시보드 확인: Docker 로그가 50GB (원인 즉시 파악)
3. 해결: docker system prune -f (1분)
4. 총 피해: 0시간 다운타임
```

**실제 대시보드 활용**:

```
Grafana 대시보드 화면:

┌─────────────────────────────────────────────┐
│ CPU 사용률 (실시간)                           │
│ ▁▂▃▅▆█ 현재: 45%                            │
│ 경고: 80% / 위험: 90%                        │
└─────────────────────────────────────────────┘

┌─────────────────────────────────────────────┐
│ 메모리 사용률 (24시간 트렌드)                  │
│ Gitea: 500MB → 520MB → 540MB (증가 추세!)   │
│ 👉 메모리 누수 의심, 재시작 권장              │
└─────────────────────────────────────────────┘

┌─────────────────────────────────────────────┐
│ 디스크 사용률 (7일 트렌드)                     │
│ 월: 70% → 화: 73% → 수: 76%                 │
│ 👉 예측: 5일 후 90% 도달, 정리 필요           │
└─────────────────────────────────────────────┘

┌─────────────────────────────────────────────┐
│ 서비스 응답 시간                              │
│ Harbor: 평소 200ms → 지금 2000ms (10배!)    │
│ 👉 PostgreSQL CPU 100% → Harbor 느려짐      │
└─────────────────────────────────────────────┘
```

**알림 예시 (Discord/Slack)**:

```
🟢 정상 작동:
"✅ 모든 서비스 정상 (CPU: 35%, RAM: 45%, Disk: 60%)"

🟡 경고 (사전 대응 가능):
"⚠️ 디스크 사용률 80% 도달
   - 현재: 80GB/100GB
   - 예상: 2일 후 100% 도달
   - 권장 조치: Docker 로그 정리"

🔴 위험 (즉시 대응 필요):
"🚨 PostgreSQL CPU 100% 지속 (5분)
   - Harbor 응답 속도 10배 저하
   - 권장 조치: 느린 쿼리 확인 또는 재시작"
```

💡 **핵심 가치**:
- **사전 예방**: 문제가 터지기 전에 미리 대응 (디스크 80%에서 알림)
- **즉시 대응**: 장애 발생 30초 이내 알림 받고 원격 대응
- **원인 분석**: 추측이 아닌 데이터 기반 의사결정 (그래프로 명확히 확인)
- **비용 절감**: 연간 $55,000 이상 절감 (다운타임 + 대응시간)
- **전문성**: 고객에게 "지난주 화요일 오후 3시 15분에 CPU 스파이크가 있었고..." 설명 가능
- **안심**: 야간/주말에도 문제 생기면 즉시 알림 → 스트레스 감소

---

## 6.1 시스템 모니터링

### Netdata 설치와 설정

[Netdata](https://www.netdata.cloud/)는 실시간 시스템 모니터링 도구로, 설치가 간단하고 리소스 사용량이 적습니다.

#### Docker로 설치

```yaml
version: '3.8'

services:
  netdata:
    image: netdata/netdata:latest
    container_name: netdata
    hostname: homeserver
    ports:
      - 19999:19999
    restart: unless-stopped
    cap_add:
      - SYS_PTRACE
      - SYS_ADMIN
    security_opt:
      - apparmor:unconfined
    volumes:
      - netdataconfig:/etc/netdata
      - netdatalib:/var/lib/netdata
      - netdatacache:/var/cache/netdata
      - /etc/passwd:/host/etc/passwd:ro
      - /etc/group:/host/etc/group:ro
      - /proc:/host/proc:ro
      - /sys:/host/sys:ro
      - /etc/os-release:/host/etc/os-release:ro
      - /var/run/docker.sock:/var/run/docker.sock:ro
    environment:
      - NETDATA_CLAIM_TOKEN=YOUR_CLAIM_TOKEN
      - NETDATA_CLAIM_URL=https://app.netdata.cloud
      - NETDATA_CLAIM_ROOMS=YOUR_ROOM_ID

volumes:
  netdataconfig:
  netdatalib:
  netdatacache:
```

```bash
docker compose up -d
```

접속: `http://homeserver:19999`

#### Netdata 대시보드 기능

- **실시간 CPU, RAM, 디스크 사용률**
- **네트워크 트래픽**
- **Docker 컨테이너별 리소스**
- **디스크 I/O**
- **프로세스 모니터링**
- **알림 기능**

#### 알림 설정 (Discord)

`/etc/netdata/health_alarm_notify.conf`:

```conf
SEND_DISCORD="YES"
DISCORD_WEBHOOK_URL="https://discord.com/api/webhooks/YOUR_WEBHOOK"
DEFAULT_RECIPIENT_DISCORD="alerts"
```

### Grafana + Prometheus 구성

[Prometheus](https://prometheus.io/)는 메트릭 수집 도구이고, [Grafana](https://grafana.com/)는 시각화 도구입니다.

#### Docker Compose 전체 구성

```yaml
version: '3.8'

services:
  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    restart: unless-stopped
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
      - '--storage.tsdb.retention.time=30d'
    ports:
      - 9090:9090
    volumes:
      - ./prometheus/prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus-data:/prometheus
    networks:
      - monitoring

  grafana:
    image: grafana/grafana:latest
    container_name: grafana
    restart: unless-stopped
    ports:
      - 3000:3000
    environment:
      - GF_SECURITY_ADMIN_USER=admin
      - GF_SECURITY_ADMIN_PASSWORD=your_secure_password
      - GF_INSTALL_PLUGINS=grafana-clock-panel,grafana-simple-json-datasource
    volumes:
      - grafana-data:/var/lib/grafana
      - ./grafana/provisioning:/etc/grafana/provisioning
    networks:
      - monitoring

  node-exporter:
    image: prom/node-exporter:latest
    container_name: node-exporter
    restart: unless-stopped
    command:
      - '--path.rootfs=/host'
    ports:
      - 9100:9100
    volumes:
      - /:/host:ro,rslave
    networks:
      - monitoring

  cadvisor:
    image: gcr.io/cadvisor/cadvisor:latest
    container_name: cadvisor
    restart: unless-stopped
    ports:
      - 8080:8080
    volumes:
      - /:/rootfs:ro
      - /var/run:/var/run:ro
      - /sys:/sys:ro
      - /var/lib/docker:/var/lib/docker:ro
    networks:
      - monitoring

volumes:
  prometheus-data:
  grafana-data:

networks:
  monitoring:
    driver: bridge
```

#### Prometheus 설정

`prometheus/prometheus.yml`:

```yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

scrape_configs:
  # Prometheus 자체 모니터링
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  # Node Exporter (시스템 메트릭)
  - job_name: 'node'
    static_configs:
      - targets: ['node-exporter:9100']

  # cAdvisor (Docker 컨테이너 메트릭)
  - job_name: 'cadvisor'
    static_configs:
      - targets: ['cadvisor:8080']
```

#### Grafana 대시보드 설정

1. Grafana 접속: `http://homeserver:3000`
2. 로그인: `admin` / `your_secure_password`
3. **Configuration** → **Data Sources** → **Add data source**
4. **Prometheus** 선택
5. URL: `http://prometheus:9090`
6. **Save & Test**

#### 인기 대시보드 import

1. **Dashboards** → **Import**
2. Dashboard ID 입력:
   - **1860**: Node Exporter Full
   - **193**: Docker 모니터링
   - **11074**: Node Exporter for Prometheus

#### 커스텀 대시보드 예제

간단한 CPU 사용률 패널:

```
Query: 100 - (avg by(instance) (irate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)
Legend: CPU Usage
```

메모리 사용률:

```
Query: (1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) * 100
Legend: Memory Usage
```

## 6.2 로그 관리 시스템

### Dozzle로 Docker 로그 관리

[Dozzle](https://dozzle.dev/)은 Docker 컨테이너 로그를 웹에서 실시간으로 볼 수 있는 도구입니다.

```yaml
version: '3.8'

services:
  dozzle:
    image: amir20/dozzle:latest
    container_name: dozzle
    restart: unless-stopped
    ports:
      - 9999:8080
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
    environment:
      DOZZLE_LEVEL: info
      DOZZLE_TAILSIZE: 300
      DOZZLE_FILTER: "status=running"
```

접속: `http://homeserver:9999`

기능:
- 실시간 로그 스트리밍
- 여러 컨테이너 동시 모니터링
- 로그 검색 및 필터링
- 다크 모드 지원

### Loki + Promtail + Grafana (고급)

더 강력한 로그 관리가 필요하다면 [Loki](https://grafana.com/oss/loki/)를 사용합니다.

```yaml
services:
  loki:
    image: grafana/loki:latest
    container_name: loki
    ports:
      - 3100:3100
    volumes:
      - ./loki-config.yml:/etc/loki/local-config.yaml
      - loki-data:/loki
    command: -config.file=/etc/loki/local-config.yaml
    networks:
      - monitoring

  promtail:
    image: grafana/promtail:latest
    container_name: promtail
    volumes:
      - /var/log:/var/log
      - ./promtail-config.yml:/etc/promtail/config.yml
      - /var/lib/docker/containers:/var/lib/docker/containers:ro
      - /var/run/docker.sock:/var/run/docker.sock
    command: -config.file=/etc/promtail/config.yml
    networks:
      - monitoring

volumes:
  loki-data:
```

`loki-config.yml`:

```yaml
auth_enabled: false

server:
  http_listen_port: 3100

ingester:
  lifecycler:
    ring:
      kvstore:
        store: inmemory
      replication_factor: 1
  chunk_idle_period: 5m
  chunk_retain_period: 30s

schema_config:
  configs:
    - from: 2020-05-15
      store: boltdb
      object_store: filesystem
      schema: v11
      index:
        prefix: index_
        period: 168h

storage_config:
  boltdb:
    directory: /loki/index
  filesystem:
    directory: /loki/chunks

limits_config:
  enforce_metric_name: false
  reject_old_samples: true
  reject_old_samples_max_age: 168h

chunk_store_config:
  max_look_back_period: 0s

table_manager:
  retention_deletes_enabled: true
  retention_period: 720h
```

## 6.3 알림 설정

### Discord 봇 연동

#### 1. Discord Webhook 생성

1. Discord 서버 → 채널 설정 → 연동
2. **Webhook 만들기**
3. Webhook URL 복사

#### 2. Prometheus Alertmanager 설정

`alertmanager.yml`:

```yaml
global:
  resolve_timeout: 5m

route:
  group_by: ['alertname']
  group_wait: 10s
  group_interval: 10s
  repeat_interval: 12h
  receiver: 'discord'

receivers:
- name: 'discord'
  webhook_configs:
  - url: 'http://alertmanager-discord:9094'

inhibit_rules:
  - source_match:
      severity: 'critical'
    target_match:
      severity: 'warning'
    equal: ['alertname', 'dev', 'instance']
```

#### 3. Discord Webhook 컨테이너

```yaml
services:
  alertmanager-discord:
    image: benjojo/alertmanager-discord:latest
    container_name: alertmanager-discord
    environment:
      - DISCORD_WEBHOOK=https://discord.com/api/webhooks/YOUR_WEBHOOK
    networks:
      - monitoring
```

### Telegram 봇 연동

#### 1. Bot 생성

1. Telegram에서 @BotFather 검색
2. `/newbot` 명령어 실행
3. Bot 이름과 username 설정
4. API Token 복사

#### 2. Chat ID 얻기

```bash
# Bot과 대화 시작 후
curl https://api.telegram.org/bot<YOUR_BOT_TOKEN>/getUpdates
```

#### 3. 알림 스크립트

```bash
#!/bin/bash
BOT_TOKEN="your_bot_token"
CHAT_ID="your_chat_id"
MESSAGE="🚨 서버 CPU 사용률 높음: 85%"

curl -s -X POST "https://api.telegram.org/bot${BOT_TOKEN}/sendMessage" \
  -d chat_id="${CHAT_ID}" \
  -d text="${MESSAGE}"
```

### 이메일 알림 구성

#### Gmail SMTP 사용

Grafana에서 이메일 알림 설정:

`grafana.ini`:

```ini
[smtp]
enabled = true
host = smtp.gmail.com:587
user = your_email@gmail.com
password = your_app_password
skip_verify = false
from_address = your_email@gmail.com
from_name = Grafana Homeserver

[alerting]
enabled = true
```

#### Mailhog (개발/테스트용)

실제 이메일을 보내지 않고 테스트하려면:

```yaml
services:
  mailhog:
    image: mailhog/mailhog:latest
    container_name: mailhog
    ports:
      - 1025:1025  # SMTP
      - 8025:8025  # Web UI
```

Grafana SMTP 설정:
```ini
host = mailhog:1025
```

웹 UI에서 이메일 확인: `http://homeserver:8025`

### Uptime Kuma (통합 모니터링 & 알림)

[Uptime Kuma](https://github.com/louislam/uptime-kuma)는 서비스 가동 상태를 모니터링하고 알림을 보냅니다.

```yaml
version: '3.8'

services:
  uptime-kuma:
    image: louislam/uptime-kuma:latest
    container_name: uptime-kuma
    volumes:
      - uptime-kuma-data:/app/data
    ports:
      - 3001:3001
    restart: unless-stopped

volumes:
  uptime-kuma-data:
```

접속: `http://homeserver:3001`

기능:
- HTTP(s) / TCP / Ping / DNS 모니터링
- 다양한 알림 채널 (Discord, Telegram, Slack, Email 등)
- 상태 페이지 생성
- 아름다운 대시보드

---

**다음 장에서는**: Nextcloud를 활용한 개인 클라우드 스토리지 구축 방법을 알아보겠습니다.
