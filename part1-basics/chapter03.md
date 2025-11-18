# 제3장: 운영체제 선택과 설치

## 3.1 운영체제 비교 분석

홈서버용 운영체제는 다양한 선택지가 있습니다. 각 OS의 특징을 비교하여 자신에게 맞는 것을 선택하세요.

### Ubuntu Server LTS

[Ubuntu Server](https://ubuntu.com/download/server)는 가장 널리 사용되는 Linux 서버 배포판입니다.

**장점**:
- 방대한 커뮤니티와 문서
- 5년간 보안 업데이트 제공 (LTS 버전)
- 최신 소프트웨어 지원
- Docker, Kubernetes 등 컨테이너 기술과 완벽 호환
- 초보자도 쉽게 접근 가능

**단점**:
- GUI가 기본 제공되지 않음 (CLI 위주)
- 일부 하드웨어 드라이버 문제 가능성

**권장 대상**:
- Linux가 처음이거나 중급 사용자
- Docker 기반 서비스 운영
- 개발 환경 구축
- **본 가이드의 기본 OS로 권장**

**최신 LTS 버전**: Ubuntu 24.04 LTS (Noble Numbat)

### Proxmox VE

[Proxmox VE](https://www.proxmox.com/en/proxmox-ve)는 가상화 전문 플랫폼입니다.

**장점**:
- 웹 GUI 제공
- KVM 가상 머신 + LXC 컨테이너 지원
- 클러스터링 기능 (여러 서버 관리)
- 스냅샷, 백업 기능 내장

**단점**:
- N100 같은 저사양에서는 오버헤드 발생
- 가상화 레이어로 인한 성능 저하
- 설정이 복잡함

**권장 대상**:
- 여러 OS를 동시에 실행하고 싶은 사용자
- 가상화 환경 학습
- 성능보다 유연성을 중시

### TrueNAS Scale

[TrueNAS Scale](https://www.truenas.com/truenas-scale/)은 NAS 전문 OS입니다.

**장점**:
- ZFS 파일시스템 지원 (데이터 무결성)
- 웹 GUI 제공
- Docker/Kubernetes 앱 지원
- RAID, 스냅샷 기능 강력

**단점**:
- ZFS는 최소 8GB RAM 권장 (16GB 이상 이상적)
- 주 목적이 스토리지 관리이므로 범용 서버로는 제한적
- 업데이트 시 시스템 재부팅 필수

**권장 대상**:
- 대용량 스토리지 관리가 주 목적
- 데이터 무결성이 매우 중요
- RAM이 16GB 이상

### OpenMediaVault

[OpenMediaVault](https://www.openmediavault.org/)는 Debian 기반 NAS OS입니다.

**장점**:
- 가볍고 빠름
- 웹 GUI 제공
- 플러그인 시스템
- 낮은 리소스 요구사항

**단점**:
- 기능이 TrueNAS보다 제한적
- Docker 지원은 있으나 별도 플러그인 필요

**권장 대상**:
- 가벼운 NAS 전용 시스템
- 웹 GUI를 선호하지만 TrueNAS는 과한 경우

### Unraid

[Unraid](https://unraid.net/)는 상용 NAS/서버 OS입니다.

**장점**:
- 매우 직관적인 웹 GUI
- 유연한 스토리지 관리 (서로 다른 크기의 HDD 혼용 가능)
- Docker, VM 지원
- 커뮤니티 앱 스토어

**단점**:
- **유료** (Basic $59, Plus $89, Pro $129)
- 독점 소프트웨어
- USB 부팅 필수 (USB가 라이선스 키 역할)

**권장 대상**:
- GUI 중심으로 편하게 관리하고 싶은 사용자
- 비용 지불 가능
- 다양한 크기의 HDD를 효율적으로 활용

### 비교 표

| 특성 | Ubuntu Server | Proxmox | TrueNAS Scale | OMV | Unraid |
|------|---------------|---------|---------------|-----|--------|
| **난이도** | 중 | 중상 | 중 | 하 | 하 |
| **GUI** | 없음 | 웹 | 웹 | 웹 | 웹 |
| **Docker 지원** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| **가상화** | KVM (수동) | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐ | ⭐⭐⭐⭐ |
| **저사양 적합성** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| **커뮤니티** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐ |
| **비용** | 무료 | 무료 | 무료 | 무료 | 유료 |
| **권장 용도** | 범용 서버 | 가상화 | NAS | 간단 NAS | 올인원 |

**결론**: 본 가이드에서는 **Ubuntu Server 24.04 LTS**를 기준으로 설명합니다.

## 3.2 Ubuntu Server 24.04 LTS 설치

### 설치 USB 제작

#### 1. Ubuntu Server ISO 다운로드

공식 사이트에서 최신 LTS 버전을 다운로드합니다:
- URL: https://ubuntu.com/download/server
- 파일명 예시: `ubuntu-24.04.1-live-server-amd64.iso`
- 용량: 약 2GB

#### 2. USB 부팅 디스크 제작 도구

**Windows 사용자**:
- [Rufus](https://rufus.ie/) 다운로드 및 실행
- ISO 파일 선택
- 파티션 방식: GPT
- 대상 시스템: UEFI
- "시작" 클릭

**macOS 사용자**:
- [balenaEtcher](https://www.balena.io/etcher/) 다운로드
- ISO 파일 선택
- USB 드라이브 선택
- "Flash!" 클릭

**Linux 사용자**:
```bash
# USB 장치 확인
lsblk

# dd 명령으로 직접 작성 (sdX는 실제 USB 장치명)
sudo dd if=ubuntu-24.04.1-live-server-amd64.iso of=/dev/sdX bs=4M status=progress && sync
```

### 기본 설치 과정

#### 1. BIOS 설정

미니 PC를 USB로 부팅하려면 BIOS 설정이 필요합니다.

1. 전원을 켜고 **Del** 또는 **F2** 키를 연타하여 BIOS 진입
2. **Boot** 메뉴로 이동
3. **Boot Priority**에서 USB를 1순위로 설정
4. **F10** 키로 저장 후 재부팅

#### 2. Ubuntu Server 설치 마법사

**언어 선택**:
- English 권장 (한글 선택 시 CLI에서 깨질 수 있음)

**키보드 레이아웃**:
- Korean (또는 English US)

**네트워크 연결**:
- 유선 연결되어 있으면 자동으로 DHCP로 IP 할당
- 고정 IP 설정은 설치 후에도 가능

**프록시 설정**:
- 보통 비워두고 진행

**미러 사이트**:
- 기본값 사용 (한국에서는 자동으로 카카오 미러 선택됨)

**스토리지 구성**:

**옵션 1: 자동 (전체 디스크 사용)**
```
Use an entire disk
[v] Set up this disk as an LVM group
```
- 간단하고 안전
- 추후 파티션 확장 가능 (LVM 사용 시)

**옵션 2: 수동 (커스텀 파티션)**
```
/dev/nvme0n1
├── /dev/nvme0n1p1  512MB   EFI System Partition
├── /dev/nvme0n1p2  50GB    / (root)
└── /dev/nvme0n1p3  나머지  /data
```
- 시스템과 데이터 분리
- 고급 사용자에게 권장

**프로필 설정**:
```
Your name: 홍길동
Your server's name: homeserver
Pick a username: gildong
Choose a password: ********
```

**SSH 설정**:
- [v] **Install OpenSSH server** (반드시 체크!)
- [ ] Import SSH identity (선택사항)

**Featured Server Snaps**:
- Docker는 여기서 설치하지 말고 나중에 수동 설치 권장
- 모두 체크 해제하고 진행

**설치 완료**:
- 설치가 완료되면 "Reboot Now" 선택
- USB를 제거하라는 메시지가 나오면 USB 제거 후 Enter

### 초기 설정 및 최적화

#### 첫 로그인

```bash
# 미니 PC에 모니터와 키보드를 연결하거나,
# SSH로 접속 (IP 주소는 공유기 관리 페이지에서 확인)
ssh gildong@192.168.0.100
```

#### 시스템 업데이트

```bash
# 패키지 목록 갱신
sudo apt update

# 설치된 패키지 업그레이드
sudo apt upgrade -y

# 불필요한 패키지 제거
sudo apt autoremove -y

# 재부팅 (커널 업데이트 시)
sudo reboot
```

#### 호스트명 설정 (선택사항)

```bash
# 호스트명 변경
sudo hostnamectl set-hostname homeserver

# 확인
hostnamectl
```

#### 시간대 설정

```bash
# 현재 시간대 확인
timedatectl

# 서울 시간대로 설정
sudo timedatectl set-timezone Asia/Seoul

# 확인
date
```

#### 필수 패키지 설치

```bash
# 유용한 도구들 설치
sudo apt install -y \
    vim \
    curl \
    wget \
    git \
    htop \
    btop \
    ncdu \
    net-tools \
    tree \
    unzip \
    ca-certificates \
    gnupg \
    lsb-release
```

**각 도구 설명**:
- `vim`: 텍스트 에디터
- `curl`, `wget`: 파일 다운로드
- `git`: 버전 관리
- `htop`, `btop`: 시스템 모니터링 (btop이 더 현대적)
- `ncdu`: 디스크 사용량 분석
- `net-tools`: 네트워크 도구
- `tree`: 디렉토리 구조 출력

## 3.3 SSH 설정과 원격 접속 환경 구축

SSH는 원격에서 서버를 안전하게 관리하기 위한 필수 도구입니다.

### SSH 키 기반 인증 설정

비밀번호 대신 SSH 키를 사용하면 보안성이 높아지고 편리합니다.

#### 클라이언트에서 SSH 키 생성 (한 번만)

**Windows (PowerShell)**:
```powershell
ssh-keygen -t ed25519 -C "your_email@example.com"
# 기본 위치에 저장: C:\Users\YourName\.ssh\id_ed25519
```

**macOS / Linux**:
```bash
ssh-keygen -t ed25519 -C "your_email@example.com"
# 기본 위치에 저장: ~/.ssh/id_ed25519
```

#### 공개 키를 서버에 복사

**방법 1: ssh-copy-id 사용 (macOS/Linux)**
```bash
ssh-copy-id gildong@192.168.0.100
```

**방법 2: 수동 복사 (Windows 등)**
```bash
# 클라이언트에서 공개 키 내용 복사
cat ~/.ssh/id_ed25519.pub

# 서버에 SSH 접속 후
mkdir -p ~/.ssh
chmod 700 ~/.ssh
vim ~/.ssh/authorized_keys
# 복사한 공개 키를 붙여넣기
chmod 600 ~/.ssh/authorized_keys
```

#### 접속 테스트

```bash
ssh gildong@192.168.0.100
# 비밀번호 입력 없이 바로 접속되어야 함
```

### SSH 보안 강화

기본 SSH 설정을 강화합니다.

```bash
# SSH 설정 파일 백업
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.bak

# 설정 파일 편집
sudo vim /etc/ssh/sshd_config
```

**권장 설정**:
```conf
# 포트 변경 (선택사항, 기본 22번 대신 다른 포트 사용)
Port 22

# Root 로그인 금지
PermitRootLogin no

# 비밀번호 인증 비활성화 (SSH 키만 허용)
PasswordAuthentication no

# 빈 비밀번호 금지
PermitEmptyPasswords no

# 공개 키 인증 활성화
PubkeyAuthentication yes

# X11 포워딩 비활성화 (필요 시 yes)
X11Forwarding no

# 로그인 제한 시간
LoginGraceTime 1m

# 최대 인증 시도 횟수
MaxAuthTries 3

# 동시 접속 제한
MaxSessions 10
```

설정 적용:
```bash
# 설정 테스트
sudo sshd -t

# SSH 서비스 재시작
sudo systemctl restart sshd

# ⚠️ 재시작 전에 기존 SSH 세션을 유지한 채로 새 터미널에서 접속 테스트!
```

### SSH 설정 파일로 편리하게 접속

클라이언트(PC/노트북)의 `~/.ssh/config` 파일을 설정하면 간편하게 접속할 수 있습니다.

```conf
Host homeserver
    HostName 192.168.0.100
    User gildong
    Port 22
    IdentityFile ~/.ssh/id_ed25519
    ServerAliveInterval 60
    ServerAliveCountMax 3
```

이제 다음과 같이 간단하게 접속할 수 있습니다:
```bash
ssh homeserver
```

## 3.4 기본 보안 설정

### 방화벽 설정 (UFW)

[UFW](https://help.ubuntu.com/community/UFW) (Uncomplicated Firewall)는 Ubuntu의 기본 방화벽입니다.

```bash
# UFW 설치 확인 (기본 설치되어 있음)
sudo apt install ufw -y

# SSH 포트 허용 (방화벽 활성화 전에 반드시 설정!)
sudo ufw allow 22/tcp
# 또는 특정 IP에서만 SSH 허용
# sudo ufw allow from 192.168.0.0/24 to any port 22

# 방화벽 활성화
sudo ufw enable

# 상태 확인
sudo ufw status verbose
```

**출력 예시**:
```
Status: active
Logging: on (low)
Default: deny (incoming), allow (outgoing), disabled (routed)
New profiles: skip

To                         Action      From
--                         ------      ----
22/tcp                     ALLOW IN    Anywhere
```

### Fail2Ban 설치 (무차별 공격 방어)

[Fail2Ban](https://www.fail2ban.org/)은 로그를 모니터링하여 비정상적인 접속 시도를 차단합니다.

```bash
# Fail2Ban 설치
sudo apt install fail2ban -y

# 설정 파일 생성
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
sudo vim /etc/fail2ban/jail.local
```

**권장 설정** (`/etc/fail2ban/jail.local`):
```ini
[DEFAULT]
bantime = 1h
findtime = 10m
maxretry = 5

[sshd]
enabled = true
port = 22
logpath = /var/log/auth.log
```

**설명**:
- `bantime`: 차단 시간 (1시간)
- `findtime`: 모니터링 시간 (10분)
- `maxretry`: 최대 시도 횟수 (5회)
- 10분 안에 5번 실패하면 1시간 동안 차단

```bash
# Fail2Ban 서비스 시작
sudo systemctl enable fail2ban
sudo systemctl start fail2ban

# 상태 확인
sudo fail2ban-client status sshd
```

### 자동 보안 업데이트 설정

```bash
# Unattended Upgrades 설치
sudo apt install unattended-upgrades -y

# 설정
sudo dpkg-reconfigure -plow unattended-upgrades
# "Yes" 선택

# 설정 파일 확인
sudo vim /etc/apt/apt.conf.d/50unattended-upgrades
```

**권장 설정**:
```conf
Unattended-Upgrade::Allowed-Origins {
    "${distro_id}:${distro_codename}-security";
    "${distro_id}ESMApps:${distro_codename}-apps-security";
    "${distro_id}ESM:${distro_codename}-infra-security";
};

Unattended-Upgrade::AutoFixInterruptedDpkg "true";
Unattended-Upgrade::Remove-Unused-Dependencies "true";
Unattended-Upgrade::Automatic-Reboot "false";
```

### 시스템 계정 보안

```bash
# Root 계정 비밀번호 설정 (비상시 사용)
sudo passwd root

# 일반 사용자를 sudo 그룹에 추가 (이미 되어있을 것)
sudo usermod -aG sudo gildong

# sudo 사용 시 비밀번호 타임아웃 설정
sudo visudo
```

다음 줄을 추가:
```
Defaults    timestamp_timeout=5
```
이제 sudo 사용 후 5분간 비밀번호를 재입력하지 않아도 됩니다.

### 보안 점검 체크리스트

설치 후 다음 사항을 확인하세요:

- [ ] Ubuntu 최신 업데이트 적용
- [ ] SSH 키 기반 인증 설정 완료
- [ ] SSH 비밀번호 로그인 비활성화
- [ ] UFW 방화벽 활성화
- [ ] Fail2Ban 설치 및 작동 확인
- [ ] 자동 보안 업데이트 활성화
- [ ] Root 로그인 비활성화
- [ ] 고정 IP 또는 DHCP 예약 설정 (공유기에서)

---

**다음 장에서는**: Docker와 Docker Compose를 설치하고, 컨테이너 기반의 서비스를 운영하는 방법을 알아보겠습니다.
