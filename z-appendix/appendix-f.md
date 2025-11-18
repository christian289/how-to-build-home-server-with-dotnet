# 부록 F: 용어 사전

이 부록은 책에서 사용된 주요 키워드와 용어를 정리한 사전입니다. 각 용어에 대한 간단한 설명과 공식 문서 링크를 제공합니다.

## 네트워크 용어

### DDNS (Dynamic DNS)
동적 IP 주소를 도메인 이름에 자동으로 매핑해주는 서비스입니다.
- 공식 문서: [DuckDNS](https://www.duckdns.org/)

### DNS (Domain Name System)
도메인 이름을 IP 주소로 변환하는 시스템입니다.
- 공식 문서: [Cloudflare DNS](https://www.cloudflare.com/learning/dns/what-is-dns/)

### IP 주소 (IP Address)
네트워크상에서 기기를 식별하는 고유한 주소입니다.
- IPv4: 192.168.0.1 형식
- IPv6: 2001:0db8:85a3::8a2e:0370:7334 형식

### NAT (Network Address Translation)
사설 IP 주소를 공인 IP 주소로 변환하는 기술입니다.

### 포트 포워딩 (Port Forwarding)
공유기의 특정 포트로 들어오는 트래픽을 내부 기기로 전달하는 설정입니다.

### 리버스 프록시 (Reverse Proxy)
클라이언트의 요청을 받아 백엔드 서버로 전달하는 중개 서버입니다.
- 공식 문서: [Nginx](https://nginx.org/)

### SSL/TLS
웹사이트와 브라우저 간 데이터를 암호화하는 보안 프로토콜입니다.
- 공식 문서: [Let's Encrypt](https://letsencrypt.org/)

### VPN (Virtual Private Network)
안전한 암호화된 연결을 통해 원격 네트워크에 접속하는 기술입니다.
- 공식 문서: [WireGuard](https://www.wireguard.com/)
- 공식 문서: [Tailscale](https://tailscale.com/)

### VLAN (Virtual LAN)
물리적 네트워크를 논리적으로 분리하는 기술입니다.
- 공식 문서: [Wikipedia - VLAN](https://en.wikipedia.org/wiki/Virtual_LAN)

## 컨테이너 관련 용어

### Docker
애플리케이션을 컨테이너로 패키징하고 실행하는 플랫폼입니다.
- 공식 문서: [Docker](https://docs.docker.com/)

### Docker Compose
여러 Docker 컨테이너를 정의하고 실행하는 도구입니다.
- 공식 문서: [Docker Compose](https://docs.docker.com/compose/)

### 컨테이너 (Container)
애플리케이션과 그 의존성을 격리된 환경에서 실행하는 경량 가상화 기술입니다.

### 이미지 (Image)
컨테이너를 생성하기 위한 읽기 전용 템플릿입니다.

### 볼륨 (Volume)
컨테이너의 데이터를 영구적으로 저장하는 저장 공간입니다.

### Docker Hub
Docker 이미지를 공유하고 다운로드할 수 있는 레지스트리입니다.
- 공식 사이트: [Docker Hub](https://hub.docker.com/)

### Kubernetes (K8s)
컨테이너 오케스트레이션 플랫폼으로, 대규모 컨테이너 관리에 사용됩니다.
- 공식 문서: [Kubernetes](https://kubernetes.io/)

### K3s
경량화된 Kubernetes 배포판으로, 홈서버나 엣지 환경에 적합합니다.
- 공식 문서: [K3s](https://k3s.io/)

### Portainer
Docker 환경을 웹 GUI로 관리할 수 있는 도구입니다.
- 공식 문서: [Portainer](https://www.portainer.io/)

## 리눅스 관련 용어

### Ubuntu Server
Canonical에서 개발한 Linux 배포판의 서버 버전입니다.
- 공식 사이트: [Ubuntu](https://ubuntu.com/server)

### LTS (Long-Term Support)
장기 지원 버전으로, 5년간 보안 업데이트를 제공합니다.

### SSH (Secure Shell)
원격 서버에 안전하게 접속하기 위한 프로토콜입니다.
- 공식 문서: [OpenSSH](https://www.openssh.com/)

### UFW (Uncomplicated Firewall)
Ubuntu의 간편한 방화벽 관리 도구입니다.
- 공식 문서: [Ubuntu Firewall](https://help.ubuntu.com/community/UFW)

### Systemd
Linux 시스템의 서비스를 관리하는 시스템 매니저입니다.
- 공식 문서: [systemd](https://systemd.io/)

### Cron
Linux에서 작업을 주기적으로 실행하는 스케줄러입니다.

### Bash
Linux의 기본 커맨드 라인 셸입니다.

### Sudo
일반 사용자가 root 권한으로 명령을 실행할 수 있게 해주는 도구입니다.

## 홈서버 관련 서비스

### Nextcloud
개인 클라우드 스토리지 플랫폼입니다.
- 공식 사이트: [Nextcloud](https://nextcloud.com/)

### Jellyfin
오픈소스 미디어 서버 소프트웨어입니다.
- 공식 사이트: [Jellyfin](https://jellyfin.org/)

### Pi-hole
네트워크 전체의 광고를 차단하는 DNS 서버입니다.
- 공식 사이트: [Pi-hole](https://pi-hole.net/)

### Home Assistant
홈 자동화 플랫폼입니다.
- 공식 사이트: [Home Assistant](https://www.home-assistant.io/)

### Vaultwarden
Bitwarden 호환 비밀번호 관리 서버입니다.
- 공식 문서: [Vaultwarden](https://github.com/dani-garcia/vaultwarden)

### Gitea
경량 Git 서버입니다.
- 공식 사이트: [Gitea](https://gitea.io/)

### Grafana
메트릭 시각화 및 대시보드 도구입니다.
- 공식 사이트: [Grafana](https://grafana.com/)

### Prometheus
오픈소스 모니터링 시스템입니다.
- 공식 사이트: [Prometheus](https://prometheus.io/)

## .NET 관련 용어

### .NET SDK
.NET 애플리케이션을 개발하기 위한 소프트웨어 개발 키트입니다.
- 공식 문서: [.NET](https://dotnet.microsoft.com/)

### ASP.NET Core
크로스 플랫폼 웹 애플리케이션 프레임워크입니다.
- 공식 문서: [ASP.NET Core](https://learn.microsoft.com/en-us/aspnet/core/)

### Kestrel
ASP.NET Core의 내장 웹 서버입니다.
- 공식 문서: [Kestrel](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/servers/kestrel)

### Entity Framework Core
.NET용 ORM (Object-Relational Mapping) 프레임워크입니다.
- 공식 문서: [EF Core](https://learn.microsoft.com/en-us/ef/core/)

### SignalR
실시간 웹 기능을 추가하는 라이브러리입니다.
- 공식 문서: [SignalR](https://learn.microsoft.com/en-us/aspnet/core/signalr/)

### Blazor
C#으로 웹 UI를 구축하는 프레임워크입니다.
- 공식 문서: [Blazor](https://dotnet.microsoft.com/apps/aspnet/web-apps/blazor)

### Native AOT
네이티브 코드로 미리 컴파일하는 배포 방식입니다.
- 공식 문서: [Native AOT](https://learn.microsoft.com/en-us/dotnet/core/deploying/native-aot/)

### NuGet
.NET 패키지 관리자입니다.
- 공식 사이트: [NuGet](https://www.nuget.org/)

## 데이터베이스 용어

### PostgreSQL
오픈소스 관계형 데이터베이스입니다.
- 공식 사이트: [PostgreSQL](https://www.postgresql.org/)

### Redis
인메모리 키-값 데이터베이스입니다.
- 공식 사이트: [Redis](https://redis.io/)

### SQLite
경량 파일 기반 데이터베이스입니다.
- 공식 사이트: [SQLite](https://www.sqlite.org/)

### MySQL/MariaDB
널리 사용되는 오픈소스 관계형 데이터베이스입니다.
- 공식 사이트: [MySQL](https://www.mysql.com/)
- 공식 사이트: [MariaDB](https://mariadb.org/)

## 하드웨어 관련 용어

### Intel N100
저전력 4코어 프로세서입니다.
- 공식 사양: [Intel N100](https://www.intel.com/content/www/us/en/products/sku/231803/intel-processor-n100-6m-cache-up-to-3-40-ghz/specifications.html)

### TDP (Thermal Design Power)
프로세서가 소비하는 최대 전력량을 나타내는 지표입니다.

### SSD (Solid State Drive)
플래시 메모리 기반의 고속 저장장치입니다.

### HDD (Hard Disk Drive)
자기 디스크 기반의 저장장치입니다.

### NVMe (Non-Volatile Memory Express)
PCIe 기반의 고속 SSD 인터페이스입니다.

### RAID (Redundant Array of Independent Disks)
여러 디스크를 하나로 묶어 성능 향상이나 중복성을 제공하는 기술입니다.
- 공식 문서: [Wikipedia - RAID](https://en.wikipedia.org/wiki/RAID)

### UPS (Uninterruptible Power Supply)
무정전 전원 공급 장치입니다.
- 공식 문서: [Wikipedia - UPS](https://en.wikipedia.org/wiki/Uninterruptible_power_supply)

## 기타 약어

### API (Application Programming Interface)
애플리케이션 간 상호작용을 위한 인터페이스입니다.

### CI/CD (Continuous Integration/Continuous Deployment)
지속적 통합 및 배포 파이프라인입니다.

### HTTPS (Hypertext Transfer Protocol Secure)
HTTP의 보안 버전입니다.

### JSON (JavaScript Object Notation)
데이터 교환 형식입니다.

### YAML (YAML Ain't Markup Language)
설정 파일에 주로 사용되는 데이터 직렬화 형식입니다.

### REST (Representational State Transfer)
웹 API 설계 아키텍처 스타일입니다.

### WebSocket
양방향 실시간 통신 프로토콜입니다.

### OAuth
인증 및 권한 부여를 위한 개방형 표준입니다.

### JWT (JSON Web Token)
JSON 기반의 토큰 인증 방식입니다.

---

이 용어 사전은 지속적으로 업데이트될 예정입니다. 새로운 기술이나 서비스가 추가되면 함께 업데이트하겠습니다.
