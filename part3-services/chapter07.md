# 제7장: 파일 관리와 클라우드 스토리지

## 7.1 Nextcloud 구축

[Nextcloud](https://nextcloud.com/)는 Google Drive를 대체할 수 있는 개인 클라우드 스토리지입니다.

### 설치와 초기 설정

```yaml
version: '3.8'

services:
  nextcloud-db:
    image: postgres:16-alpine
    container_name: nextcloud-db
    restart: unless-stopped
    volumes:
      - nextcloud-db:/var/lib/postgresql/data
    environment:
      - POSTGRES_DB=nextcloud
      - POSTGRES_USER=nextcloud
      - POSTGRES_PASSWORD=secure_password
    networks:
      - nextcloud

  nextcloud-redis:
    image: redis:alpine
    container_name: nextcloud-redis
    restart: unless-stopped
    networks:
      - nextcloud

  nextcloud:
    image: nextcloud:latest
    container_name: nextcloud
    restart: unless-stopped
    ports:
      - 8888:80
    links:
      - nextcloud-db
      - nextcloud-redis
    volumes:
      - nextcloud:/var/www/html
      - /mnt/data/nextcloud:/var/www/html/data
    environment:
      - POSTGRES_HOST=nextcloud-db
      - POSTGRES_DB=nextcloud
      - POSTGRES_USER=nextcloud
      - POSTGRES_PASSWORD=secure_password
      - REDIS_HOST=nextcloud-redis
      - NEXTCLOUD_TRUSTED_DOMAINS=cloud.example.com homeserver
    networks:
      - nextcloud

volumes:
  nextcloud-db:
  nextcloud:

networks:
  nextcloud:
```

### 모바일 앱 연동

- iOS: App Store에서 "Nextcloud" 검색
- Android: Play Store에서 "Nextcloud" 검색

설정:
- Server URL: `https://cloud.example.com`
- Username/Password: 웹에서 생성한 계정

### Office 문서 편집 기능 추가

[OnlyOffice](https://www.onlyoffice.com/) 또는 [Collabora Online](https://www.collaboraoffice.com/) 연동:

```yaml
services:
  onlyoffice:
    image: onlyoffice/documentserver:latest
    container_name: onlyoffice
    restart: unless-stopped
    ports:
      - 8889:80
    environment:
      - JWT_ENABLED=true
      - JWT_SECRET=your_secret_key
    volumes:
      - onlyoffice-data:/var/www/onlyoffice/Data

volumes:
  onlyoffice-data:
```

Nextcloud에서 OnlyOffice 앱 설치 후 연결.

## 7.2 Syncthing으로 파일 동기화

[Syncthing](https://syncthing.net/)은 P2P 방식의 파일 동기화 도구입니다.

```yaml
version: '3.8'

services:
  syncthing:
    image: syncthing/syncthing:latest
    container_name: syncthing
    hostname: homeserver
    environment:
      - PUID=1000
      - PGID=1000
    volumes:
      - ./config:/var/syncthing/config
      - /mnt/data/sync:/sync
    ports:
      - 8384:8384  # Web UI
      - 22000:22000/tcp  # Sync protocol
      - 22000:22000/udp
      - 21027:21027/udp  # Discovery
    restart: unless-stopped
```

웹 UI: `http://homeserver:8384`

## 7.3 Filebrowser 설치

[Filebrowser](https://filebrowser.org/)는 웹 기반 파일 관리자입니다.

```yaml
version: '3.8'

services:
  filebrowser:
    image: filebrowser/filebrowser:latest
    container_name: filebrowser
    ports:
      - 8082:80
    volumes:
      - /mnt/data:/srv
      - ./filebrowser.db:/database.db
      - ./settings.json:/config/settings.json
    restart: unless-stopped
```

## 7.4 Samba 공유 폴더 설정

Windows/Mac에서 네트워크 드라이브로 접근:

```bash
# Samba 설치
sudo apt install samba -y

# 공유 폴더 생성
sudo mkdir -p /mnt/data/share
sudo chmod 777 /mnt/data/share

# Samba 사용자 추가
sudo smbpasswd -a $USER

# Samba 설정
sudo nano /etc/samba/smb.conf
```

`smb.conf` 추가:
```ini
[share]
   path = /mnt/data/share
   browseable = yes
   writable = yes
   valid users = gildong
```

```bash
# Samba 재시작
sudo systemctl restart smbd
```

Windows에서 연결: `\\homeserver\share`

---

**다음 장에서는**: Jellyfin 미디어 서버와 사진/음악 관리 서비스를 알아보겠습니다.
