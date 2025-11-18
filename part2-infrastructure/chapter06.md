# 제6장: 모니터링 시스템 구축

시스템 모니터링은 홈서버의 건강 상태를 파악하고 문제를 조기에 발견하는 데 필수적입니다.

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
