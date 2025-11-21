# 제4장: Docker 컨테이너 환경 구축

## 서비스가 늘어나면서 겪는 문제들

### 1단계: 수동 설치의 시작 (서비스 1-3개)

처음에는 간단합니다. 홈서버에 PostgreSQL과 Nginx 정도만 설치했을 때:

```bash
sudo apt install postgresql nginx
```

이 정도는 문제없습니다. 설정 파일 몇 개 수정하고, 서비스 시작하면 끝입니다.

### 2단계: 의존성 충돌 시작 (서비스 5-10개)

프로젝트가 늘어나면서 문제가 시작됩니다:

```bash
# 프로젝트 A는 Node.js 14 필요
sudo apt install nodejs=14.x

# 프로젝트 B는 Node.js 18 필요 (충돌!)
# PostgreSQL 12와 PostgreSQL 15를 동시에 설치할 수 없음
# Redis 6.x와 Redis 7.x 동시 운영 불가
```

**실제로 겪는 문제**:
- Gitea 설치 시 PostgreSQL 15 필요 → 기존 12 버전 업그레이드 → 다른 앱 망가짐
- Node.js 버전 업데이트 → 기존 애플리케이션 작동 중단
- Python 2와 Python 3 공존 문제
- `/etc` 디렉터리 설정 파일이 뒤엉켜서 어떤 게 어떤 서비스 것인지 파악 불가

### 3단계: 관리 불가능한 상태 (서비스 15개+)

```bash
# 설치된 서비스 목록
- PostgreSQL (3개 데이터베이스)
- MySQL (레거시 프로젝트용)
- Nginx (여러 사이트)
- Node.js 앱 5개 (각각 다른 버전 필요)
- Python 앱 3개
- Redis
- Grafana
- Prometheus
```

**재앙의 시작**:

**상황 1: 서버 이전이 필요한 경우**
```
기존 서버 → 새 서버로 이전
- 어떤 서비스가 설치되어 있었는지 문서화 안 됨
- 각 서비스 설정 파일 위치가 제각각 (/etc, /opt, /var 등)
- 수동 재설치 시 3일 소요
- 재설치 중 설정 누락으로 서비스 다운타임 발생
```

**상황 2: 팀원이 로컬에서 동일 환경 구축**
```
팀원: "개발 환경 어떻게 세팅하나요?"
당신: "음... PostgreSQL 12 설치하고, Node.js 14 설치하고..."
팀원: "PostgreSQL 12가 apt에서 안 받아져요"
당신: "아, 저장소 추가해야 하는데..." (30분 소요)
```

**상황 3: 한 서비스 업데이트가 전체 시스템 영향**
```
Nginx 업데이트 → PHP-FPM 의존성 변경 → 기존 웹사이트 3개 다운
Redis 업데이트 → 기존 애플리케이션과 호환 안 됨 → 롤백 시도 → 롤백도 실패
```

### 4단계: Docker 도입 - 모든 문제 해결

Docker를 도입하면 위 모든 문제가 사라집니다.

**격리된 환경**:
```yaml
# PostgreSQL 12와 15 동시 운영
services:
  postgres12:
    image: postgres:12-alpine
    ports: ["5432:5432"]

  postgres15:
    image: postgres:15-alpine
    ports: ["5433:5432"]  # 다른 포트로 분리
```

각 서비스는 완전히 독립적인 환경에서 실행됩니다. 한 컨테이너의 업데이트가 다른 컨테이너에 영향을 주지 않습니다.

**한 줄 배포**:
```bash
# 15개 서비스를 한 번에 시작
docker compose up -d
```

**서버 이전**:
```bash
# 기존 서버에서
tar -czf backup.tar.gz docker-compose.yml volumes/

# 새 서버에서
tar -xzf backup.tar.gz
docker compose up -d  # 완료!
```

3일 걸리던 서버 이전이 30분으로 단축됩니다.

**팀원 온보딩**:
```bash
# 신입 개발자
git clone repo
docker compose up -d
# 끝. 5분 만에 동일한 개발 환경 구축 완료
```

**버전 관리와 롤백**:
```yaml
services:
  myapp:
    image: myapp:1.2.3  # 특정 버전 고정
```

문제 발생 시 이미지 태그만 변경하면 즉시 이전 버전으로 롤백됩니다.

💼 **소규모 조직 관점**:
- **개발자 시간 절약**: 환경 설정에 3일 → 5분
- **장애 복구 시간**: 몇 시간 → 몇 분
- **문서화 부담 감소**: `docker-compose.yml` 파일 자체가 문서
- **신입 온보딩 속도**: 3일 → 30분

**결론**: 홈서버에 서비스 3개 이상 운영한다면 Docker는 선택이 아닌 필수입니다.

---

## 4.1 Docker와 Docker Compose 설치

[Docker](https://www.docker.com/)는 애플리케이션을 컨테이너로 패키징하여 실행하는 플랫폼입니다. 위에서 설명한 모든 문제를 해결해주는 홈서버의 핵심 기술입니다.

### Docker vs Podman

컨테이너 런타임으로는 Docker 외에도 [Podman](https://podman.io/)이 있습니다.

**Docker**:
- 가장 널리 사용되는 컨테이너 플랫폼
- 풍부한 생태계와 문서
- 데몬 기반 아키텍처 (dockerd)

**Podman**:
- Red Hat에서 개발한 Docker 대체재
- 데몬리스 아키텍처 (루트 권한 불필요)
- Docker 명령어 호환 (`alias docker=podman`)
- Kubernetes YAML 지원

💼 **소규모 조직 적용**: 이 책에서는 생태계가 더 성숙한 **Docker**를 기준으로 설명합니다. Podman은 보안 요구사항이 높은 환경에서 고려할 수 있습니다.

### Docker가 홈서버에 적합한 이유

**장점**:
1. **격리된 환경**: 각 서비스가 독립적으로 실행되어 충돌 방지
2. **쉬운 관리**: 설치/업데이트/삭제가 간단
3. **이식성**: 동일한 설정을 다른 서버에서도 사용 가능
4. **버전 관리**: 특정 버전을 고정하거나 쉽게 롤백
5. **리소스 효율**: VM보다 가벼움

### Docker 설치 (Ubuntu 24.04)

공식 Docker 저장소를 사용하여 최신 버전을 설치합니다.

```bash
# 기존 Docker 제거 (있을 경우)
sudo apt remove docker docker-engine docker.io containerd runc

# 필수 패키지 설치
sudo apt update
sudo apt install -y ca-certificates curl gnupg lsb-release

# Docker GPG 키 추가
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | \
  sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

# Docker 저장소 추가
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
  https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Docker 설치
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# 설치 확인
docker --version
docker compose version
```

**출력 예시**:
```
Docker version 24.0.7, build afdd53b
Docker Compose version v2.23.0
```

### Docker 그룹에 사용자 추가

기본적으로 Docker는 root 권한이 필요합니다. 매번 `sudo`를 입력하지 않으려면 사용자를 docker 그룹에 추가합니다.

```bash
# 현재 사용자를 docker 그룹에 추가
sudo usermod -aG docker $USER

# 그룹 변경사항 적용 (로그아웃 후 재로그인 또는)
newgrp docker

# 확인
docker ps
```

### Docker 서비스 자동 시작 설정

```bash
# 부팅 시 자동 시작
sudo systemctl enable docker
sudo systemctl enable containerd

# 서비스 상태 확인
sudo systemctl status docker
```

### Hello World 컨테이너 실행

```bash
# Docker 설치 테스트
docker run hello-world
```

성공하면 다음과 같은 메시지가 출력됩니다:
```
Hello from Docker!
This message shows that your installation appears to be working correctly.
```

## 4.2 Portainer 설치로 GUI 관리 환경 구축

[Portainer](https://www.portainer.io/)는 Docker를 웹 GUI로 관리할 수 있는 도구입니다. CLI가 익숙하지 않은 사용자에게 매우 유용합니다.

### Portainer Community Edition 설치

```bash
# Portainer 데이터 볼륨 생성
docker volume create portainer_data

# Portainer 컨테이너 실행
docker run -d \
  --name=portainer \
  --restart=always \
  -p 9000:9000 \
  -p 9443:9443 \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v portainer_data:/data \
  portainer/portainer-ce:latest
```

**매개변수 설명**:
- `-d`: 백그라운드 실행
- `--name=portainer`: 컨테이너 이름
- `--restart=always`: 서버 재부팅 시 자동 시작
- `-p 9000:9000`: HTTP 포트
- `-p 9443:9443`: HTTPS 포트
- `-v /var/run/docker.sock:/var/run/docker.sock`: Docker 소켓 마운트 (Docker 제어용)
- `-v portainer_data:/data`: 데이터 영속성

### 첫 설정

1. 웹 브라우저에서 `https://192.168.0.100:9443` 접속
2. 관리자 계정 생성
   - Username: admin
   - Password: (최소 12자 이상 강력한 비밀번호)
3. "Get Started" 클릭
4. "Local" 환경 선택

이제 Portainer 대시보드에서 컨테이너, 이미지, 볼륨, 네트워크 등을 GUI로 관리할 수 있습니다.

## 4.3 컨테이너 네트워크 이해와 설정

Docker는 컨테이너 간 통신을 위해 다양한 네트워크 드라이버를 제공합니다.

### Docker 네트워크 종류

#### 1. Bridge (기본값)
- 가장 일반적인 네트워크 모드
- 컨테이너들이 동일한 브리지 네트워크에서 서로 통신 가능
- 호스트와 격리되어 있음

```bash
# 브리지 네트워크 생성
docker network create my-network

# 컨테이너를 특정 네트워크에 연결
docker run -d --name nginx --network my-network nginx
```

#### 2. Host
- 컨테이너가 호스트의 네트워크를 직접 사용
- 포트 매핑 불필요
- 성능이 약간 더 좋지만 격리성 감소

```bash
docker run -d --name nginx --network host nginx
```

#### 3. None
- 네트워크 없음 (완전 격리)

#### 4. Custom Bridge (권장)
사용자 정의 브리지 네트워크를 사용하면 컨테이너 간 이름으로 통신할 수 있습니다 ([DNS 자동 해석](https://docs.docker.com/network/bridge/)).

```bash
# 사용자 정의 네트워크 생성
docker network create homeserver-net

# 컨테이너들을 동일 네트워크에 배치
docker run -d --name db --network homeserver-net postgres
docker run -d --name webapp --network homeserver-net myapp

# webapp에서 db로 연결 시: postgresql://db:5432
```

### 네트워크 관련 명령어

```bash
# 네트워크 목록
docker network ls

# 네트워크 상세 정보
docker network inspect homeserver-net

# 실행 중인 컨테이너를 네트워크에 연결
docker network connect homeserver-net mycontainer

# 네트워크에서 분리
docker network disconnect homeserver-net mycontainer

# 사용하지 않는 네트워크 삭제
docker network prune
```

## 4.4 볼륨 관리와 데이터 지속성

컨테이너는 기본적으로 상태가 없습니다 (stateless). 컨테이너를 삭제하면 내부 데이터도 함께 사라집니다. 데이터를 영구적으로 보존하려면 **볼륨**을 사용해야 합니다.

### 볼륨 종류

#### 1. Named Volume (권장)
Docker가 관리하는 볼륨입니다.

```bash
# 볼륨 생성
docker volume create my-data

# 컨테이너 실행 시 볼륨 마운트
docker run -d --name db -v my-data:/var/lib/postgresql/data postgres

# 볼륨 목록
docker volume ls

# 볼륨 상세 정보 (실제 저장 위치 확인)
docker volume inspect my-data
```

**장점**:
- Docker가 자동으로 관리
- 백업과 마이그레이션이 쉬움
- 플랫폼 독립적

#### 2. Bind Mount
호스트의 특정 경로를 컨테이너에 직접 마운트합니다.

```bash
# 호스트의 /home/gildong/data를 컨테이너의 /app/data로 마운트
docker run -d --name app -v /home/gildong/data:/app/data myapp
```

**장점**:
- 호스트에서 직접 파일 접근 가능
- 개발 시 코드 실시간 반영

**단점**:
- 경로 의존성 (이식성 낮음)
- 권한 문제 발생 가능

### 볼륨 관련 명령어

```bash
# 볼륨 생성
docker volume create my-volume

# 볼륨 목록
docker volume ls

# 볼륨 상세 정보
docker volume inspect my-volume

# 사용하지 않는 볼륨 삭제
docker volume prune

# 특정 볼륨 삭제
docker volume rm my-volume
```

### 실전 예제: PostgreSQL with Volume

```bash
# PostgreSQL 볼륨 생성
docker volume create postgres-data

# PostgreSQL 컨테이너 실행
docker run -d \
  --name postgres \
  --restart always \
  -e POSTGRES_PASSWORD=mysecretpassword \
  -e POSTGRES_DB=mydb \
  -v postgres-data:/var/lib/postgresql/data \
  -p 5432:5432 \
  postgres:16-alpine

# 컨테이너를 삭제해도 데이터는 보존됨
docker rm -f postgres

# 동일한 볼륨으로 다시 실행하면 데이터 유지
docker run -d \
  --name postgres \
  -e POSTGRES_PASSWORD=mysecretpassword \
  -v postgres-data:/var/lib/postgresql/data \
  -p 5432:5432 \
  postgres:16-alpine
```

## 4.5 컨테이너 자동 업데이트 설정 (Watchtower)

[Watchtower](https://containrrr.dev/watchtower/)는 Docker 컨테이너를 자동으로 업데이트해주는 도구입니다.

### Watchtower 설치

```bash
docker run -d \
  --name watchtower \
  --restart always \
  -v /var/run/docker.sock:/var/run/docker.sock \
  containrrr/watchtower \
  --cleanup \
  --interval 86400
```

**매개변수 설명**:
- `--cleanup`: 이전 이미지 자동 삭제
- `--interval 86400`: 24시간마다 체크 (초 단위)

### 특정 컨테이너만 자동 업데이트

```bash
# Watchtower가 특정 컨테이너만 업데이트하도록 설정
docker run -d \
  --name watchtower \
  --restart always \
  -v /var/run/docker.sock:/var/run/docker.sock \
  containrrr/watchtower \
  --cleanup \
  --interval 86400 \
  nginx postgres
```

### 특정 컨테이너 제외

레이블을 사용하여 특정 컨테이너를 자동 업데이트에서 제외할 수 있습니다.

```bash
# 자동 업데이트 제외
docker run -d \
  --name critical-app \
  --label com.centurylinklabs.watchtower.enable=false \
  myapp
```

### Docker Compose에서 사용

나중에 Docker Compose를 사용할 때는 다음과 같이 설정합니다:

```yaml
version: '3.8'

services:
  watchtower:
    image: containrrr/watchtower
    container_name: watchtower
    restart: always
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    environment:
      - WATCHTOWER_CLEANUP=true
      - WATCHTOWER_POLL_INTERVAL=86400
```

### 주의사항

⚠️ **중요**: Watchtower는 편리하지만 다음을 주의하세요:
- 예상치 못한 업데이트로 서비스가 중단될 수 있음
- 중요한 서비스는 수동 업데이트 권장
- 업데이트 전 백업 필수

---

## Docker Compose 소개

여러 컨테이너를 함께 관리하려면 Docker Compose를 사용합니다. YAML 파일로 서비스를 정의하고 한 번에 시작/종료할 수 있습니다.

### Docker Compose 예제: WordPress + MySQL

`docker-compose.yml` 파일 생성:

```yaml
version: '3.8'

services:
  db:
    image: mysql:8.0
    container_name: wordpress-db
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: rootpassword
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wpuser
      MYSQL_PASSWORD: wppassword
    volumes:
      - db-data:/var/lib/mysql
    networks:
      - wp-network

  wordpress:
    image: wordpress:latest
    container_name: wordpress
    restart: always
    depends_on:
      - db
    ports:
      - "8080:80"
    environment:
      WORDPRESS_DB_HOST: db:3306
      WORDPRESS_DB_USER: wpuser
      WORDPRESS_DB_PASSWORD: wppassword
      WORDPRESS_DB_NAME: wordpress
    volumes:
      - wp-data:/var/www/html
    networks:
      - wp-network

volumes:
  db-data:
  wp-data:

networks:
  wp-network:
    driver: bridge
```

### Docker Compose 명령어

```bash
# 서비스 시작 (백그라운드)
docker compose up -d

# 로그 확인
docker compose logs -f

# 서비스 중지
docker compose stop

# 서비스 중지 및 제거 (볼륨은 유지)
docker compose down

# 서비스 중지 및 제거 (볼륨도 삭제)
docker compose down -v

# 서비스 재시작
docker compose restart

# 특정 서비스만 재시작
docker compose restart wordpress
```

---

**다음 장에서는**: Nginx Proxy Manager를 설치하여 리버스 프록시와 SSL 인증서를 설정하고, 도메인으로 서비스에 접근하는 방법을 알아보겠습니다.
