# N100 미니 PC로 시작하는 홈서버 구축 완벽 가이드

## 제1부: 기초 준비편

### [제1장: 홈서버 입문](part1-basics/chapter01.md)

- 1.1 홈서버란 무엇인가?
- 1.2 N100 미니 PC가 홈서버로 적합한 이유
  - 저전력 운영의 경제성
  - 24시간 운영에 최적화된 설계
  - 소음과 발열 관리
- 1.3 홈서버로 할 수 있는 일들
- 1.4 필요한 준비물과 예산 계획

### [제2장: 하드웨어 준비](part1-basics/chapter02.md)

- 2.1 N100 미니 PC 선택 가이드
  - 제조사별 특징 비교
  - 최소/권장 사양 결정하기
- 2.2 스토리지 구성 전략
  - SSD vs HDD 선택 기준
  - RAID 구성 여부 결정
  - 외장 스토리지 활용법
- 2.3 네트워크 장비 준비
  - 공유기 설정 최적화
  - 유선/무선 네트워크 구성
- 2.4 무정전 전원장치(UPS) 고려사항

### [제3장: 운영체제 선택과 설치](part1-basics/chapter03.md)

- 3.1 운영체제 비교 분석
  - Ubuntu Server LTS
  - Proxmox VE
  - TrueNAS Scale
  - OpenMediaVault
  - Unraid
- 3.2 Ubuntu Server 22.04 LTS 설치 (권장)
  - 설치 USB 제작
  - 기본 설치 과정
  - 초기 설정 및 최적화
- 3.3 SSH 설정과 원격 접속 환경 구축
- 3.4 기본 보안 설정

## 제2부: 핵심 인프라 구축

### [제4장: Docker 컨테이너 환경 구축](part2-infrastructure/chapter04.md)

- 4.1 Docker와 Docker Compose 설치
- 4.2 Portainer 설치로 GUI 관리 환경 구축
- 4.3 컨테이너 네트워크 이해와 설정
- 4.4 볼륨 관리와 데이터 지속성
- 4.5 컨테이너 자동 업데이트 설정 (Watchtower)

### [제5장: 리버스 프록시와 도메인 설정](part2-infrastructure/chapter05.md)

- 5.1 Nginx Proxy Manager 설치
- 5.2 도메인 구매와 DDNS 설정
  - DuckDNS 활용법
  - Cloudflare 연동
- 5.3 SSL 인증서 자동화 (Let's Encrypt)
- 5.4 내부 DNS 서버 구축 (Pi-hole)

### [제6장: 모니터링 시스템 구축](part2-infrastructure/chapter06.md)

- 6.1 시스템 모니터링
  - Netdata 설치와 설정
  - Grafana + Prometheus 구성
- 6.2 로그 관리 시스템
  - Dozzle로 Docker 로그 관리
- 6.3 알림 설정
  - Discord/Telegram 봇 연동
  - 이메일 알림 구성

## 제3부: 실용 서비스 구축

### [제7장: 파일 관리와 클라우드 스토리지](part3-services/chapter07.md)

- 7.1 Nextcloud 구축
  - 설치와 초기 설정
  - 모바일 앱 연동
  - Office 문서 편집 기능 추가
- 7.2 Syncthing으로 파일 동기화
- 7.3 Filebrowser 설치
- 7.4 Samba 공유 폴더 설정

### [제8장: 미디어 서버 구축](part3-services/chapter08.md)

- 8.1 Jellyfin 미디어 서버
  - 설치와 라이브러리 구성
  - 하드웨어 가속 설정 (Intel QuickSync)
  - 모바일/TV 앱 설정
- 8.2 음악 스트리밍 (Navidrome)
- 8.3 사진 관리 (Immich)
  - AI 얼굴 인식 설정
  - 모바일 자동 백업
- 8.4 전자책 관리 (Calibre-Web)

### [제9장: 다운로드 자동화 시스템](part3-services/chapter09.md)

- 9.1 qBittorrent 설치와 설정
- 9.2 Sonarr/Radarr/Prowlarr 연동
- 9.3 YouTube 다운로드 자동화 (Tube Archivist)
- 9.4 VPN 연결 설정 (Gluetun)

### [제10장: 개인 생산성 도구](part3-services/chapter10.md)

- 10.1 개인 위키 (WikiJS/BookStack)
- 10.2 노트 앱 (Joplin Server)
- 10.3 할 일 관리 (Vikunja)
- 10.4 비밀번호 관리자 (Vaultwarden)
- 10.5 RSS 리더 (FreshRSS)

### [제11장: 홈 자동화와 IoT](part3-services/chapter11.md)

- 11.1 Home Assistant 구축
  - 설치와 기본 설정
  - 스마트 기기 연동
  - 자동화 시나리오 작성
- 11.2 MQTT 브로커 설정 (Mosquitto)
- 11.3 Node-RED로 자동화 플로우 구성
- 11.4 ESPHome 연동
- 11.5 N8N 워크플로우 자동화
  - N8N 설치와 기본 설정
  - 웹훅과 API 연동
  - 실용적인 워크플로우 예제
  - 외부 서비스 통합 (Google, Slack, Discord)

### [제12장: 개발 환경 구축](part3-services/chapter12.md)

- 12.1 Git 서버 (Gitea)
- 12.2 CI/CD 파이프라인 (Drone CI)
- 12.3 코드 서버 (code-server)
- 12.4 데이터베이스 관리
  - PostgreSQL/MySQL 설치
  - Adminer 웹 관리 도구

### [제13장: 웹 애플리케이션 개발과 배포](part3-services/chapter13.md)

- 13.1 .NET 로컬 개발 환경 구성
  - .NET SDK 설치와 버전 관리
  - Visual Studio Code + Dev Containers
  - Docker Compose로 의존성 서비스 구축
  - Hot Reload와 Watch Mode
  - 개발용 HTTPS 인증서 (dotnet dev-certs)
- 13.2 .NET 애플리케이션 배포 전략
  - Docker 컨테이너 배포
  - IIS on Linux (Kestrel + Nginx)
  - .NET Runtime vs Self-Contained 배포
  - Native AOT 최적화 배포
  - Health Checks와 Readiness Probes
  - 환경별 Configuration 관리
- 13.3 외부 접속 설정
  - 포트 포워딩 설정
  - Cloudflare Tunnel 활용
  - Ngrok 대안 (Localtunnel, Bore)
  - 자체 터널링 서비스 구축 (Rathole)
- 13.4 도메인 연결과 SSL
  - 도메인 구매와 설정
  - Cloudflare DNS 설정
  - SSL 인증서 자동화
  - 서브도메인 라우팅
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
  - 분산 캐싱 (Redis/SQL Server)
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

- 14.1 방화벽 설정 (ufw)
- 14.2 Fail2ban으로 무차별 공격 방어
- 14.3 VPN 서버 구축
  - WireGuard 설정
  - OpenVPN 대안
- 14.4 Tailscale 메시 VPN
  - Tailscale 설치와 설정
  - Exit Node 설정으로 전체 트래픽 라우팅
  - Subnet Router로 홈 네트워크 접근
  - ACL(Access Control Lists) 설정
  - MagicDNS와 HTTPS 인증서
- 14.5 2단계 인증 구현 (Authelia)
- 14.6 보안 감사와 로그 분석

### [제15장: 백업과 복구 전략](part4-security/chapter15.md)

- 15.1 백업 전략 수립
  - 3-2-1 백업 규칙
  - 백업 대상 선정
- 15.2 자동 백업 시스템
  - Duplicati 설정
  - Restic 활용
- 15.3 클라우드 백업 연동
  - Google Drive/OneDrive 연동
  - S3 호환 스토리지 활용
- 15.4 시스템 복구 절차
- 15.5 재해 복구 계획

### [제16장: 성능 최적화](part4-security/chapter16.md)

- 16.1 시스템 리소스 최적화
  - CPU 거버너 설정
  - 메모리 관리
  - 스왑 최적화
- 16.2 스토리지 성능 튜닝
  - 파일시스템 최적화
  - 캐시 설정
- 16.3 네트워크 최적화
- 16.4 Docker 컨테이너 리소스 제한

### [제17장: 유지보수와 문제 해결](part4-security/chapter17.md)

- 17.1 정기 유지보수 체크리스트
- 17.2 시스템 업데이트 전략
- 17.3 로그 로테이션과 디스크 정리
- 17.4 일반적인 문제와 해결 방법
  - 네트워크 문제
  - 디스크 공간 부족
  - 컨테이너 충돌
  - 성능 저하

## 제5부: 고급 활용편

### [제18장: 쿠버네티스 입문](part5-advanced/chapter18.md)

- 18.1 K3s 설치와 설정
- 18.2 Helm으로 애플리케이션 배포
- 18.3 Longhorn으로 스토리지 관리
- 18.4 ArgoCD로 GitOps 구현

### [제19장: 게임 서버 호스팅](part5-advanced/chapter19.md)

- 19.1 Minecraft 서버 구축
- 19.2 Valheim 서버 운영
- 19.3 Pterodactyl Panel로 게임 서버 관리
- 19.4 Steam 게임 스트리밍 (Sunshine/Moonlight)

### [제20장: AI와 머신러닝 서비스](part5-advanced/chapter20.md)

- 20.1 로컬 LLM 실행 (Ollama)
- 20.2 Stable Diffusion WebUI
- 20.3 음성 인식 서버 (Whisper)
- 20.4 얼굴 인식 시스템 구축

### [제21장: 전문 서비스 구축](part5-advanced/chapter21.md)

- 21.1 이메일 서버 (Mailcow)
- 21.2 웹 분석 (Matomo/Plausible)
- 21.3 온라인 오피스 (OnlyOffice)
- 21.4 화상회의 서버 (Jitsi Meet)

## 부록

### [부록 A: 유용한 Docker 이미지 모음](z-appendix/appendix-a.md)

- A.1 필수 유틸리티
- A.2 개발 도구
- A.3 미디어 관련
- A.4 생산성 도구
- A.5 모니터링 도구

### [부록 B: 스크립트와 자동화](z-appendix/appendix-b.md)

- B.1 백업 스크립트
- B.2 모니터링 스크립트
- B.3 정리 스크립트
- B.4 배포 자동화
  - Docker 이미지 빌드 스크립트
  - Zero-Downtime 배포 스크립트
  - 롤백 스크립트
  - 헬스체크 스크립트

### [부록 C: 트러블슈팅 가이드](z-appendix/appendix-c.md)

- C.1 포트 충돌 해결
- C.2 권한 문제 해결
- C.3 네트워크 진단
- C.4 성능 병목 찾기

### [부록 D: 하드웨어 업그레이드 가이드](z-appendix/appendix-d.md)

- D.1 RAM 추가
- D.2 스토리지 확장
- D.3 네트워크 카드 업그레이드
- D.4 쿨링 개선

### [부록 E: 커뮤니티와 리소스](z-appendix/appendix-e.md)

- E.1 유용한 웹사이트
- E.2 커뮤니티 포럼
- E.3 YouTube 채널
- E.4 추천 도서

### [부록 F: 용어 사전](z-appendix/appendix-f.md)

- 네트워크 용어
- 컨테이너 관련 용어
- 리눅스 명령어
- 홈서버 관련 약어

## [맺음말](conclusion.md)

- 지속적인 학습의 중요성
- 커뮤니티 참여
- 다음 단계로의 발전
