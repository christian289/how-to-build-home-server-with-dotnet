# 제8장: 미디어 서버 구축

## 8.1 Jellyfin 미디어 서버

[Jellyfin](https://jellyfin.org/)은 Netflix와 같은 개인 미디어 서버입니다.

### 설치와 라이브러리 구성

```yaml
version: '3.8'

services:
  jellyfin:
    image: jellyfin/jellyfin:latest
    container_name: jellyfin
    user: 1000:1000
    network_mode: 'host'
    volumes:
      - ./config:/config
      - ./cache:/cache
      - /mnt/data/media/movies:/media/movies
      - /mnt/data/media/tv:/media/tv
      - /mnt/data/media/music:/media/music
    restart: unless-stopped
    devices:
      - /dev/dri:/dev/dri  # Intel QuickSync
```

웹 UI: `http://homeserver:8096`

### 하드웨어 가속 설정 (Intel QuickSync)

N100 프로세서의 내장 GPU를 활용한 트랜스코딩:

1. **Dashboard** → **Playback** 이동
2. **Hardware acceleration**: Intel QuickSync (QSV) 선택
3. 인코딩 옵션 체크:
   - Enable hardware decoding for: H264, HEVC, VP9
   - Enable hardware encoding
   - Enable VPP Tone mapping

### 모바일/TV 앱 설정

- iOS/Android: "Jellyfin" 앱 다운로드
- Smart TV: 앱 스토어에서 Jellyfin 검색
- 서버 주소 입력: `http://homeserver:8096`

## 8.2 음악 스트리밍 (Navidrome)

[Navidrome](https://www.navidrome.org/)는 개인 Spotify 서버입니다.

```yaml
version: '3.8'

services:
  navidrome:
    image: deluan/navidrome:latest
    container_name: navidrome
    user: 1000:1000
    ports:
      - 4533:4533
    environment:
      ND_SCANSCHEDULE: 1h
      ND_LOGLEVEL: info
      ND_SESSIONTIMEOUT: 24h
      ND_BASEURL: ""
    volumes:
      - ./data:/data
      - /mnt/data/media/music:/music:ro
    restart: unless-stopped
```

Subsonic 호환 앱으로 접속 가능:
- iOS: play:Sub
- Android: DSub, Ultrasonic

## 8.3 사진 관리 (Immich)

[Immich](https://immich.app/)은 Google Photos 대체 솔루션입니다.

```yaml
version: '3.8'

services:
  immich-server:
    container_name: immich_server
    image: ghcr.io/immich-app/immich-server:release
    command: ['start.sh', 'immich']
    volumes:
      - /mnt/data/photos:/usr/src/app/upload
      - /etc/localtime:/etc/localtime:ro
    env_file:
      - .env
    depends_on:
      - redis
      - database
    restart: unless-stopped

  immich-microservices:
    container_name: immich_microservices
    image: ghcr.io/immich-app/immich-server:release
    command: ['start.sh', 'microservices']
    volumes:
      - /mnt/data/photos:/usr/src/app/upload
      - /etc/localtime:/etc/localtime:ro
    env_file:
      - .env
    depends_on:
      - redis
      - database
    restart: unless-stopped

  immich-machine-learning:
    container_name: immich_machine_learning
    image: ghcr.io/immich-app/immich-machine-learning:release
    volumes:
      - model-cache:/cache
    env_file:
      - .env
    restart: unless-stopped

  immich-web:
    container_name: immich_web
    image: ghcr.io/immich-app/immich-web:release
    env_file:
      - .env
    restart: unless-stopped

  redis:
    container_name: immich_redis
    image: redis:6.2-alpine
    restart: unless-stopped

  database:
    container_name: immich_postgres
    image: tensorchord/pgvecto-rs:pg14-v0.2.0
    env_file:
      - .env
    environment:
      POSTGRES_PASSWORD: ${DB_PASSWORD}
      POSTGRES_USER: ${DB_USERNAME}
      POSTGRES_DB: ${DB_DATABASE_NAME}
    volumes:
      - pgdata:/var/lib/postgresql/data
    restart: unless-stopped

  immich-proxy:
    container_name: immich_proxy
    image: ghcr.io/immich-app/immich-proxy:release
    environment:
      IMMICH_SERVER_URL: http://immich-server:3001
      IMMICH_WEB_URL: http://immich-web:3000
    ports:
      - 2283:8080
    depends_on:
      - immich-server
      - immich-web
    restart: unless-stopped

volumes:
  pgdata:
  model-cache:
```

`.env`:
```env
DB_PASSWORD=postgres
DB_USERNAME=postgres
DB_DATABASE_NAME=immich
```

### AI 얼굴 인식 설정

Immich는 자동으로 얼굴을 인식하고 그룹화합니다. 추가 설정 불필요.

### 모바일 자동 백업

- iOS/Android 앱 설치
- 설정 → Background backup 활성화
- 자동으로 사진/비디오 업로드

## 8.4 전자책 관리 (Calibre-Web)

[Calibre-Web](https://github.com/janeczku/calibre-web)으로 전자책 라이브러리 관리:

```yaml
version: '3.8'

services:
  calibre-web:
    image: lscr.io/linuxserver/calibre-web:latest
    container_name: calibre-web
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Asia/Seoul
    volumes:
      - ./config:/config
      - /mnt/data/books:/books
    ports:
      - 8083:8083
    restart: unless-stopped
```

기본 로그인: `admin` / `admin123`

---

**다음 장에서는**: qBittorrent와 Sonarr/Radarr를 활용한 다운로드 자동화 시스템을 알아보겠습니다.
