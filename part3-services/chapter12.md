# 제12장: 개발 환경 구축

## 12.1 Git 서버 (Gitea)

[Gitea](https://gitea.io/)는 경량 Git 호스팅 서버입니다.

```yaml
version: '3.8'

services:
  gitea-db:
    image: postgres:16-alpine
    container_name: gitea-db
    restart: unless-stopped
    environment:
      - POSTGRES_USER=gitea
      - POSTGRES_PASSWORD=gitea
      - POSTGRES_DB=gitea
    volumes:
      - gitea-db:/var/lib/postgresql/data

  gitea:
    image: gitea/gitea:latest
    container_name: gitea
    restart: unless-stopped
    environment:
      - USER_UID=1000
      - USER_GID=1000
      - GITEA__database__DB_TYPE=postgres
      - GITEA__database__HOST=gitea-db:5432
      - GITEA__database__NAME=gitea
      - GITEA__database__USER=gitea
      - GITEA__database__PASSWD=gitea
    volumes:
      - ./gitea:/data
      - /etc/timezone:/etc/timezone:ro
      - /etc/localtime:/etc/localtime:ro
    ports:
      - "3002:3000"
      - "2222:22"
    depends_on:
      - gitea-db

volumes:
  gitea-db:
```

웹 UI: `http://homeserver:3002`

### 첫 저장소 생성

1. 회원가입 (첫 사용자가 관리자)
2. **+** → **New Repository**
3. Repository name 입력
4. **Create Repository**

### Git 클라이언트 설정

```bash
# SSH 키 추가 (Gitea 웹에서)
cat ~/.ssh/id_ed25519.pub

# 저장소 클론
git clone ssh://git@homeserver:2222/username/repo.git

# 또는 HTTP
git clone http://homeserver:3002/username/repo.git
```

## 12.2 CI/CD 파이프라인 (Drone CI)

[Drone CI](https://www.drone.io/)는 컨테이너 기반 CI/CD 도구입니다.

```yaml
version: '3.8'

services:
  drone-server:
    image: drone/drone:latest
    container_name: drone-server
    restart: unless-stopped
    environment:
      - DRONE_GITEA_SERVER=http://homeserver:3002
      - DRONE_GITEA_CLIENT_ID=your_oauth_client_id
      - DRONE_GITEA_CLIENT_SECRET=your_oauth_client_secret
      - DRONE_RPC_SECRET=super_secret_rpc_key
      - DRONE_SERVER_HOST=drone.homelab.local
      - DRONE_SERVER_PROTO=http
      - DRONE_USER_CREATE=username:gildong,admin:true
    ports:
      - "8086:80"
    volumes:
      - ./drone:/data

  drone-runner:
    image: drone/drone-runner-docker:latest
    container_name: drone-runner
    restart: unless-stopped
    environment:
      - DRONE_RPC_PROTO=http
      - DRONE_RPC_HOST=drone-server
      - DRONE_RPC_SECRET=super_secret_rpc_key
      - DRONE_RUNNER_CAPACITY=2
      - DRONE_RUNNER_NAME=homeserver-runner
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    depends_on:
      - drone-server
```

### Gitea OAuth 앱 생성

1. Gitea → Settings → Applications → **Create OAuth2 Application**
2. Application Name: `Drone CI`
3. Redirect URI: `http://drone.homelab.local/login`
4. Client ID와 Secret 복사 → Drone 환경변수에 입력

### .drone.yml 파이프라인 예제

**.NET 프로젝트 빌드 및 테스트**:

```yaml
kind: pipeline
type: docker
name: default

steps:
  - name: restore
    image: mcr.microsoft.com/dotnet/sdk:8.0
    commands:
      - dotnet restore

  - name: build
    image: mcr.microsoft.com/dotnet/sdk:8.0
    commands:
      - dotnet build --no-restore -c Release

  - name: test
    image: mcr.microsoft.com/dotnet/sdk:8.0
    commands:
      - dotnet test --no-build -c Release

  - name: publish
    image: mcr.microsoft.com/dotnet/sdk:8.0
    commands:
      - dotnet publish -c Release -o ./publish
    when:
      branch:
        - main

  - name: docker-build
    image: plugins/docker
    settings:
      repo: homeserver:5000/myapp
      registry: homeserver:5000
      tags:
        - latest
        - ${DRONE_COMMIT_SHA:0:8}
      insecure: true
    when:
      branch:
        - main
```

## 12.3 코드 서버 (code-server)

[code-server](https://github.com/coder/code-server)는 웹 기반 VS Code입니다.

```yaml
version: '3.8'

services:
  code-server:
    image: lscr.io/linuxserver/code-server:latest
    container_name: code-server
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Asia/Seoul
      - PASSWORD=your_secure_password
      - SUDO_PASSWORD=your_sudo_password
      - DEFAULT_WORKSPACE=/config/workspace
    volumes:
      - ./code-server/config:/config
      - ./projects:/config/workspace/projects
    ports:
      - 8443:8443
    restart: unless-stopped
```

웹 IDE: `http://homeserver:8443`

### 유용한 확장 프로그램

- C# Dev Kit
- Docker
- GitLens
- ESLint / Prettier
- Remote - SSH

## 12.4 데이터베이스 관리

### PostgreSQL/MySQL 설치

**PostgreSQL**:

```yaml
version: '3.8'

services:
  postgres:
    image: postgres:16-alpine
    container_name: postgres
    restart: unless-stopped
    environment:
      - POSTGRES_USER=admin
      - POSTGRES_PASSWORD=admin_password
      - POSTGRES_DB=default_db
    volumes:
      - postgres-data:/var/lib/postgresql/data
    ports:
      - "5432:5432"

volumes:
  postgres-data:
```

**MySQL**:

```yaml
version: '3.8'

services:
  mysql:
    image: mysql:8.0
    container_name: mysql
    restart: unless-stopped
    environment:
      - MYSQL_ROOT_PASSWORD=root_password
      - MYSQL_DATABASE=default_db
      - MYSQL_USER=admin
      - MYSQL_PASSWORD=admin_password
    volumes:
      - mysql-data:/var/lib/mysql
    ports:
      - "3306:3306"
    command: --default-authentication-plugin=mysql_native_password

volumes:
  mysql-data:
```

### Adminer 웹 관리 도구

[Adminer](https://www.adminer.org/)는 경량 DB 관리 도구입니다.

```yaml
version: '3.8'

services:
  adminer:
    image: adminer:latest
    container_name: adminer
    restart: unless-stopped
    ports:
      - 8085:8080
    environment:
      - ADMINER_DEFAULT_SERVER=postgres
```

웹 UI: `http://homeserver:8085`

로그인:
- System: PostgreSQL / MySQL
- Server: `postgres` / `mysql`
- Username: `admin`
- Password: 설정한 비밀번호

### pgAdmin (PostgreSQL 전용)

더 강력한 PostgreSQL 관리 도구:

```yaml
version: '3.8'

services:
  pgadmin:
    image: dpage/pgadmin4:latest
    container_name: pgadmin
    restart: unless-stopped
    environment:
      - PGADMIN_DEFAULT_EMAIL=admin@homelab.local
      - PGADMIN_DEFAULT_PASSWORD=admin_password
    ports:
      - "5050:80"
    volumes:
      - pgadmin-data:/var/lib/pgadmin

volumes:
  pgadmin-data:
```

웹 UI: `http://homeserver:5050`

---

**다음 장에서는**: .NET 웹 애플리케이션을 개발하고 홈서버에 배포하는 전 과정을 상세히 알아보겠습니다 (이미 제13장에서 다뤘습니다).
