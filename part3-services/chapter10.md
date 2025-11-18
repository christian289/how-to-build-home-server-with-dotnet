# 제10장: 개인 생산성 도구

## 10.1 개인 위키 (WikiJS/BookStack)

### WikiJS

[Wiki.js](https://js.wiki/)는 강력한 오픈소스 위키 시스템입니다.

```yaml
version: '3.8'

services:
  wikijs-db:
    image: postgres:15-alpine
    container_name: wikijs-db
    environment:
      POSTGRES_DB: wiki
      POSTGRES_PASSWORD: wikijsrocks
      POSTGRES_USER: wikijs
    volumes:
      - wikijs-db-data:/var/lib/postgresql/data
    restart: unless-stopped

  wikijs:
    image: requarks/wiki:2
    container_name: wikijs
    depends_on:
      - wikijs-db
    environment:
      DB_TYPE: postgres
      DB_HOST: wikijs-db
      DB_PORT: 5432
      DB_USER: wikijs
      DB_PASS: wikijsrocks
      DB_NAME: wiki
    restart: unless-stopped
    ports:
      - "3003:3000"

volumes:
  wikijs-db-data:
```

초기 설정: `http://homeserver:3003`

### BookStack

[BookStack](https://www.bookstackapp.com/)은 사용하기 쉬운 위키 플랫폼입니다.

```yaml
version: '3.8'

services:
  bookstack-db:
    image: mariadb:10
    container_name: bookstack-db
    environment:
      - MYSQL_ROOT_PASSWORD=secret
      - MYSQL_DATABASE=bookstack
      - MYSQL_USER=bookstack
      - MYSQL_PASSWORD=secret
    volumes:
      - bookstack-db-data:/var/lib/mysql
    restart: unless-stopped

  bookstack:
    image: lscr.io/linuxserver/bookstack:latest
    container_name: bookstack
    environment:
      - PUID=1000
      - PGID=1000
      - APP_URL=http://bookstack.homelab.local
      - DB_HOST=bookstack-db
      - DB_PORT=3306
      - DB_USER=bookstack
      - DB_PASS=secret
      - DB_DATABASE=bookstack
    volumes:
      - ./bookstack:/config
    ports:
      - 6875:80
    restart: unless-stopped
    depends_on:
      - bookstack-db

volumes:
  bookstack-db-data:
```

기본 로그인: `admin@admin.com` / `password`

## 10.2 노트 앱 (Joplin Server)

[Joplin](https://joplinapp.org/)은 Evernote 대체 노트 앱입니다.

```yaml
version: '3.8'

services:
  joplin-db:
    image: postgres:16-alpine
    container_name: joplin-db
    volumes:
      - joplin-db:/var/lib/postgresql/data
    ports:
      - "5432:5432"
    restart: unless-stopped
    environment:
      - POSTGRES_PASSWORD=joplin
      - POSTGRES_USER=joplin
      - POSTGRES_DB=joplin

  joplin:
    image: joplin/server:latest
    container_name: joplin
    depends_on:
      - joplin-db
    ports:
      - "22300:22300"
    restart: unless-stopped
    environment:
      - APP_PORT=22300
      - APP_BASE_URL=https://joplin.example.com
      - DB_CLIENT=pg
      - POSTGRES_PASSWORD=joplin
      - POSTGRES_DATABASE=joplin
      - POSTGRES_USER=joplin
      - POSTGRES_PORT=5432
      - POSTGRES_HOST=joplin-db

volumes:
  joplin-db:
```

데스크톱/모바일 앱에서 동기화 설정.

## 10.3 할 일 관리 (Vikunja)

[Vikunja](https://vikunja.io/)는 Todoist 대체 할 일 관리 도구입니다.

```yaml
version: '3.8'

services:
  vikunja-db:
    image: postgres:16-alpine
    container_name: vikunja-db
    environment:
      POSTGRES_PASSWORD: secret
      POSTGRES_USER: vikunja
      POSTGRES_DB: vikunja
    volumes:
      - vikunja-db:/var/lib/postgresql/data
    restart: unless-stopped

  vikunja:
    image: vikunja/vikunja:latest
    container_name: vikunja
    environment:
      VIKUNJA_DATABASE_HOST: vikunja-db
      VIKUNJA_DATABASE_PASSWORD: secret
      VIKUNJA_DATABASE_TYPE: postgres
      VIKUNJA_DATABASE_USER: vikunja
      VIKUNJA_DATABASE_DATABASE: vikunja
      VIKUNJA_SERVICE_JWTSECRET: random_secret_key
      VIKUNJA_SERVICE_FRONTENDURL: http://homeserver:3456
    ports:
      - 3456:3456
    volumes:
      - vikunja-files:/app/vikunja/files
    depends_on:
      - vikunja-db
    restart: unless-stopped

volumes:
  vikunja-db:
  vikunja-files:
```

## 10.4 비밀번호 관리자 (Vaultwarden)

[Vaultwarden](https://github.com/dani-garcia/vaultwarden)은 Bitwarden 호환 비밀번호 관리자입니다.

```yaml
version: '3.8'

services:
  vaultwarden:
    image: vaultwarden/server:latest
    container_name: vaultwarden
    restart: unless-stopped
    environment:
      - WEBSOCKET_ENABLED=true
      - SIGNUPS_ALLOWED=true
      - INVITATIONS_ALLOWED=true
      - DOMAIN=https://vault.example.com
      - SMTP_HOST=smtp.gmail.com
      - SMTP_FROM=your-email@gmail.com
      - SMTP_PORT=587
      - SMTP_SECURITY=starttls
      - SMTP_USERNAME=your-email@gmail.com
      - SMTP_PASSWORD=your-app-password
    volumes:
      - ./vaultwarden-data:/data
    ports:
      - 8087:80
      - 3012:3012
```

**보안 권장사항**:
- HTTPS 필수 (Nginx Proxy Manager로 설정)
- 2FA 활성화
- 첫 계정 생성 후 `SIGNUPS_ALLOWED=false`로 변경

## 10.5 RSS 리더 (FreshRSS)

[FreshRSS](https://freshrss.org/)로 뉴스 피드 통합 관리:

```yaml
version: '3.8'

services:
  freshrss-db:
    image: postgres:16-alpine
    container_name: freshrss-db
    environment:
      POSTGRES_USER: freshrss
      POSTGRES_PASSWORD: freshrss
      POSTGRES_DB: freshrss
    volumes:
      - freshrss-db:/var/lib/postgresql/data
    restart: unless-stopped

  freshrss:
    image: freshrss/freshrss:latest
    container_name: freshrss
    restart: unless-stopped
    ports:
      - 8084:80
    environment:
      - TZ=Asia/Seoul
      - CRON_MIN=*/15
    volumes:
      - ./freshrss/data:/var/www/FreshRSS/data
      - ./freshrss/extensions:/var/www/FreshRSS/extensions
    depends_on:
      - freshrss-db

volumes:
  freshrss-db:
```

웹 UI: `http://homeserver:8084`

모바일 앱:
- iOS: Reeder, NetNewsWire
- Android: FeedMe, News+

---

**다음 장에서는**: Home Assistant를 활용한 스마트홈 자동화 시스템을 구축해보겠습니다.
