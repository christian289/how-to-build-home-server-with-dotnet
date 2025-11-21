# N100 미니 PC로 시작하는 홈서버 구축 완벽 가이드

## 제1부: 기초 준비편

### [제1장: 홈서버 입문](part1-basics/chapter01.md)

- [1.1 홈서버란 무엇인가?](part1-basics/chapter01.md#11-홈서버란-무엇인가)
- [1.2 N100 미니 PC가 홈서버로 적합한 이유](part1-basics/chapter01.md#12-n100-미니-pc가-홈서버로-적합한-이유)
  - [저전력 운영의 경제성](part1-basics/chapter01.md#저전력-운영의-경제성)
  - [24시간 운영에 최적화된 설계](part1-basics/chapter01.md#24시간-운영에-최적화된-설계)
  - [소음과 발열 관리](part1-basics/chapter01.md#소음과-발열-관리)
- [1.3 홈서버로 할 수 있는 일들](part1-basics/chapter01.md#13-홈서버로-할-수-있는-일들)
- [1.4 필요한 준비물과 예산 계획](part1-basics/chapter01.md#14-필요한-준비물과-예산-계획)

### [제2장: 하드웨어 준비](part1-basics/chapter02.md)

- [2.1 N100 미니 PC 선택 가이드](part1-basics/chapter02.md#21-n100-미니-pc-선택-가이드)
  - [제조사별 특징 비교](part1-basics/chapter02.md#제조사별-특징-비교)
  - [최소/권장 사양 결정하기](part1-basics/chapter02.md#최소권장-사양-결정하기)
- [2.2 스토리지 구성 전략](part1-basics/chapter02.md#22-스토리지-구성-전략)
  - [SSD vs HDD 선택 기준](part1-basics/chapter02.md#ssd-vs-hdd-선택-기준)
  - [RAID 구성 여부 결정](part1-basics/chapter02.md#raid-구성-여부-결정)
  - [외장 스토리지 활용법](part1-basics/chapter02.md#외장-스토리지-활용법)
- [2.3 네트워크 장비 준비](part1-basics/chapter02.md#23-네트워크-장비-준비)
  - [공유기 설정 최적화](part1-basics/chapter02.md#공유기-설정-최적화)
  - [유선/무선 네트워크 구성](part1-basics/chapter02.md#유선무선-네트워크-구성)
- [2.4 무정전 전원장치(UPS) 고려사항](part1-basics/chapter02.md#24-무정전-전원장치ups-고려사항)

### [제3장: 운영체제 선택과 설치](part1-basics/chapter03.md)

- [3.1 운영체제 비교 분석](part1-basics/chapter03.md#31-운영체제-비교-분석)
  - [Ubuntu Server LTS](part1-basics/chapter03.md#ubuntu-server-lts)
  - [Proxmox VE](part1-basics/chapter03.md#proxmox-ve)
  - [TrueNAS Scale](part1-basics/chapter03.md#truenas-scale)
  - [OpenMediaVault](part1-basics/chapter03.md#openmediavault)
  - [Unraid](part1-basics/chapter03.md#unraid)
- [3.2 Ubuntu Server 22.04 LTS 설치 (권장)](part1-basics/chapter03.md#32-ubuntu-server-2404-lts-설치)
  - [설치 USB 제작](part1-basics/chapter03.md#설치-usb-제작)
  - [기본 설치 과정](part1-basics/chapter03.md#기본-설치-과정)
  - [초기 설정 및 최적화](part1-basics/chapter03.md#초기-설정-및-최적화)
- [3.3 SSH 설정과 원격 접속 환경 구축](part1-basics/chapter03.md#33-ssh-설정과-원격-접속-환경-구축)
- [3.4 기본 보안 설정](part1-basics/chapter03.md#34-기본-보안-설정)

## 제2부: 핵심 인프라 구축

### [제4장: Docker 컨테이너 환경 구축](part2-infrastructure/chapter04.md)

- [4.1 Docker와 Docker Compose 설치](part2-infrastructure/chapter04.md#41-docker와-docker-compose-설치)
- [4.2 Portainer 설치로 GUI 관리 환경 구축](part2-infrastructure/chapter04.md#42-portainer-설치로-gui-관리-환경-구축)
- [4.3 컨테이너 네트워크 이해와 설정](part2-infrastructure/chapter04.md#43-컨테이너-네트워크-이해와-설정)
- [4.4 볼륨 관리와 데이터 지속성](part2-infrastructure/chapter04.md#44-볼륨-관리와-데이터-지속성)
- [4.5 컨테이너 자동 업데이트 설정 (Watchtower)](part2-infrastructure/chapter04.md#45-컨테이너-자동-업데이트-설정-watchtower)

### [제5장: 리버스 프록시와 도메인 설정](part2-infrastructure/chapter05.md)

- [5.1 Nginx Proxy Manager 설치](part2-infrastructure/chapter05.md#51-nginx-proxy-manager-설치)
- [5.2 도메인 구매와 DDNS 설정](part2-infrastructure/chapter05.md#52-도메인-구매와-ddns-설정)
  - [DuckDNS 활용법](part2-infrastructure/chapter05.md#duckdns-활용법)
  - [Cloudflare 연동](part2-infrastructure/chapter05.md#cloudflare-연동)
- [5.3 SSL 인증서 자동화 (Let's Encrypt)](part2-infrastructure/chapter05.md#53-ssl-인증서-자동화-lets-encrypt)
- [5.4 내부 DNS 서버 구축 (Pi-hole)](part2-infrastructure/chapter05.md#54-내부-dns-서버-구축-pi-hole)

### [제6장: 모니터링 시스템 구축](part2-infrastructure/chapter06.md)

- [6.1 시스템 모니터링](part2-infrastructure/chapter06.md#61-시스템-모니터링)
  - [Netdata 설치와 설정](part2-infrastructure/chapter06.md#netdata-설치와-설정)
  - [Grafana + Prometheus 구성](part2-infrastructure/chapter06.md#grafana--prometheus-구성)
- [6.2 로그 관리 시스템](part2-infrastructure/chapter06.md#62-로그-관리-시스템)
  - [Dozzle로 Docker 로그 관리](part2-infrastructure/chapter06.md#dozzle로-docker-로그-관리)
- [6.3 알림 설정](part2-infrastructure/chapter06.md#63-알림-설정)
  - [Discord 봇 연동](part2-infrastructure/chapter06.md#discord-봇-연동)
  - [Telegram 봇 연동](part2-infrastructure/chapter06.md#telegram-봇-연동)

## 제3부: 실용 서비스 구축

### [제7장: 파일 관리와 클라우드 스토리지](part3-services/chapter07.md)

- [7.1 Nextcloud 구축](part3-services/chapter07.md#71-nextcloud-구축)
  - [설치와 초기 설정](part3-services/chapter07.md#설치와-초기-설정)
  - [모바일 앱 연동](part3-services/chapter07.md#모바일-앱-연동)
  - [Office 문서 편집 기능 추가](part3-services/chapter07.md#office-문서-편집-기능-추가)
- [7.2 Syncthing으로 파일 동기화](part3-services/chapter07.md#72-syncthing으로-파일-동기화)
- [7.3 Filebrowser 설치](part3-services/chapter07.md#73-filebrowser-설치)
- [7.4 Samba 공유 폴더 설정](part3-services/chapter07.md#74-samba-공유-폴더-설정)

### [제8장: 미디어 서버 구축](part3-services/chapter08.md)

- [8.1 Jellyfin 미디어 서버](part3-services/chapter08.md#81-jellyfin-미디어-서버)
  - [설치와 라이브러리 구성](part3-services/chapter08.md#설치와-라이브러리-구성)
  - [하드웨어 가속 설정 (Intel QuickSync)](part3-services/chapter08.md#하드웨어-가속-설정-intel-quicksync)
  - [모바일/TV 앱 설정](part3-services/chapter08.md#모바일tv-앱-설정)
- [8.2 음악 스트리밍 (Navidrome)](part3-services/chapter08.md#82-음악-스트리밍-navidrome)
- [8.3 사진 관리 (Immich)](part3-services/chapter08.md#83-사진-관리-immich)
  - [AI 얼굴 인식 설정](part3-services/chapter08.md#ai-얼굴-인식-설정)
  - [모바일 자동 백업](part3-services/chapter08.md#모바일-자동-백업)
- [8.4 전자책 관리 (Calibre-Web)](part3-services/chapter08.md#84-전자책-관리-calibre-web)

### [제9장: 다운로드 자동화 시스템](part3-services/chapter09.md)

- [9.1 qBittorrent 설치와 설정](part3-services/chapter09.md#91-qbittorrent-설치와-설정)
- [9.2 Sonarr/Radarr/Prowlarr 연동](part3-services/chapter09.md#92-sonarrradarrprowlarr-연동)
- [9.3 YouTube 다운로드 자동화 (Tube Archivist)](part3-services/chapter09.md#93-youtube-다운로드-자동화-tube-archivist)
- [9.4 VPN 연결 설정 (Gluetun)](part3-services/chapter09.md#94-vpn-연결-설정-gluetun)

### [제10장: 개인 생산성 도구](part3-services/chapter10.md)

- [10.1 개인 위키 (WikiJS/BookStack)](part3-services/chapter10.md#101-개인-위키-wikijsbookstack)
- [10.2 노트 앱 (Joplin Server)](part3-services/chapter10.md#102-노트-앱-joplin-server)
- [10.3 할 일 관리 (Vikunja)](part3-services/chapter10.md#103-할-일-관리-vikunja)
- [10.4 비밀번호 관리자 (Vaultwarden)](part3-services/chapter10.md#104-비밀번호-관리자-vaultwarden)
- [10.5 RSS 리더 (FreshRSS)](part3-services/chapter10.md#105-rss-리더-freshrss)

### [제11장: 홈 자동화와 IoT](part3-services/chapter11.md)

- [11.1 Home Assistant 구축](part3-services/chapter11.md#111-home-assistant-구축)
  - [설치와 기본 설정](part3-services/chapter11.md#설치와-기본-설정)
  - [스마트 기기 연동](part3-services/chapter11.md#스마트-기기-연동)
  - [자동화 시나리오 작성](part3-services/chapter11.md#자동화-시나리오-작성)
- [11.2 MQTT 브로커 설정 (Mosquitto)](part3-services/chapter11.md#112-mqtt-브로커-설정-mosquitto)
- [11.3 Node-RED로 자동화 플로우 구성](part3-services/chapter11.md#113-node-red로-자동화-플로우-구성)
- [11.4 ESPHome 연동](part3-services/chapter11.md#114-esphome-연동)
- [11.5 N8N 워크플로우 자동화](part3-services/chapter11.md#115-n8n-워크플로우-자동화)
  - [N8N 설치와 기본 설정](part3-services/chapter11.md#n8n-설치와-기본-설정)
  - [웹훅과 API 연동](part3-services/chapter11.md#웹훅과-api-연동)
  - [실용적인 워크플로우 예제](part3-services/chapter11.md#실용적인-워크플로우-예제)
  - [외부 서비스 통합 (Google, Slack, Discord)](part3-services/chapter11.md#외부-서비스-통합-google-slack-discord)

### [제12장: 개발 환경 구축](part3-services/chapter12.md)

- [12.1 Git 서버 (Gitea)](part3-services/chapter12.md#121-git-서버-gitea)
- [12.2 CI/CD 파이프라인 (Gitea Actions)](part3-services/chapter12.md#122-cicd-파이프라인-gitea-actions)
- [12.3 코드 서버 (code-server)](part3-services/chapter12.md#123-코드-서버-code-server)
- [12.4 데이터베이스 관리](part3-services/chapter12.md#124-데이터베이스-관리)
  - [PostgreSQL/MySQL 설치](part3-services/chapter12.md#postgresqlmysql-설치)
  - [Adminer 웹 관리 도구](part3-services/chapter12.md#adminer-웹-관리-도구)
  - [pgAdmin (PostgreSQL 전용)](part3-services/chapter12.md#pgadmin-postgresql-전용)
  - [phpMyAdmin (MySQL/MariaDB 전용)](part3-services/chapter12.md#phpmyadmin-mysqlmariadb-전용)
- [12.5 Docker 레지스트리 (Harbor)](part3-services/chapter12.md#125-docker-레지스트리-harbor)
- [12.6 애플리케이션 모니터링 (Sentry)](part3-services/chapter12.md#126-애플리케이션-모니터링-sentry)
  - [.NET 애플리케이션에서 Sentry 연동](part3-services/chapter12.md#net-애플리케이션에서-sentry-연동)
- [12.7 패키지 저장소 (Nexus Repository)](part3-services/chapter12.md#127-패키지-저장소-nexus-repository)
  - [NuGet Feed 설정](part3-services/chapter12.md#nuget-feed-설정)
  - [npm Registry 설정 (Node.js)](part3-services/chapter12.md#npm-registry-설정-nodejs)
  - [Docker Registry 설정](part3-services/chapter12.md#docker-registry-설정)
  - [Maven/Gradle Repository](part3-services/chapter12.md#mavengradle-repository)

### [제13장: 웹 애플리케이션 개발과 배포](part3-services/chapter13.md)

- [13.1 .NET 로컬 개발 환경 구성](part3-services/chapter13.md#131-net-로컬-개발-환경-구성)
  - [.NET SDK 설치와 버전 관리](part3-services/chapter13.md#net-sdk-설치와-버전-관리)
  - [Visual Studio Code + Dev Containers](part3-services/chapter13.md#visual-studio-code--dev-containers)
  - [Docker Compose로 의존성 서비스 구축](part3-services/chapter13.md#docker-compose로-의존성-서비스-구축)
  - [Hot Reload와 Watch Mode](part3-services/chapter13.md#hot-reload와-watch-mode)
  - [개발용 HTTPS 인증서 (dotnet dev-certs)](part3-services/chapter13.md#개발용-https-인증서-dotnet-dev-certs)
- [13.2 .NET 애플리케이션 배포 전략](part3-services/chapter13.md#132-net-애플리케이션-배포-전략)
  - [Docker 컨테이너 배포](part3-services/chapter13.md#docker-컨테이너-배포)
  - [IIS on Linux (Kestrel + Nginx)](part3-services/chapter13.md#iis-on-linux-kestrel--nginx)
  - [.NET Runtime vs Self-Contained 배포](part3-services/chapter13.md#net-runtime-vs-self-contained-배포)
  - [Native AOT 최적화 배포](part3-services/chapter13.md#native-aot-최적화-배포)
  - [Health Checks와 Readiness Probes](part3-services/chapter13.md#health-checks와-readiness-probes)
  - [환경별 Configuration 관리](part3-services/chapter13.md#환경별-configuration-관리)
- [13.3 외부 접속 설정](part3-services/chapter13.md#133-외부-접속-설정)
  - [포트 포워딩 설정](part3-services/chapter13.md#포트-포워딩-설정)
  - [Cloudflare Tunnel 활용](part3-services/chapter13.md#cloudflare-tunnel-활용)
  - [Ngrok 대안 (Localtunnel, Bore)](part3-services/chapter13.md#ngrok-대안-localtunnel-bore)
  - [자체 터널링 서비스 구축 (Rathole)](part3-services/chapter13.md#자체-터널링-서비스-구축-rathole)
- [13.4 도메인 연결과 SSL](part3-services/chapter13.md#134-도메인-연결과-ssl)
  - [도메인 구매와 설정](part3-services/chapter13.md#도메인-구매와-설정)
  - [Cloudflare DNS 설정](part3-services/chapter13.md#cloudflare-dns-설정)
  - [SSL 인증서 자동화](part3-services/chapter13.md#ssl-인증서-자동화)
  - [서브도메인 라우팅](part3-services/chapter13.md#서브도메인-라우팅)
- 13.5 리버스 프록시 고급 설정
  - Traefik으로 동적 라우팅
  - 로드 밸런싱
  - 웹소켓 지원
  - 캐싱 전략
- 13.6 API 게이트웨이 구축
  - Kong Gateway 설정
  - 레이트 리미팅
  - API 키 관리
  - API 문서화 (Swagger/OpenAPI)
- 13.7 .NET 실시간 애플리케이션 배포
  - SignalR Hub 구축
  - Azure SignalR Service 연동
  - WebSocket 직접 구현
  - Server-Sent Events (SSE)
  - Long Polling 폴백 전략
- 13.8 .NET 데이터베이스 연동
  - Entity Framework Core 설정
  - Dapper를 활용한 경량 ORM
  - 연결 풀 관리와 성능 최적화
  - Code First Migration
  - Database First Scaffolding
  - 분산 캐싱 (Garnet/SQL Server)
- 13.9 .NET 애플리케이션 모니터링과 로깅
  - Application Insights 연동
  - Serilog 구조화된 로깅
  - Seq 로그 서버 구축
  - OpenTelemetry 구현
  - MiniProfiler로 성능 분석
  - Health Monitoring Dashboard
- 13.10 .NET 애플리케이션 실전 배포
  - ASP.NET Core Web API 배포
  - Blazor WebAssembly 애플리케이션
  - Blazor Server 애플리케이션
  - .NET MAUI Blazor Hybrid 웹뷰 연동
  - Minimal API with Native AOT
  - SignalR 실시간 애플리케이션
  - gRPC 서비스 배포
  - .NET Aspire 오케스트레이션

## 제4부: 보안과 유지보수

### [제14장: 보안 강화](part4-security/chapter14.md)

- [14.1 방화벽 설정 (ufw)](part4-security/chapter14.md#141-방화벽-설정-ufw)
- [14.2 Fail2ban으로 무차별 공격 방어](part4-security/chapter14.md#142-fail2ban으로-무차별-공격-방어)
- [14.3 VPN 서버 구축](part4-security/chapter14.md#143-vpn-서버-구축)
  - [WireGuard 설정](part4-security/chapter14.md#wireguard-설정)
  - OpenVPN 대안
- [14.4 Tailscale 메시 VPN](part4-security/chapter14.md#144-tailscale-메시-vpn)
  - [Tailscale 설치와 설정](part4-security/chapter14.md#tailscale-설치와-설정)
  - [Exit Node 설정으로 전체 트래픽 라우팅](part4-security/chapter14.md#exit-node-설정으로-전체-트래픽-라우팅)
  - Subnet Router로 홈 네트워크 접근
  - ACL(Access Control Lists) 설정
  - MagicDNS와 HTTPS 인증서
- 14.5 2단계 인증 구현 (Authelia)
- 14.6 보안 감사와 로그 분석

### [제15장: 백업과 복구 전략](part4-security/chapter15.md)

- [15.1 백업 전략 수립](part4-security/chapter15.md#151-백업-전략-수립)
  - [3-2-1 백업 규칙](part4-security/chapter15.md#3-2-1-백업-규칙)
  - [백업 대상 선정](part4-security/chapter15.md#백업-대상-선정)
- [15.2 자동 백업 시스템](part4-security/chapter15.md#152-자동-백업-시스템)
  - [Duplicati 설정](part4-security/chapter15.md#duplicati-설정)
  - [Restic 활용](part4-security/chapter15.md#restic-활용)
- [15.3 클라우드 백업 연동](part4-security/chapter15.md#153-클라우드-백업-연동)
  - Google Drive/OneDrive 연동
  - S3 호환 스토리지 활용
- [15.4 시스템 복구 절차](part4-security/chapter15.md#154-시스템-복구-절차)
- [15.5 재해 복구 계획](part4-security/chapter15.md#155-재해-복구-계획)

### [제16장: 성능 최적화](part4-security/chapter16.md)

- [16.1 시스템 리소스 최적화](part4-security/chapter16.md#161-시스템-리소스-최적화)
  - [CPU 거버너 설정](part4-security/chapter16.md#cpu-거버너-설정)
  - [메모리 관리](part4-security/chapter16.md#메모리-관리)
  - [스왑 최적화](part4-security/chapter16.md#스왑-최적화)
- [16.2 스토리지 성능 튜닝](part4-security/chapter16.md#162-스토리지-성능-튜닝)
  - 파일시스템 최적화
  - 캐시 설정
- [16.3 네트워크 최적화](part4-security/chapter16.md#163-네트워크-최적화)
- [16.4 Docker 컨테이너 리소스 제한](part4-security/chapter16.md#164-docker-컨테이너-리소스-제한)

### [제17장: 유지보수와 문제 해결](part4-security/chapter17.md)

- [17.1 정기 유지보수 체크리스트](part4-security/chapter17.md#171-정기-유지보수-체크리스트)
- [17.2 시스템 업데이트 전략](part4-security/chapter17.md#172-시스템-업데이트-전략)
- [17.3 로그 로테이션과 디스크 정리](part4-security/chapter17.md#173-로그-로테이션과-디스크-정리)
- [17.4 일반적인 문제와 해결 방법](part4-security/chapter17.md#174-일반적인-문제와-해결-방법)
  - [네트워크 문제](part4-security/chapter17.md#네트워크-문제)
  - 디스크 공간 부족
  - 컨테이너 충돌
  - 성능 저하

## 제5부: 고급 활용편

### [제18장: 쿠버네티스 입문](part5-advanced/chapter18.md)

- [18.1 K3s 설치와 설정](part5-advanced/chapter18.md#181-k3s-설치와-설정)
- [18.2 Helm으로 애플리케이션 배포](part5-advanced/chapter18.md#182-helm으로-애플리케이션-배포)
- [18.3 Longhorn으로 스토리지 관리](part5-advanced/chapter18.md#183-longhorn으로-스토리지-관리)
- [18.4 ArgoCD로 GitOps 구현](part5-advanced/chapter18.md#184-argocd로-gitops-구현)

### [제19장: 게임 서버 호스팅](part5-advanced/chapter19.md)

- [19.1 Minecraft 서버 구축](part5-advanced/chapter19.md#191-minecraft-서버-구축)
- [19.2 Valheim 서버 운영](part5-advanced/chapter19.md#192-valheim-서버-운영)
- [19.3 Pterodactyl Panel로 게임 서버 관리](part5-advanced/chapter19.md#193-pterodactyl-panel로-게임-서버-관리)
- [19.4 Steam 게임 스트리밍 (Sunshine/Moonlight)](part5-advanced/chapter19.md#194-steam-게임-스트리밍-sunshinemoonlight)

### [제20장: AI와 머신러닝 서비스](part5-advanced/chapter20.md)

- [20.1 로컬 LLM 실행 (Ollama)](part5-advanced/chapter20.md#201-로컬-llm-실행-ollama)
- [20.2 Stable Diffusion WebUI](part5-advanced/chapter20.md#202-stable-diffusion-webui)
- [20.3 음성 인식 서버 (Whisper)](part5-advanced/chapter20.md#203-음성-인식-서버-whisper)
- [20.4 얼굴 인식 시스템 구축](part5-advanced/chapter20.md#204-얼굴-인식-시스템-구축)

### [제21장: 전문 서비스 구축](part5-advanced/chapter21.md)

- [21.1 이메일 서버 (Mailcow)](part5-advanced/chapter21.md#211-이메일-서버-mailcow)
- [21.2 웹 분석 (Matomo/Plausible)](part5-advanced/chapter21.md#212-웹-분석-matomoplausible)
- [21.3 온라인 오피스 (OnlyOffice)](part5-advanced/chapter21.md#213-온라인-오피스-onlyoffice)
- [21.4 화상회의 서버 (Jitsi Meet)](part5-advanced/chapter21.md#214-화상회의-서버-jitsi-meet)

## 부록

### [부록 A: 유용한 Docker 이미지 모음](z-appendix/appendix-a.md)

- [A.1 필수 유틸리티](z-appendix/appendix-a.md#a1-필수-유틸리티)
- [A.2 개발 도구](z-appendix/appendix-a.md#a2-개발-도구)
- [A.3 미디어 관련](z-appendix/appendix-a.md#a3-미디어-관련)
- [A.4 생산성 도구](z-appendix/appendix-a.md#a4-생산성-도구)
- [A.5 모니터링 도구](z-appendix/appendix-a.md#a5-모니터링-도구)

### [부록 B: 스크립트와 자동화](z-appendix/appendix-b.md)

- [B.1 백업 스크립트](z-appendix/appendix-b.md#b1-백업-스크립트)
- [B.2 모니터링 스크립트](z-appendix/appendix-b.md#b2-모니터링-스크립트)
- [B.3 정리 스크립트](z-appendix/appendix-b.md#b3-정리-스크립트)
- [B.4 배포 자동화](z-appendix/appendix-b.md#b4-배포-자동화)
  - Docker 이미지 빌드 스크립트
  - Zero-Downtime 배포 스크립트
  - 롤백 스크립트
  - 헬스체크 스크립트

### [부록 C: 트러블슈팅 가이드](z-appendix/appendix-c.md)

- [C.1 포트 충돌 해결](z-appendix/appendix-c.md#c1-포트-충돌-해결)
- [C.2 권한 문제 해결](z-appendix/appendix-c.md#c2-권한-문제-해결)
- [C.3 네트워크 진단](z-appendix/appendix-c.md#c3-네트워크-진단)
- C.4 성능 병목 찾기

### [부록 D: 하드웨어 업그레이드 가이드](z-appendix/appendix-d.md)

- [D.1 RAM 추가](z-appendix/appendix-d.md#d1-ram-추가)
- [D.2 스토리지 확장](z-appendix/appendix-d.md#d2-스토리지-확장)
- [D.3 네트워크 카드 업그레이드](z-appendix/appendix-d.md#d3-네트워크-카드-업그레이드)
- D.4 쿨링 개선

### [부록 E: 커뮤니티와 리소스](z-appendix/appendix-e.md)

- [E.1 유용한 웹사이트](z-appendix/appendix-e.md#e1-유용한-웹사이트)
- [E.2 커뮤니티 포럼](z-appendix/appendix-e.md#e2-커뮤니티-포럼)
- [E.3 YouTube 채널](z-appendix/appendix-e.md#e3-youtube-채널)
- [E.4 추천 도서](z-appendix/appendix-e.md#e4-추천-도서)

### [부록 F: 용어 사전](z-appendix/appendix-f.md)

- [네트워크 용어](z-appendix/appendix-f.md#네트워크-용어)
- [컨테이너 관련 용어](z-appendix/appendix-f.md#컨테이너-관련-용어)
- [리눅스 관련 용어](z-appendix/appendix-f.md#리눅스-관련-용어)
- 홈서버 관련 약어

### [부록 G: 추가로 고려할 고급 서비스](z-appendix/appendix-g.md)

- [G.1 서비스 디스커버리 및 설정 관리](z-appendix/appendix-g.md#g1-서비스-디스커버리-및-설정-관리)
  - [Consul](z-appendix/appendix-g.md#consul)
- [G.2 시크릿 관리](z-appendix/appendix-g.md#g2-시크릿-관리)
  - [Vault](z-appendix/appendix-g.md#vault)
- [G.3 현대적인 리버스 프록시](z-appendix/appendix-g.md#g3-현대적인-리버스-프록시)
  - [Traefik](z-appendix/appendix-g.md#traefik)
- [G.4 메시지 큐](z-appendix/appendix-g.md#g4-메시지-큐)
  - [RabbitMQ](z-appendix/appendix-g.md#rabbitmq)
  - [Apache Kafka](z-appendix/appendix-g.md#apache-kafka)
- [G.5 오브젝트 스토리지](z-appendix/appendix-g.md#g5-오브젝트-스토리지)
  - [MinIO](z-appendix/appendix-g.md#minio)
- [G.6 로그 분석](z-appendix/appendix-g.md#g6-로그-분석)
  - [Elastic Stack (ELK)](z-appendix/appendix-g.md#elastic-stack-elk)
- [G.7 통합 인증 (SSO)](z-appendix/appendix-g.md#g7-통합-인증-sso)
  - [Keycloak](z-appendix/appendix-g.md#keycloak)
- [G.8 완전한 DevOps 플랫폼](z-appendix/appendix-g.md#g8-완전한-devops-플랫폼)
  - [GitLab](z-appendix/appendix-g.md#gitlab)
- [G.9 워크플로우 오케스트레이션](z-appendix/appendix-g.md#g9-워크플로우-오케스트레이션)
  - [Apache Airflow](z-appendix/appendix-g.md#apache-airflow)
- [G.10 분산 추적](z-appendix/appendix-g.md#g10-분산-추적)
  - [Jaeger](z-appendix/appendix-g.md#jaeger)
- [G.11 서비스별 권장 환경](z-appendix/appendix-g.md#g11-서비스별-권장-환경)
- [G.12 선택 가이드](z-appendix/appendix-g.md#g12-선택-가이드)

## [맺음말](conclusion.md)

- 지속적인 학습의 중요성
- 커뮤니티 참여
- 다음 단계로의 발전
