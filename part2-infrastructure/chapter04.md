# 제4장: Docker 컨테이너 환경 구축

## 4.1 Docker와 Docker Compose 설치

[Docker](https://www.docker.com/)는 애플리케이션을 컨테이너로 패키징하여 실행하는 플랫폼입니다. 홈서버의 핵심 기술이며, 대부분의 서비스를 Docker로 운영하게 됩니다.

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
