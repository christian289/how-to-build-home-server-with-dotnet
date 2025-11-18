# 제21장: 전문 서비스 구축

## 21.1 이메일 서버 (Mailcow)

[Mailcow](https://mailcow.email/)는 완전한 메일 서버 솔루션입니다.

```bash
# Mailcow 설치
cd /opt
git clone https://github.com/mailcow/mailcow-dockerized
cd mailcow-dockerized
./generate_config.sh

# docker-compose 실행
docker compose up -d
```

웹 UI: `https://mail.example.com`

**주의**: 이메일 서버는 복잡하고 관리가 어렵습니다. Gmail 사용 권장.

## 21.2 웹 분석 (Matomo/Plausible)

### Plausible Analytics

[Plausible](https://plausible.io/)은 프라이버시 친화적 웹 분석 도구입니다.

```yaml
version: '3.8'

services:
  plausible-db:
    image: postgres:14-alpine
    volumes:
      - db-data:/var/lib/postgresql/data
    environment:
      - POSTGRES_PASSWORD=postgres

  plausible-events-db:
    image: clickhouse/clickhouse-server:latest
    volumes:
      - event-data:/var/lib/clickhouse

  plausible:
    image: plausible/analytics:latest
    command: sh -c "sleep 10 && /entrypoint.sh db createdb && /entrypoint.sh db migrate && /entrypoint.sh run"
    depends_on:
      - plausible-db
      - plausible-events-db
    ports:
      - 8000:8000
    environment:
      - BASE_URL=http://analytics.homelab.local
      - SECRET_KEY_BASE=changeme
      - DATABASE_URL=postgres://postgres:postgres@plausible-db:5432/plausible_db

volumes:
  db-data:
  event-data:
```

## 21.3 온라인 오피스 (OnlyOffice)

제7장에서 Nextcloud 연동으로 다뤘습니다.

단독 설치:

```yaml
version: '3.8'

services:
  onlyoffice:
    image: onlyoffice/documentserver:latest
    container_name: onlyoffice
    ports:
      - 8889:80
    environment:
      - JWT_ENABLED=true
      - JWT_SECRET=your_secret
    volumes:
      - ./onlyoffice/logs:/var/log/onlyoffice
      - ./onlyoffice/data:/var/www/onlyoffice/Data
    restart: unless-stopped
```

## 21.4 화상회의 서버 (Jitsi Meet)

[Jitsi Meet](https://jitsi.org/)는 Zoom 대체 화상회의 서버입니다.

```bash
# Jitsi 설치
cd /opt
git clone https://github.com/jitsi/docker-jitsi-meet
cd docker-jitsi-meet
cp env.example .env

# 설정
./gen-passwords.sh
nano .env  # PUBLIC_URL 설정

# 실행
docker compose up -d
```

웹 UI: `https://meet.example.com`

---

**축하합니다!** 홈서버 구축 가이드의 모든 장을 완료했습니다. 이제 부록과 맺음말을 참고하여 더 깊이 탐구해보세요!
