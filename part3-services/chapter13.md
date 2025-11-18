# 제13장: 웹 애플리케이션 개발과 배포

이 장에서는 홈서버에서 .NET 애플리케이션을 개발하고 배포하는 전 과정을 다룹니다. 로컬 개발 환경부터 프로덕션 배포, 모니터링까지 실전에서 바로 사용할 수 있는 내용을 담았습니다.

## 13.1 .NET 로컬 개발 환경 구성

### .NET SDK 설치와 버전 관리

홈서버에 [.NET SDK](https://dotnet.microsoft.com/download)를 설치합니다.

#### Ubuntu에 .NET 8.0 SDK 설치

```bash
# Microsoft 패키지 저장소 추가
wget https://packages.microsoft.com/config/ubuntu/$(lsb_release -rs)/packages-microsoft-prod.deb -O packages-microsoft-prod.deb
sudo dpkg -i packages-microsoft-prod.deb
rm packages-microsoft-prod.deb

# .NET SDK 설치
sudo apt update
sudo apt install -y dotnet-sdk-8.0

# 설치 확인
dotnet --version
```

**출력 예시**:
```
8.0.100
```

#### 여러 .NET 버전 관리

여러 프로젝트에서 다른 .NET 버전을 사용할 경우:

```bash
# 설치된 .NET SDK 목록
dotnet --list-sdks

# 특정 프로젝트에 .NET 버전 지정
# global.json 파일 생성
cd ~/myproject
dotnet new globaljson --sdk-version 8.0.100
```

`global.json` 내용:
```json
{
  "sdk": {
    "version": "8.0.100",
    "rollForward": "latestMinor"
  }
}
```

### Visual Studio Code + Dev Containers

#### VS Code Server 설치 (선택사항)

원격으로 코딩하려면 [code-server](https://github.com/coder/code-server)를 사용할 수 있습니다.

```yaml
# docker-compose.yml
version: '3.8'

services:
  code-server:
    image: lscr.io/linuxserver/code-server:latest
    container_name: code-server
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Asia/Seoul
      - PASSWORD=yourpassword
      - SUDO_PASSWORD=yoursudopassword
    volumes:
      - ./config:/config
      - ./projects:/config/workspace
    ports:
      - 8443:8443
    restart: unless-stopped
```

```bash
docker compose up -d
```

이제 `http://homeserver:8443`으로 접속하여 웹 브라우저에서 코딩할 수 있습니다.

#### Dev Containers 설정

`.devcontainer/devcontainer.json`:

```json
{
  "name": ".NET 8",
  "image": "mcr.microsoft.com/devcontainers/dotnet:8.0",
  "customizations": {
    "vscode": {
      "extensions": [
        "ms-dotnettools.csharp",
        "ms-dotnettools.csdevkit"
      ]
    }
  },
  "forwardPorts": [5000, 5001],
  "postCreateCommand": "dotnet restore",
  "remoteUser": "vscode"
}
```

### Docker Compose로 의존성 서비스 구축

개발 시 필요한 데이터베이스, Redis 등을 Docker Compose로 관리합니다.

`docker-compose.dev.yml`:

```yaml
version: '3.8'

services:
  postgres:
    image: postgres:16-alpine
    container_name: dev-postgres
    environment:
      POSTGRES_USER: devuser
      POSTGRES_PASSWORD: devpassword
      POSTGRES_DB: myapp_dev
    ports:
      - "5432:5432"
    volumes:
      - postgres-data:/var/lib/postgresql/data

  redis:
    image: redis:7-alpine
    container_name: dev-redis
    ports:
      - "6379:6379"
    volumes:
      - redis-data:/data

  mailhog:
    image: mailhog/mailhog
    container_name: dev-mailhog
    ports:
      - "1025:1025"  # SMTP
      - "8025:8025"  # Web UI

volumes:
  postgres-data:
  redis-data:
```

```bash
# 개발 환경 시작
docker compose -f docker-compose.dev.yml up -d

# 중지
docker compose -f docker-compose.dev.yml down
```

### Hot Reload와 Watch Mode

.NET 8은 Hot Reload 기능을 지원합니다.

```bash
# 프로젝트 생성
dotnet new webapi -n MyApi
cd MyApi

# Watch 모드로 실행 (코드 변경 시 자동 재시작)
dotnet watch run

# Hot Reload 활성화
dotnet watch run --no-hot-reload false
```

### 개발용 HTTPS 인증서 (dotnet dev-certs)

```bash
# HTTPS 개발 인증서 생성
dotnet dev-certs https --trust

# 인증서 확인
dotnet dev-certs https --check

# 기존 인증서 제거 후 재생성
dotnet dev-certs https --clean
dotnet dev-certs https --trust
```

이제 `https://localhost:5001`로 접속 시 브라우저 경고 없이 접속할 수 있습니다.

## 13.2 .NET 애플리케이션 배포 전략

### Docker 컨테이너 배포

#### Dockerfile 작성

`Dockerfile`:

```dockerfile
# Build stage
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
WORKDIR /src

# 프로젝트 파일 복사 및 복원
COPY ["MyApi/MyApi.csproj", "MyApi/"]
RUN dotnet restore "MyApi/MyApi.csproj"

# 소스 코드 복사 및 빌드
COPY . .
WORKDIR "/src/MyApi"
RUN dotnet build "MyApi.csproj" -c Release -o /app/build

# Publish stage
FROM build AS publish
RUN dotnet publish "MyApi.csproj" -c Release -o /app/publish /p:UseAppHost=false

# Runtime stage
FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS final
WORKDIR /app
EXPOSE 8080
EXPOSE 8081

# 비root 사용자로 실행 (보안)
USER $APP_UID

COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "MyApi.dll"]
```

#### 최적화된 Dockerfile (멀티 스테이지 빌드)

```dockerfile
FROM mcr.microsoft.com/dotnet/sdk:8.0-alpine AS build
WORKDIR /src

# 캐시 활용을 위해 csproj만 먼저 복사
COPY ["MyApi.csproj", "./"]
RUN dotnet restore "MyApi.csproj"

# 소스 코드 복사 및 빌드
COPY . .
RUN dotnet publish "MyApi.csproj" \
    -c Release \
    -o /app/publish \
    --no-restore \
    /p:PublishTrimmed=false

# Runtime (Alpine으로 최소화)
FROM mcr.microsoft.com/dotnet/aspnet:8.0-alpine AS final
WORKDIR /app

# 시간대 설정
ENV TZ=Asia/Seoul
RUN apk add --no-cache tzdata

COPY --from=build /app/publish .

EXPOSE 8080
ENTRYPOINT ["dotnet", "MyApi.dll"]
```

#### Docker Compose로 배포

`docker-compose.yml`:

```yaml
version: '3.8'

services:
  myapi:
    build:
      context: .
      dockerfile: Dockerfile
    container_name: myapi
    restart: always
    ports:
      - "8080:8080"
    environment:
      - ASPNETCORE_ENVIRONMENT=Production
      - ASPNETCORE_URLS=http://+:8080
      - ConnectionStrings__DefaultConnection=Host=postgres;Database=myapp;Username=appuser;Password=apppassword
      - Redis__ConnectionString=redis:6379
    depends_on:
      - postgres
      - redis
    networks:
      - app-network

  postgres:
    image: postgres:16-alpine
    container_name: myapi-postgres
    restart: always
    environment:
      POSTGRES_USER: appuser
      POSTGRES_PASSWORD: apppassword
      POSTGRES_DB: myapp
    volumes:
      - postgres-data:/var/lib/postgresql/data
    networks:
      - app-network

  redis:
    image: redis:7-alpine
    container_name: myapi-redis
    restart: always
    volumes:
      - redis-data:/data
    networks:
      - app-network

volumes:
  postgres-data:
  redis-data:

networks:
  app-network:
    driver: bridge
```

```bash
# 빌드 및 실행
docker compose up -d --build

# 로그 확인
docker compose logs -f myapi

# 중지
docker compose down
```

### IIS on Linux (Kestrel + Nginx)

.NET은 [Kestrel](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/servers/kestrel) 웹 서버를 내장하고 있으며, Nginx를 리버스 프록시로 사용하여 프로덕션 배포가 가능합니다.

#### systemd 서비스로 등록

1. 앱 게시:

```bash
dotnet publish -c Release -o /var/www/myapi
```

2. systemd 서비스 파일 생성:

`/etc/systemd/system/myapi.service`:

```ini
[Unit]
Description=My ASP.NET Core API
After=network.target

[Service]
Type=notify
User=www-data
WorkingDirectory=/var/www/myapi
ExecStart=/usr/bin/dotnet /var/www/myapi/MyApi.dll
Restart=always
RestartSec=10
KillSignal=SIGINT
SyslogIdentifier=myapi
Environment=ASPNETCORE_ENVIRONMENT=Production
Environment=DOTNET_PRINT_TELEMETRY_MESSAGE=false
Environment=ASPNETCORE_URLS=http://localhost:5000

[Install]
WantedBy=multi-user.target
```

3. 서비스 시작:

```bash
sudo systemctl daemon-reload
sudo systemctl enable myapi
sudo systemctl start myapi
sudo systemctl status myapi
```

4. Nginx 리버스 프록시 설정:

`/etc/nginx/sites-available/myapi`:

```nginx
server {
    listen 80;
    server_name api.example.com;

    location / {
        proxy_pass http://localhost:5000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection keep-alive;
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

```bash
sudo ln -s /etc/nginx/sites-available/myapi /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

### .NET Runtime vs Self-Contained 배포

#### Framework-Dependent Deployment (FDD)
```bash
dotnet publish -c Release -o ./publish
```
- 서버에 .NET Runtime 설치 필요
- 패키지 크기 작음 (~50MB)
- 런타임 업데이트 자동 적용

#### Self-Contained Deployment (SCD)
```bash
dotnet publish -c Release -r linux-x64 --self-contained true -o ./publish
```
- .NET Runtime 포함
- 패키지 크기 큼 (~80-100MB)
- 런타임 버전 고정

#### 권장 전략
- **Docker 배포**: FDD 사용 (베이스 이미지에 런타임 포함)
- **직접 배포**: SCD 사용 (의존성 최소화)

### Native AOT 최적화 배포

.NET 8은 [Native AOT](https://learn.microsoft.com/en-us/dotnet/core/deploying/native-aot/) (Ahead-of-Time) 컴파일을 지원하여 시작 시간과 메모리 사용량을 대폭 줄일 수 있습니다.

`MyApi.csproj`:

```xml
<Project Sdk="Microsoft.NET.Sdk.Web">
  <PropertyGroup>
    <TargetFramework>net8.0</TargetFramework>
    <PublishAot>true</PublishAot>
    <InvariantGlobalization>true</InvariantGlobalization>
  </PropertyGroup>
</Project>
```

```bash
dotnet publish -c Release -r linux-x64 -o ./publish
```

**Native AOT 장점**:
- 빠른 시작 시간 (~10ms)
- 낮은 메모리 사용량 (~10MB)
- 단일 실행 파일

**제약사항**:
- Reflection 제한
- 일부 라이브러리 미지원
- Minimal API 권장

### Health Checks와 Readiness Probes

`Program.cs`:

```csharp
var builder = WebApplication.CreateBuilder(args);

// Health Checks 추가
builder.Services.AddHealthChecks()
    .AddNpgSql(builder.Configuration.GetConnectionString("DefaultConnection")!)
    .AddRedis(builder.Configuration["Redis:ConnectionString"]!);

var app = builder.Build();

// Health Check 엔드포인트
app.MapHealthChecks("/health");
app.MapHealthChecks("/health/ready", new HealthCheckOptions
{
    Predicate = check => check.Tags.Contains("ready")
});
app.MapHealthChecks("/health/live", new HealthCheckOptions
{
    Predicate = _ => false
});

app.Run();
```

Docker Compose에서 Health Check 사용:

```yaml
services:
  myapi:
    # ...
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8080/health"]
      interval: 30s
      timeout: 3s
      retries: 3
      start_period: 40s
```

### 환경별 Configuration 관리

`appsettings.json`:
```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information"
    }
  }
}
```

`appsettings.Development.json`:
```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Debug"
    }
  },
  "ConnectionStrings": {
    "DefaultConnection": "Host=localhost;Database=myapp_dev;Username=devuser;Password=devpassword"
  }
}
```

`appsettings.Production.json`:
```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Warning"
    }
  },
  "ConnectionStrings": {
    "DefaultConnection": "Host=postgres;Database=myapp;Username=appuser;Password=${DB_PASSWORD}"
  }
}
```

환경 변수로 설정 오버라이드:
```bash
export ASPNETCORE_ENVIRONMENT=Production
export ConnectionStrings__DefaultConnection="Host=prod-db;..."
dotnet MyApi.dll
```

## 13.3 외부 접속 설정

### 포트 포워딩 설정

공유기 관리 페이지에서:
1. 외부 포트: 80
2. 내부 IP: 192.168.0.100
3. 내부 포트: 80
4. 프로토콜: TCP

### Cloudflare Tunnel 활용

[Cloudflare Tunnel](https://www.cloudflare.com/products/tunnel/)을 사용하면 포트 포워딩 없이 외부 접속이 가능합니다.

```bash
# Cloudflare Tunnel (cloudflared) 설치
wget -q https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64.deb
sudo dpkg -i cloudflared-linux-amd64.deb

# 인증
cloudflared tunnel login

# 터널 생성
cloudflared tunnel create homeserver

# 설정 파일 생성
mkdir -p ~/.cloudflared
cat > ~/.cloudflared/config.yml <<EOF
tunnel: <TUNNEL-ID>
credentials-file: /home/gildong/.cloudflared/<TUNNEL-ID>.json

ingress:
  - hostname: api.example.com
    service: http://localhost:8080
  - service: http_status:404
EOF

# 터널 라우팅 설정
cloudflared tunnel route dns homeserver api.example.com

# 터널 실행
cloudflared tunnel run homeserver
```

Docker Compose로 관리:

```yaml
services:
  cloudflared:
    image: cloudflare/cloudflared:latest
    container_name: cloudflared
    restart: always
    command: tunnel --no-autoupdate run
    environment:
      - TUNNEL_TOKEN=${TUNNEL_TOKEN}
```

### Ngrok 대안 (Localtunnel, Bore)

#### Localtunnel

```bash
npm install -g localtunnel
lt --port 8080 --subdomain myapi
```

#### Bore

```bash
# Bore 설치
cargo install bore-cli

# 터널 시작
bore local 8080 --to bore.pub
```

### 자체 터널링 서비스 구축 (Rathole)

[Rathole](https://github.com/rapiz1/rathole)은 Rust로 작성된 빠른 터널링 도구입니다.

서버 (VPS):
```toml
# server.toml
[server]
bind_addr = "0.0.0.0:2333"

[server.services.web]
token = "use_a_secret_that_only_you_know"
bind_addr = "0.0.0.0:8080"
```

클라이언트 (홈서버):
```toml
# client.toml
[client]
remote_addr = "your-vps-ip:2333"

[client.services.web]
token = "use_a_secret_that_only_you_know"
local_addr = "127.0.0.1:8080"
```

## 13.4 도메인 연결과 SSL

### 도메인 구매와 설정

도메인 등록 대행사:
- [Cloudflare Registrar](https://www.cloudflare.com/products/registrar/) (가장 저렴)
- [Namecheap](https://www.namecheap.com/)
- [가비아](https://www.gabia.com/) (한국)

### Cloudflare DNS 설정

1. Cloudflare에 도메인 추가
2. DNS 레코드 추가:

```
Type: A
Name: api
Content: <홈서버 공인 IP>
Proxy: Enabled (주황색 구름)
```

또는 Cloudflare Tunnel 사용 시:
```
Type: CNAME
Name: api
Content: <TUNNEL-ID>.cfargotunnel.com
Proxy: Enabled
```

### SSL 인증서 자동화

#### Nginx Proxy Manager 사용 (권장)

제5장에서 설정한 Nginx Proxy Manager로 관리.

#### Certbot으로 직접 관리

```bash
sudo apt install certbot python3-certbot-nginx -y

# 인증서 발급
sudo certbot --nginx -d api.example.com

# 자동 갱신 테스트
sudo certbot renew --dry-run
```

### 서브도메인 라우팅

```nginx
# api.example.com
server {
    listen 443 ssl;
    server_name api.example.com;

    ssl_certificate /etc/letsencrypt/live/api.example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/api.example.com/privkey.pem;

    location / {
        proxy_pass http://localhost:8080;
    }
}

# app.example.com
server {
    listen 443 ssl;
    server_name app.example.com;

    ssl_certificate /etc/letsencrypt/live/app.example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/app.example.com/privkey.pem;

    location / {
        proxy_pass http://localhost:3000;
    }
}
```

---

이 장에서는 .NET 애플리케이션을 홈서버에 배포하는 핵심 내용을 다뤘습니다. 다음 섹션에서는 리버스 프록시 고급 설정, API 게이트웨이, 실시간 애플리케이션 등을 더 자세히 살펴보겠습니다.

**다음 섹션 미리보기**:
- 13.5 리버스 프록시 고급 설정 (Traefik)
- 13.6 API 게이트웨이 구축 (Kong/YARP)
- 13.7 .NET SignalR 실시간 애플리케이션
- 13.8 Entity Framework Core 연동
- 13.9 애플리케이션 모니터링 (Serilog, Seq)
- 13.10 실전 배포 예제들
