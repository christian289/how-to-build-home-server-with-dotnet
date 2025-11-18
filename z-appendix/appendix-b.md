# 부록 B: 스크립트와 자동화

## B.1 백업 스크립트

### 전체 백업

```bash
#!/bin/bash
# backup-all.sh

BACKUP_DIR="/mnt/backup/$(date +%Y%m%d)"
mkdir -p "$BACKUP_DIR"

# Docker 볼륨 백업
docker run --rm -v /var/lib/docker/volumes:/volumes \
  -v "$BACKUP_DIR":/backup alpine \
  tar czf /backup/docker-volumes.tar.gz -C /volumes .

# 설정 파일 백업
tar czf "$BACKUP_DIR/configs.tar.gz" \
  ~/docker-compose \
  ~/scripts \
  /etc/nginx

echo "백업 완료: $BACKUP_DIR"
```

### PostgreSQL 백업

```bash
#!/bin/bash
# backup-postgres.sh

CONTAINER="postgres"
BACKUP_DIR="/mnt/backup/postgres"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"

docker exec "$CONTAINER" pg_dumpall -U postgres | \
  gzip > "$BACKUP_DIR/postgres_$DATE.sql.gz"

# 7일 이상 된 백업 삭제
find "$BACKUP_DIR" -name "*.sql.gz" -mtime +7 -delete
```

## B.2 모니터링 스크립트

### 디스크 사용량 알림

```bash
#!/bin/bash
# disk-alert.sh

THRESHOLD=80
USAGE=$(df -h / | awk 'NR==2 {print $5}' | sed 's/%//')

if [ "$USAGE" -gt "$THRESHOLD" ]; then
    curl -X POST "https://discord.com/api/webhooks/YOUR_WEBHOOK" \
      -H "Content-Type: application/json" \
      -d "{\"content\": \"⚠️ 디스크 사용률: ${USAGE}%\"}"
fi
```

### 컨테이너 상태 체크

```bash
#!/bin/bash
# container-health.sh

CONTAINERS=$(docker ps --format '{{.Names}}')

for container in $CONTAINERS; do
    STATUS=$(docker inspect --format='{{.State.Health.Status}}' "$container" 2>/dev/null)
    if [ "$STATUS" = "unhealthy" ]; then
        echo "⚠️ $container is unhealthy!"
        # 알림 발송
    fi
done
```

## B.3 정리 스크립트

### Docker 정리

```bash
#!/bin/bash
# docker-cleanup.sh

# 중지된 컨테이너 삭제
docker container prune -f

# 사용하지 않는 이미지 삭제
docker image prune -a -f

# 사용하지 않는 볼륨 삭제
docker volume prune -f

# 사용하지 않는 네트워크 삭제
docker network prune -f

# 빌드 캐시 정리
docker builder prune -a -f

echo "Docker 정리 완료!"
```

## B.4 배포 자동화

### Docker 이미지 빌드 스크립트

```bash
#!/bin/bash
# build.sh

APP_NAME="myapp"
VERSION=$(git describe --tags --always)
REGISTRY="homeserver:5000"

docker build -t "$APP_NAME:$VERSION" .
docker tag "$APP_NAME:$VERSION" "$APP_NAME:latest"
docker tag "$APP_NAME:$VERSION" "$REGISTRY/$APP_NAME:$VERSION"
docker tag "$APP_NAME:$VERSION" "$REGISTRY/$APP_NAME:latest"

echo "빌드 완료: $APP_NAME:$VERSION"
```

### Zero-Downtime 배포 스크립트

```bash
#!/bin/bash
# deploy.sh

SERVICE_NAME="myapp"
NEW_IMAGE="myapp:latest"

# 새 이미지 pull
docker compose pull "$SERVICE_NAME"

# 무중단 재배포
docker compose up -d --no-deps --build "$SERVICE_NAME"

# 헬스 체크
sleep 5
if docker compose ps "$SERVICE_NAME" | grep -q "Up"; then
    echo "✅ 배포 성공"
else
    echo "❌ 배포 실패, 롤백 필요"
    exit 1
fi
```

### 롤백 스크립트

```bash
#!/bin/bash
# rollback.sh

SERVICE_NAME="myapp"
PREVIOUS_IMAGE="myapp:previous"

docker compose stop "$SERVICE_NAME"
docker compose rm -f "$SERVICE_NAME"
docker tag "$PREVIOUS_IMAGE" "myapp:latest"
docker compose up -d "$SERVICE_NAME"

echo "롤백 완료"
```

### 헬스체크 스크립트

```bash
#!/bin/bash
# healthcheck.sh

URL="http://localhost:8080/health"
TIMEOUT=5

response=$(curl -s -o /dev/null -w "%{http_code}" --max-time "$TIMEOUT" "$URL")

if [ "$response" -eq 200 ]; then
    exit 0
else
    exit 1
fi
```

---

모든 스크립트를 `~/scripts/` 폴더에 저장하고 실행 권한 부여:

```bash
chmod +x ~/scripts/*.sh
```

Cron 등록 예시:

```cron
# 매일 새벽 2시 백업
0 2 * * * ~/scripts/backup-all.sh >> ~/logs/backup.log 2>&1

# 매시간 디스크 체크
0 * * * * ~/scripts/disk-alert.sh

# 매일 새벽 4시 Docker 정리
0 4 * * * ~/scripts/docker-cleanup.sh
```
