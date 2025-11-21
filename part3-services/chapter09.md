# 제9장: 다운로드 자동화 시스템

## 왜 자동 다운로드 시스템이 필요한가?

### 정당한 콘텐츠 관리의 효율화

**⚠️ 중요: 저작권 준수**
이 장의 모든 도구는 **합법적으로 소유하거나 배포가 허용된 콘텐츠**를 관리하기 위한 목적입니다:
- 오픈소스 소프트웨어 배포판 (Linux ISO, Docker 이미지 등)
- 크리에이티브 커먼즈(CC) 라이선스 콘텐츠
- 퍼블릭 도메인 자료
- 자체 제작 콘텐츠 (회사 교육 자료, 홍보 영상 등)
- 구독 중인 합법 콘텐츠의 개인 백업

**회사 활용 사례 - 대용량 파일 자동 다운로드**:

```
상황 1: 오픈소스 소프트웨어 배포
- Ubuntu ISO (3GB), CentOS ISO (4GB), Windows 11 ISO (5GB)
- Docker Hub에서 대용량 이미지 다운로드 (10GB+)
- 한 번에 여러 파일을 자동으로 다운로드하고 정리

상황 2: 교육 콘텐츠 아카이빙
- YouTube에 공개된 개발 강의 (MIT OpenCourseWare, Stanford 강의 등)
- 나중에 인터넷 없이도 볼 수 있도록 보관
- 채널이 삭제되거나 영상이 내려가도 안전하게 보관

상황 3: 회사 자체 제작 콘텐츠 배포
- 대용량 홍보 영상, 제품 데모 영상
- 여러 지사/협력사에 동시 배포
- BitTorrent P2P로 대역폭 절약
```

💼 **소규모 조직 관점 - 대역폭과 시간 절약**:
- **자동화**: 수동 다운로드 대신 자동으로 새 버전 확인 및 다운로드
- **대역폭 절감**: P2P 방식으로 여러 지사 간 파일 공유 시 중앙 서버 부담 감소
- **백업**: 중요한 온라인 콘텐츠를 로컬에 보관

---

## 9.1 qBittorrent 설치와 설정

[qBittorrent](https://www.qbittorrent.org/)는 웹 UI를 제공하는 BitTorrent 클라이언트입니다.

```yaml
version: '3.8'

services:
  qbittorrent:
    image: lscr.io/linuxserver/qbittorrent:latest
    container_name: qbittorrent
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Asia/Seoul
      - WEBUI_PORT=8081
    volumes:
      - ./config:/config
      - /mnt/data/downloads:/downloads
    ports:
      - 8081:8081
      - 6881:6881
      - 6881:6881/udp
    restart: unless-stopped
```

웹 UI: `http://homeserver:8081`
기본 로그인: `admin` / `adminadmin`

## 9.2 Sonarr/Radarr/Prowlarr 연동

자동 TV 프로그램/영화 다운로드 시스템 구축.

### Prowlarr (인덱서 관리)

```yaml
services:
  prowlarr:
    image: lscr.io/linuxserver/prowlarr:latest
    container_name: prowlarr
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Asia/Seoul
    volumes:
      - ./prowlarr:/config
    ports:
      - 9696:9696
    restart: unless-stopped
```

### Sonarr (TV 프로그램)

```yaml
services:
  sonarr:
    image: lscr.io/linuxserver/sonarr:latest
    container_name: sonarr
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Asia/Seoul
    volumes:
      - ./sonarr:/config
      - /mnt/data/media/tv:/tv
      - /mnt/data/downloads:/downloads
    ports:
      - 8989:8989
    restart: unless-stopped
```

### Radarr (영화)

```yaml
services:
  radarr:
    image: lscr.io/linuxserver/radarr:latest
    container_name: radarr
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Asia/Seoul
    volumes:
      - ./radarr:/config
      - /mnt/data/media/movies:/movies
      - /mnt/data/downloads:/downloads
    ports:
      - 7878:7878
    restart: unless-stopped
```

### 연동 설정

1. Prowlarr에서 인덱서 추가
2. Sonarr/Radarr에 Prowlarr 연동
3. qBittorrent를 다운로드 클라이언트로 추가
4. 원하는 TV 프로그램/영화 추가 → 자동 다운로드

## 9.3 YouTube 다운로드 자동화 (Tube Archivist)

[Tube Archivist](https://www.tubearchivist.com/)로 YouTube 채널 자동 아카이빙:

```yaml
version: '3.8'

services:
  tubearchivist:
    image: bbilly1/tubearchivist:latest
    container_name: tubearchivist
    restart: unless-stopped
    ports:
      - 8000:8000
    volumes:
      - /mnt/data/youtube/media:/youtube
      - /mnt/data/youtube/cache:/cache
    environment:
      - ES_URL=http://elasticsearch:9200
      - REDIS_HOST=redis
      - HOST_UID=1000
      - HOST_GID=1000
      - TA_HOST=tubearchivist.local
      - TA_USERNAME=admin
      - TA_PASSWORD=your_password
      - ELASTIC_PASSWORD=your_elastic_password
    depends_on:
      - elasticsearch
      - redis

  redis:
    image: redis/redis-stack-server:latest
    container_name: tubearchivist-redis
    restart: unless-stopped
    volumes:
      - redis-data:/data
    depends_on:
      - elasticsearch

  elasticsearch:
    image: bbilly1/tubearchivist-es:latest
    container_name: tubearchivist-es
    restart: unless-stopped
    environment:
      - "ELASTIC_PASSWORD=your_elastic_password"
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
      - "xpack.security.enabled=true"
      - "discovery.type=single-node"
      - "path.repo=/usr/share/elasticsearch/data/snapshot"
    volumes:
      - es-data:/usr/share/elasticsearch/data

volumes:
  redis-data:
  es-data:
```

## 9.4 VPN 연결 설정 (Gluetun)

[Gluetun](https://github.com/qdm12/gluetun)으로 다운로드 트래픽을 VPN으로 라우팅:

```yaml
services:
  gluetun:
    image: qmcgaw/gluetun:latest
    container_name: gluetun
    cap_add:
      - NET_ADMIN
    devices:
      - /dev/net/tun:/dev/net/tun
    ports:
      - 8081:8081  # qBittorrent Web UI
      - 6881:6881
      - 6881:6881/udp
    volumes:
      - ./gluetun:/gluetun
    environment:
      - VPN_SERVICE_PROVIDER=nordvpn
      - VPN_TYPE=openvpn
      - OPENVPN_USER=your_username
      - OPENVPN_PASSWORD=your_password
      - SERVER_COUNTRIES=South Korea
    restart: unless-stopped

  qbittorrent:
    image: lscr.io/linuxserver/qbittorrent:latest
    container_name: qbittorrent
    network_mode: "service:gluetun"
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Asia/Seoul
      - WEBUI_PORT=8081
    volumes:
      - ./qbittorrent:/config
      - /mnt/data/downloads:/downloads
    depends_on:
      - gluetun
    restart: unless-stopped
```

이제 qBittorrent의 모든 트래픽이 VPN을 통해 전송됩니다.

---

**다음 장에서는**: 개인 위키, 비밀번호 관리자 등 생산성 도구를 구축해보겠습니다.
