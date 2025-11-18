# 제5장: 리버스 프록시와 도메인 설정

## 5.1 Nginx Proxy Manager 설치

[Nginx Proxy Manager](https://nginxproxymanager.com/)는 리버스 프록시를 웹 GUI로 쉽게 관리할 수 있는 도구입니다.

### Docker Compose로 설치

`docker-compose.yml`:

```yaml
version: '3.8'

services:
  nginx-proxy-manager:
    image: 'jc21/nginx-proxy-manager:latest'
    container_name: nginx-proxy-manager
    restart: unless-stopped
    ports:
      - '80:80'     # HTTP
      - '443:443'   # HTTPS
      - '81:81'     # Admin Web UI
    environment:
      DB_SQLITE_FILE: "/data/database.sqlite"
    volumes:
      - ./data:/data
      - ./letsencrypt:/etc/letsencrypt
    networks:
      - proxy-network

networks:
  proxy-network:
    driver: bridge
```

```bash
# 설치
docker compose up -d

# 로그 확인
docker compose logs -f
```

### 초기 설정

1. 웹 브라우저에서 `http://homeserver:81` 접속
2. 기본 로그인 정보:
   - Email: `admin@example.com`
   - Password: `changeme`
3. 즉시 비밀번호 변경
4. 이메일 주소를 본인 것으로 변경

### 첫 프록시 호스트 추가

1. **Proxy Hosts** → **Add Proxy Host** 클릭
2. 설정:
   - Domain Names: `portainer.homelab.local`
   - Scheme: `http`
   - Forward Hostname/IP: `portainer`
   - Forward Port: `9000`
   - Block Common Exploits: ✓
   - Websockets Support: ✓

이제 `http://portainer.homelab.local`로 Portainer에 접속할 수 있습니다.

## 5.2 도메인 구매와 DDNS 설정

### DuckDNS 활용법

[DuckDNS](https://www.duckdns.org/)는 무료 DDNS 서비스입니다.

#### 1. DuckDNS 계정 생성

1. https://www.duckdns.org/ 방문
2. GitHub/Google 계정으로 로그인
3. 원하는 서브도메인 생성: `myserver.duckdns.org`
4. 토큰 복사

#### 2. IP 자동 업데이트 설정

```bash
# DuckDNS 업데이트 스크립트 생성
mkdir -p ~/scripts
cat > ~/scripts/duckdns-update.sh <<'EOF'
#!/bin/bash
echo url="https://www.duckdns.org/update?domains=myserver&token=YOUR_TOKEN&ip=" | curl -k -o ~/duckdns.log -K -
EOF

chmod +x ~/scripts/duckdns-update.sh

# Cron으로 5분마다 실행
crontab -e
```

Cron 항목 추가:
```
*/5 * * * * ~/scripts/duckdns-update.sh >/dev/null 2>&1
```

#### Docker로 자동 업데이트

더 간편하게 Docker 컨테이너로 관리할 수 있습니다:

```yaml
services:
  duckdns:
    image: lscr.io/linuxserver/duckdns:latest
    container_name: duckdns
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Asia/Seoul
      - SUBDOMAINS=myserver
      - TOKEN=YOUR_DUCKDNS_TOKEN
      - UPDATE_IP=ipv4
    restart: unless-stopped
```

### Cloudflare 연동

[Cloudflare](https://www.cloudflare.com/)는 무료 CDN과 DNS 서비스를 제공합니다.

#### 1. 도메인 추가

1. Cloudflare 가입 후 로그인
2. **Add a Site** 클릭
3. 도메인 입력 (예: `example.com`)
4. Free 플랜 선택
5. 네임서버를 Cloudflare 것으로 변경 (도메인 등록 대행사에서)

#### 2. DNS 레코드 추가

**A 레코드**:
```
Type: A
Name: home
Content: <집 공인 IP>
Proxy status: Proxied (주황색 구름)
TTL: Auto
```

**CNAME 레코드**:
```
Type: CNAME
Name: *.home
Content: home.example.com
Proxy status: Proxied
```

이제 `home.example.com`, `portainer.home.example.com` 등으로 접속 가능합니다.

#### 3. Cloudflare API로 동적 IP 업데이트

```bash
cat > ~/scripts/cloudflare-ddns.sh <<'EOF'
#!/bin/bash

# Cloudflare 설정
CF_API_TOKEN="your_api_token"
ZONE_ID="your_zone_id"
RECORD_NAME="home.example.com"
RECORD_ID="your_record_id"

# 현재 공인 IP 가져오기
CURRENT_IP=$(curl -s https://api.ipify.org)

# Cloudflare DNS 업데이트
curl -X PUT "https://api.cloudflare.com/client/v4/zones/${ZONE_ID}/dns_records/${RECORD_ID}" \
  -H "Authorization: Bearer ${CF_API_TOKEN}" \
  -H "Content-Type: application/json" \
  --data "{\"type\":\"A\",\"name\":\"${RECORD_NAME}\",\"content\":\"${CURRENT_IP}\",\"ttl\":120,\"proxied\":true}"
EOF

chmod +x ~/scripts/cloudflare-ddns.sh
```

## 5.3 SSL 인증서 자동화 (Let's Encrypt)

### Nginx Proxy Manager에서 SSL 설정

1. **Proxy Hosts**에서 호스트 편집
2. **SSL** 탭으로 이동
3. SSL Certificate: **Request a new SSL Certificate**
4. Email: 본인 이메일 입력
5. **I Agree to the Let's Encrypt Terms of Service** 체크
6. **Force SSL** 체크
7. **Save** 클릭

자동으로 [Let's Encrypt](https://letsencrypt.org/) 인증서를 발급받고 90일마다 자동 갱신됩니다.

### Wildcard 인증서 발급

여러 서브도메인을 사용한다면 와일드카드 인증서가 편리합니다.

1. **SSL Certificates** → **Add SSL Certificate**
2. Domain Names: `*.example.com`, `example.com`
3. Email: 본인 이메일
4. **Use a DNS Challenge** 체크
5. DNS Provider: Cloudflare 선택
6. Credentials File Content:
```
dns_cloudflare_api_token = your_cloudflare_api_token
```
7. **Save** 클릭

이제 모든 서브도메인에서 동일한 SSL 인증서를 사용할 수 있습니다.

### Certbot으로 직접 관리 (고급)

Nginx Proxy Manager를 사용하지 않는 경우:

```bash
# Certbot 설치
sudo apt install certbot python3-certbot-nginx -y

# 인증서 발급
sudo certbot --nginx -d example.com -d www.example.com

# 자동 갱신 테스트
sudo certbot renew --dry-run

# Cron으로 자동 갱신 (이미 자동 설정됨)
sudo systemctl status certbot.timer
```

## 5.4 내부 DNS 서버 구축 (Pi-hole)

[Pi-hole](https://pi-hole.net/)은 네트워크 전체의 광고를 차단하는 DNS 서버입니다.

### Docker로 Pi-hole 설치

`docker-compose.yml`:

```yaml
version: '3.8'

services:
  pihole:
    container_name: pihole
    image: pihole/pihole:latest
    ports:
      - "53:53/tcp"
      - "53:53/udp"
      - "67:67/udp"
      - "8089:80/tcp"
    environment:
      TZ: 'Asia/Seoul'
      WEBPASSWORD: 'your_secure_password'
      FTLCONF_LOCAL_IPV4: '192.168.0.100'
      PIHOLE_DNS_: '1.1.1.1;8.8.8.8'
    volumes:
      - './etc-pihole:/etc/pihole'
      - './etc-dnsmasq.d:/etc/dnsmasq.d'
    restart: unless-stopped
```

```bash
docker compose up -d
```

### 공유기 DNS 설정 변경

1. 공유기 관리 페이지 접속
2. **DHCP 설정** 메뉴
3. Primary DNS: `192.168.0.100` (홈서버 IP)
4. Secondary DNS: `1.1.1.1` (백업용)
5. 저장 후 재부팅

### Pi-hole 웹 인터페이스

- URL: `http://homeserver:8089/admin`
- 비밀번호: docker-compose에서 설정한 비밀번호

### 로컬 DNS 레코드 추가

Pi-hole에서 내부 호스트명을 관리할 수 있습니다.

1. **Local DNS** → **DNS Records**
2. Domain: `portainer.homelab.local`
3. IP Address: `192.168.0.100`
4. **Add** 클릭

이제 네트워크의 모든 기기에서 `portainer.homelab.local`로 접속할 수 있습니다.

### AdLists 추가 (광고 차단 강화)

1. **Adlists** 메뉴
2. 다음 리스트 추가:
   - https://raw.githubusercontent.com/StevenBlack/hosts/master/hosts
   - https://v.firebog.net/hosts/AdguardDNS.txt
   - https://v.firebog.net/hosts/Easylist.txt
3. **Tools** → **Update Gravity** 실행

### 통계 확인

대시보드에서:
- 차단된 광고 수
- DNS 쿼리 통계
- Top Domains
- Top Blocked Domains

---

**다음 장에서는**: Grafana, Prometheus, Netdata를 활용한 시스템 모니터링 환경 구축 방법을 알아보겠습니다.
