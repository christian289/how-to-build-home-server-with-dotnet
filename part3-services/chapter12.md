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

## 12.2 CI/CD 파이프라인 (Gitea Actions)

[Gitea Actions](https://docs.gitea.com/usage/actions/overview)는 Gitea 1.19+에 내장된 CI/CD 시스템으로, GitHub Actions와 호환됩니다.

### Gitea Actions 활성화

Gitea `app.ini` 설정 파일 수정 (또는 환경변수):

```yaml
version: '3.8'

services:
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
      - GITEA__actions__ENABLED=true
    volumes:
      - ./gitea:/data
      - /etc/timezone:/etc/timezone:ro
      - /etc/localtime:/etc/localtime:ro
    ports:
      - "3002:3000"
      - "2222:22"
    depends_on:
      - gitea-db
```

### Act Runner 설치 (Gitea Actions 실행기)

```yaml
  act-runner:
    image: gitea/act_runner:latest
    container_name: act-runner
    restart: unless-stopped
    environment:
      - GITEA_INSTANCE_URL=http://gitea:3000
      - GITEA_RUNNER_REGISTRATION_TOKEN=your_registration_token
      - GITEA_RUNNER_NAME=homeserver-runner
    volumes:
      - ./act-runner:/data
      - /var/run/docker.sock:/var/run/docker.sock
    depends_on:
      - gitea
```

### Runner 등록 토큰 생성

1. Gitea 웹 UI → **Site Administration** → **Actions** → **Runners**
2. **Create new Runner** 클릭
3. Registration Token 복사 → `act-runner` 환경변수에 입력

### .gitea/workflows/build.yml 파이프라인 예제

**.NET 프로젝트 빌드 및 테스트** (GitHub Actions 호환 문법):

```yaml
name: .NET Build and Test

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup .NET
        uses: actions/setup-dotnet@v4
        with:
          dotnet-version: '8.0.x'

      - name: Restore dependencies
        run: dotnet restore

      - name: Build
        run: dotnet build --no-restore -c Release

      - name: Test
        run: dotnet test --no-build -c Release --verbosity normal

      - name: Publish
        if: github.ref == 'refs/heads/main'
        run: dotnet publish -c Release -o ./publish

      - name: Build Docker image
        if: github.ref == 'refs/heads/main'
        run: |
          docker build -t homeserver:5000/myapp:latest \
                       -t homeserver:5000/myapp:${{ github.sha }} .
          docker push homeserver:5000/myapp:latest
          docker push homeserver:5000/myapp:${{ github.sha }}
```

💼 **소규모 조직 적용**: Gitea Actions는 별도 서비스 없이 Gitea에 내장되어 있어 관리 부담이 적습니다. GitHub Actions와 동일한 문법을 사용하므로 개발자 학습 비용도 낮습니다.

### 다른 CI/CD 옵션

**[Jenkins](https://www.jenkins.io/)** (전통적인 선택):
- 가장 많이 사용되는 오픈소스 CI/CD
- 플러그인 생태계가 풍부
- 설정이 복잡할 수 있음

**[Woodpecker CI](https://woodpecker-ci.org/)** (경량 대안):
- Drone CI의 오픈소스 포크
- Gitea 네이티브 지원
- 간단한 YAML 설정

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

### phpMyAdmin (MySQL/MariaDB 전용)

[phpMyAdmin](https://www.phpmyadmin.net/)은 MySQL/MariaDB 웹 관리 도구입니다.

```yaml
version: '3.8'

services:
  phpmyadmin:
    image: phpmyadmin:latest
    container_name: phpmyadmin
    restart: unless-stopped
    ports:
      - 8082:80
    environment:
      - PMA_HOST=mysql
      - PMA_PORT=3306
      - UPLOAD_LIMIT=50M
```

웹 UI: `http://homeserver:8082`

## 12.5 Docker 레지스트리 (Harbor)

[Harbor](https://goharbor.io/)는 엔터프라이즈급 Docker 레지스트리입니다.

```yaml
version: '3.8'

services:
  harbor-db:
    image: goharbor/harbor-db:v2.10.0
    container_name: harbor-db
    restart: unless-stopped
    environment:
      - POSTGRES_PASSWORD=harbor_password
      - POSTGRES_USER=postgres
      - POSTGRES_DB=registry
    volumes:
      - harbor-db:/var/lib/postgresql/data

  harbor-core:
    image: goharbor/harbor-core:v2.10.0
    container_name: harbor-core
    restart: unless-stopped
    environment:
      - CORE_SECRET=change_this_secret
      - JOBSERVICE_SECRET=change_this_secret
    depends_on:
      - harbor-db
    volumes:
      - harbor-data:/data

  harbor-registry:
    image: goharbor/registry-photon:v2.10.0
    container_name: harbor-registry
    restart: unless-stopped
    volumes:
      - harbor-registry:/storage

  harbor-portal:
    image: goharbor/harbor-portal:v2.10.0
    container_name: harbor-portal
    restart: unless-stopped
    ports:
      - "8088:8080"
    depends_on:
      - harbor-core

volumes:
  harbor-db:
  harbor-data:
  harbor-registry:
```

기본 로그인: `admin` / `Harbor12345`

💼 **소규모 조직 적용**: Harbor는 Docker 이미지 스캔, 권한 관리, 복제 기능을 제공합니다. 팀 내 Docker 이미지를 중앙에서 관리하고 보안 취약점을 검사할 수 있습니다.

## 12.6 애플리케이션 모니터링 (Sentry)

[Sentry](https://sentry.io/)는 오픈소스 애플리케이션 에러 추적 시스템입니다.

```yaml
version: '3.8'

services:
  sentry-redis:
    image: redis:7-alpine
    container_name: sentry-redis
    restart: unless-stopped
    volumes:
      - sentry-redis:/data

  sentry-postgres:
    image: postgres:16-alpine
    container_name: sentry-postgres
    restart: unless-stopped
    environment:
      - POSTGRES_USER=sentry
      - POSTGRES_PASSWORD=sentry
      - POSTGRES_DB=sentry
    volumes:
      - sentry-postgres:/var/lib/postgresql/data

  sentry:
    image: sentry:latest
    container_name: sentry
    restart: unless-stopped
    ports:
      - "9090:9000"
    environment:
      - SENTRY_SECRET_KEY=your_secret_key_here
      - SENTRY_POSTGRES_HOST=sentry-postgres
      - SENTRY_DB_USER=sentry
      - SENTRY_DB_PASSWORD=sentry
      - SENTRY_REDIS_HOST=sentry-redis
    depends_on:
      - sentry-redis
      - sentry-postgres
    volumes:
      - sentry-data:/var/lib/sentry/files

  sentry-cron:
    image: sentry:latest
    container_name: sentry-cron
    restart: unless-stopped
    command: run cron
    environment:
      - SENTRY_SECRET_KEY=your_secret_key_here
      - SENTRY_POSTGRES_HOST=sentry-postgres
      - SENTRY_DB_USER=sentry
      - SENTRY_DB_PASSWORD=sentry
      - SENTRY_REDIS_HOST=sentry-redis
    depends_on:
      - sentry

  sentry-worker:
    image: sentry:latest
    container_name: sentry-worker
    restart: unless-stopped
    command: run worker
    environment:
      - SENTRY_SECRET_KEY=your_secret_key_here
      - SENTRY_POSTGRES_HOST=sentry-postgres
      - SENTRY_DB_USER=sentry
      - SENTRY_DB_PASSWORD=sentry
      - SENTRY_REDIS_HOST=sentry-redis
    depends_on:
      - sentry

volumes:
  sentry-redis:
  sentry-postgres:
  sentry-data:
```

웹 UI: `http://homeserver:9090`

### .NET 애플리케이션에서 Sentry 연동

```bash
dotnet add package Sentry.AspNetCore
```

`Program.cs`:

```csharp
var builder = WebApplication.CreateBuilder(args);

// Sentry 초기화
builder.WebHost.UseSentry(o =>
{
    o.Dsn = "http://your_sentry_key@homeserver:9090/1";
    o.TracesSampleRate = 1.0;
    o.Environment = builder.Environment.EnvironmentName;
});

var app = builder.Build();
app.Run();
```

💼 **소규모 조직 적용**:
- **SaaS 대비 비용 절감**: Sentry Cloud는 월 $26부터 시작하지만, 셀프 호스팅은 무료
- 운영 중인 .NET 애플리케이션의 에러를 실시간으로 추적
- Slack/Discord 연동으로 에러 발생 시 즉시 알림

---

**다음 장에서는**: .NET 웹 애플리케이션을 개발하고 홈서버에 배포하는 전 과정을 상세히 알아보겠습니다 (이미 제13장에서 다뤘습니다).
