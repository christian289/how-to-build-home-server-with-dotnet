# 부록 A: 유용한 Docker 이미지 모음

## A.1 필수 유틸리티

- **Portainer**: `portainer/portainer-ce` - Docker GUI 관리
- **Watchtower**: `containrrr/watchtower` - 자동 업데이트
- **Dozzle**: `amir20/dozzle` - 로그 뷰어
- **Diun**: `crazymax/diun` - 이미지 업데이트 알림

## A.2 개발 도구

- **Gitea**: `gitea/gitea` - Git 서버
- **Gitea Act Runner**: `gitea/act_runner` - CI/CD (Gitea Actions)
- **Nexus Repository**: `sonatype/nexus3` - 패키지 저장소 (NuGet, npm, Maven, Docker)
- **Harbor**: `goharbor/harbor-core` - 엔터프라이즈 Docker 레지스트리
- **Sentry**: `sentry` - 애플리케이션 에러 추적
- **code-server**: `lscr.io/linuxserver/code-server` - 웹 VS Code
- **Adminer**: `adminer` - 데이터베이스 관리
- **pgAdmin**: `dpage/pgadmin4` - PostgreSQL 관리
- **phpMyAdmin**: `phpmyadmin` - MySQL/MariaDB 관리
- **Microsoft Garnet**: `ghcr.io/microsoft/garnet` - Redis 호환 캐시

## A.3 미디어 관련

- **Jellyfin**: `jellyfin/jellyfin` - 미디어 서버
- **Navidrome**: `deluan/navidrome` - 음악 스트리밍
- **Immich**: `ghcr.io/immich-app/immich-server` - 사진 관리
- **Calibre-Web**: `lscr.io/linuxserver/calibre-web` - 전자책
- **Tube Archivist**: `bbilly1/tubearchivist` - YouTube 아카이빙
- **qBittorrent**: `lscr.io/linuxserver/qbittorrent` - 토렌트

## A.4 생산성 도구

- **Nextcloud**: `nextcloud` - 클라우드 스토리지
- **Vaultwarden**: `vaultwarden/server` - 비밀번호 관리
- **Wiki.js**: `requarks/wiki` - 위키
- **BookStack**: `lscr.io/linuxserver/bookstack` - 위키
- **Vikunja**: `vikunja/vikunja` - 할 일 관리
- **Joplin**: `joplin/server` - 노트
- **FreshRSS**: `freshrss/freshrss` - RSS 리더
- **Syncthing**: `syncthing/syncthing` - 파일 동기화

## A.5 모니터링 도구

- **Grafana**: `grafana/grafana` - 시각화
- **Prometheus**: `prom/prometheus` - 메트릭 수집
- **Netdata**: `netdata/netdata` - 실시간 모니터링
- **Uptime Kuma**: `louislam/uptime-kuma` - 가동 상태 모니터링
- **Loki**: `grafana/loki` - 로그 수집
- **cAdvisor**: `gcr.io/cadvisor/cadvisor` - 컨테이너 모니터링

## A.6 네트워크 도구

- **Nginx Proxy Manager**: `jc21/nginx-proxy-manager` - 리버스 프록시
- **Traefik**: `traefik` - 리버스 프록시
- **Pi-hole**: `pihole/pihole` - 광고 차단 DNS
- **Cloudflared**: `cloudflare/cloudflared` - Cloudflare Tunnel
- **WireGuard**: `lscr.io/linuxserver/wireguard` - VPN
- **Gluetun**: `qmcgaw/gluetun` - VPN 클라이언트

## A.7 홈 오토메이션

- **Home Assistant**: `ghcr.io/home-assistant/home-assistant` - 스마트홈
- **Node-RED**: `nodered/node-red` - 자동화 플로우
- **Mosquitto**: `eclipse-mosquitto` - MQTT 브로커
- **n8n**: `n8nio/n8n` - 워크플로우 자동화
- **ESPHome**: `ghcr.io/esphome/esphome` - ESP 펌웨어

---

모든 이미지는 Docker Hub 또는 GitHub Container Registry에서 다운로드 가능합니다.
