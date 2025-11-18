# 제14장: 보안 강화

## 14.1 방화벽 설정 (ufw)

제3장에서 기본 설정을 다뤘습니다. 추가 포트 관리:

```bash
# 특정 포트 허용
sudo ufw allow 8080/tcp

# 특정 IP에서만 허용
sudo ufw allow from 192.168.0.50 to any port 22

# 포트 범위 허용
sudo ufw allow 8000:8100/tcp

# 규칙 삭제
sudo ufw delete allow 8080/tcp

# 상태 확인
sudo ufw status numbered
```

## 14.2 Fail2ban으로 무차별 공격 방어

제3장에서 기본 설정을 다뤘습니다. 추가 jail 설정:

```bash
# Docker 컨테이너 로그 모니터링
sudo nano /etc/fail2ban/jail.local
```

```ini
[nginx-proxy-manager]
enabled = true
filter = nginx-proxy-manager
logpath = /var/log/nginx/error.log
maxretry = 3
bantime = 3600
```

## 14.3 VPN 서버 구축

### WireGuard 설정

```yaml
version: '3.8'

services:
  wireguard:
    image: lscr.io/linuxserver/wireguard:latest
    container_name: wireguard
    cap_add:
      - NET_ADMIN
      - SYS_MODULE
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Asia/Seoul
      - SERVERURL=your.domain.com
      - SERVERPORT=51820
      - PEERS=phone,laptop,tablet
      - PEERDNS=auto
      - INTERNAL_SUBNET=10.13.13.0
    volumes:
      - ./wireguard:/config
      - /lib/modules:/lib/modules
    ports:
      - 51820:51820/udp
    sysctls:
      - net.ipv4.conf.all.src_valid_mark=1
    restart: unless-stopped
```

클라이언트 설정은 `./wireguard/peerX/peerX.conf` 참조.

## 14.4 Tailscale 메시 VPN

[Tailscale](https://tailscale.com/)은 설정이 간단한 메시 VPN입니다.

### Tailscale 설치와 설정

```bash
# Tailscale 설치
curl -fsSL https://tailscale.com/install.sh | sh

# 인증
sudo tailscale up

# 상태 확인
tailscale status
```

### Exit Node 설정으로 전체 트래픽 라우팅

```bash
# 서버를 Exit Node로 설정
sudo tailscale up --advertise-exit-node

# 클라이언트에서 Exit Node 사용
tailscale up --exit-node=homeserver
```

### Subnet Router로 홈 네트워크 접근

```bash
# 서버에서 서브넷 광고
sudo tailscale up --advertise-routes=192.168.0.0/24

# Admin Console에서 승인
# https://login.tailscale.com/admin/machines
```

이제 외부에서도 홈 네트워크의 모든 기기에 접근 가능합니다.

## 14.5 2단계 인증 구현 (Authelia)

[Authelia](https://www.authelia.com/)로 서비스에 2FA 추가:

```yaml
version: '3.8'

services:
  authelia:
    image: authelia/authelia:latest
    container_name: authelia
    volumes:
      - ./authelia:/config
    ports:
      - 9091:9091
    restart: unless-stopped
    environment:
      - TZ=Asia/Seoul
```

Nginx Proxy Manager에서 각 서비스에 Authelia 연동.

## 14.6 보안 감사와 로그 분석

### Lynis 보안 감사

```bash
# Lynis 설치
sudo apt install lynis -y

# 보안 감사 실행
sudo lynis audit system

# 보고서 확인
cat /var/log/lynis-report.dat
```

### ClamAV 바이러스 스캔

```bash
# ClamAV 설치
sudo apt install clamav clamav-daemon -y

# 바이러스 정의 업데이트
sudo freshclam

# 스캔
sudo clamscan -r /home --infected --remove
```

---

**다음 장에서는**: 백업 전략과 시스템 복구 방법을 알아보겠습니다.
