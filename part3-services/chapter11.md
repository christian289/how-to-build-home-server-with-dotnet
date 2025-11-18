# 제11장: 홈 자동화와 IoT

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
