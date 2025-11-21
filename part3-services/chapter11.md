# 제11장: 홈 자동화와 IoT

## 왜 자동화 시스템이 필요한가?

### 업무 자동화로 생산성 극대화

10장에서 생산성 도구를 구축했습니다. 이제 자동화: **반복 작업을 자동화하여 시간을 절약할 것인가?**

#### 워크플로우 자동화 SaaS 비용 (20명 조직)

```
💰 Zapier Pro:
- $20/월 기본 + 사용량 기반 요금
- 팀 평균 사용 시: $200/월 = $2,400/년

💰 Make (구 Integromat):
- $16/월 기본 + 사용량 기반 요금
- 팀 평균 사용 시: $150/월 = $1,800/년

💰 Power Automate:
- $15/user/월 × 10명 = $150/월 = $1,800/년

5년 총 비용: $9,000 - $12,000
```

**실제 활용 사례 - N8N 워크플로우 자동화**:

```
시나리오 1: GitHub → Discord 알림 자동화
- GitHub에 이슈/PR 생성 시 자동으로 Discord 알림
- Zapier 비용: $20/월 → N8N: $0/월
- 연간 절감: $240

시나리오 2: 폼 응답 → Google Sheets 자동 저장
- 고객 문의 폼 → Google Sheets에 자동 추가
- → Slack 알림
- Zapier 비용: $20/월 → N8N: $0/월
- 연간 절감: $240

시나리오 3: 주간 리포트 자동 생성
- 매주 월요일 자동으로:
  - GitHub 커밋 통계 수집
  - Jira 완료 티켓 수집
  - Slack으로 주간 리포트 전송
- Zapier 비용: $30/월 → N8N: $0/월
- 연간 절감: $360

총 절감: $840/년 (3개 워크플로우만으로)
```

💼 **소규모 조직 관점 - 자동화로 시간 절약**:
- **비용 절감**: Zapier/Make 대비 연간 $2,000+ 절약
- **무제한 실행**: 워크플로우 실행 횟수 제한 없음
- **프라이버시**: 회사 데이터가 외부 서버로 나가지 않음
- **커스터마이징**: JavaScript로 복잡한 로직 구현 가능

#### 스마트 오피스 자동화 - Home Assistant

**사무실 에너지 절약**:

```
자동화 시나리오:
1. 퇴근 시간(오후 7시) 되면 자동으로:
   - 사무실 조명 끄기
   - 에어컨 끄기
   - 모니터 전원 끄기

2. 출근 시간(오전 9시) 되면 자동으로:
   - 사무실 조명 켜기 (밝기 70%)
   - 에어컨 24도로 설정

3. 주말/공휴일:
   - 모든 전기 기기 자동 차단

예상 절감:
- 전기요금 월 10% 절감
- 월 $50 절감 = 연간 $600 절감
```

💡 **핵심 가치 (N8N 워크플로우 자동화)**:
- **비용 절감**: Zapier 대비 연간 $2,400 절약
- **무제한**: 워크플로우 실행 횟수, 복잡도 제한 없음
- **프라이버시**: 민감한 비즈니스 데이터가 외부로 나가지 않음
- **통합**: 사내 모든 시스템과 자유롭게 연동 (Gitea, Nexus, Wiki 등)
- **확장성**: JavaScript로 복잡한 비즈니스 로직 구현 가능

---

## 11.1 Home Assistant 구축

[Home Assistant](https://www.home-assistant.io/)는 모든 스마트홈 기기를 통합 제어하는 플랫폼입니다.

### 설치와 기본 설정

```yaml
version: '3.8'

services:
  homeassistant:
    container_name: homeassistant
    image: ghcr.io/home-assistant/home-assistant:stable
    volumes:
      - ./homeassistant:/config
      - /etc/localtime:/etc/localtime:ro
    restart: unless-stopped
    privileged: true
    network_mode: host
```

웹 UI: `http://homeserver:8123`

### 스마트 기기 연동

**지원 기기**:
- Philips Hue
- Xiaomi (Mi Home)
- Tuya (SmartLife)
- Samsung SmartThings
- IKEA Trådfri
- Sonoff
- TP-Link Kasa

**연동 방법**:
1. **Settings** → **Devices & Services**
2. **Add Integration** 클릭
3. 기기 제조사 검색 후 인증

### 자동화 시나리오 작성

**예제 1: 일몰 시 조명 켜기**

```yaml
automation:
  - alias: "저녁 조명 자동 켜기"
    trigger:
      - platform: sun
        event: sunset
        offset: "-00:30:00"
    action:
      - service: light.turn_on
        target:
          entity_id: light.living_room
        data:
          brightness: 200
          color_temp: 370
```

**예제 2: 집에 도착하면 환영 메시지**

```yaml
automation:
  - alias: "집 도착 환영"
    trigger:
      - platform: state
        entity_id: person.gildong
        to: 'home'
    action:
      - service: notify.mobile_app
        data:
          message: "어서오세요! 온도를 조절하고 있습니다."
      - service: climate.set_temperature
        target:
          entity_id: climate.living_room
        data:
          temperature: 23
```

## 11.2 MQTT 브로커 설정 (Mosquitto)

[MQTT](https://mqtt.org/)는 IoT 기기 간 통신 프로토콜입니다.

```yaml
version: '3.8'

services:
  mosquitto:
    image: eclipse-mosquitto:latest
    container_name: mosquitto
    restart: unless-stopped
    ports:
      - "1883:1883"
      - "9001:9001"
    volumes:
      - ./mosquitto/config:/mosquitto/config
      - ./mosquitto/data:/mosquitto/data
      - ./mosquitto/log:/mosquitto/log
```

`mosquitto/config/mosquitto.conf`:

```conf
listener 1883
listener 9001
protocol websockets

allow_anonymous false
password_file /mosquitto/config/passwd
```

비밀번호 생성:

```bash
docker exec -it mosquitto mosquitto_passwd -c /mosquitto/config/passwd admin
```

Home Assistant에서 MQTT 연동:
1. **Settings** → **Devices & Services**
2. **MQTT** 추가
3. Broker: `homeserver`, Port: `1883`

## 11.3 Node-RED로 자동화 플로우 구성

[Node-RED](https://nodered.org/)는 비주얼 프로그래밍 도구입니다.

```yaml
version: '3.8'

services:
  nodered:
    image: nodered/node-red:latest
    container_name: nodered
    restart: unless-stopped
    ports:
      - "1880:1880"
    volumes:
      - ./nodered:/data
    environment:
      - TZ=Asia/Seoul
```

웹 UI: `http://homeserver:1880`

**예제 플로우: 온도 센서 → Telegram 알림**

1. MQTT In 노드: `home/sensor/temperature`
2. Function 노드:
```javascript
if (msg.payload > 30) {
    msg.payload = "🌡️ 실내 온도가 30°C를 초과했습니다!";
    return msg;
}
```
3. Telegram 노드: 봇 토큰 설정

## 11.4 ESPHome 연동

[ESPHome](https://esphome.io/)은 ESP32/ESP8266 펌웨어 관리 도구입니다.

```yaml
version: '3.8'

services:
  esphome:
    container_name: esphome
    image: ghcr.io/esphome/esphome:latest
    volumes:
      - ./esphome/config:/config
      - /etc/localtime:/etc/localtime:ro
    restart: unless-stopped
    privileged: true
    network_mode: host
```

웹 UI: `http://homeserver:6052`

**예제 설정: DHT22 온습도 센서**

```yaml
esphome:
  name: room-sensor
  platform: ESP32
  board: esp32dev

wifi:
  ssid: "YourWiFi"
  password: "YourPassword"

api:
  encryption:
    key: "your_encryption_key"

ota:
  password: "your_ota_password"

sensor:
  - platform: dht
    pin: GPIO4
    temperature:
      name: "Room Temperature"
    humidity:
      name: "Room Humidity"
    update_interval: 60s
```

## 11.5 N8N 워크플로우 자동화

[n8n](https://n8n.io/)은 Zapier 대체 워크플로우 자동화 도구입니다.

### N8N 설치와 기본 설정

```yaml
version: '3.8'

services:
  n8n:
    image: n8nio/n8n:latest
    container_name: n8n
    restart: unless-stopped
    ports:
      - "5678:5678"
    environment:
      - N8N_HOST=n8n.homelab.local
      - N8N_PORT=5678
      - N8N_PROTOCOL=http
      - GENERIC_TIMEZONE=Asia/Seoul
      - N8N_BASIC_AUTH_ACTIVE=true
      - N8N_BASIC_AUTH_USER=admin
      - N8N_BASIC_AUTH_PASSWORD=your_password
    volumes:
      - ./n8n:/home/node/.n8n
```

웹 UI: `http://homeserver:5678`

### 웹훅과 API 연동

**예제: GitHub 이슈 → Discord 알림**

1. Webhook 트리거 노드
2. GitHub 노드: 새 이슈 감지
3. Function 노드: 메시지 포맷팅
```javascript
return {
  json: {
    content: `🐛 새 이슈: ${$json.title}\n${$json.url}`
  }
};
```
4. Discord Webhook 노드

### 실용적인 워크플로우 예제

**예제 1: 일일 날씨 리포트**

```
Cron (매일 7시) → OpenWeather API → Telegram 메시지
```

**예제 2: 웹사이트 모니터링**

```
Cron (5분마다) → HTTP Request → If (상태코드 ≠ 200) → Discord 알림
```

**예제 3: 이메일 자동 분류**

```
Gmail Trigger → If (제목 contains "청구서") → Google Sheets에 추가
```

### 외부 서비스 통합 (Google, Slack, Discord)

**Google Sheets 연동**:
1. Google Sheets 노드 추가
2. OAuth2 인증
3. 스프레드시트 ID 입력
4. 데이터 읽기/쓰기

**Slack 연동**:
1. Slack App 생성
2. Bot Token 발급
3. N8N에서 Slack 노드 설정

**Discord 연동**:
1. Discord Webhook URL 생성
2. Webhook 노드에서 URL 입력

---

**다음 장에서는**: Gitea와 CI/CD 파이프라인을 활용한 개발 환경을 구축해보겠습니다.
