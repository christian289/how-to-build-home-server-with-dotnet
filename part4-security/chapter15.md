# 제15장: 백업과 복구 전략

## 15.1 백업 전략 수립

### 3-2-1 백업 규칙

- **3**: 데이터의 복사본 3개 유지
- **2**: 2가지 다른 저장 매체 사용
- **1**: 1개는 오프사이트 (다른 물리적 위치)

**홈서버 적용 예시**:
1. 주 저장장치 (NVMe SSD)
2. 로컬 백업 (외장 HDD)
3. 클라우드 백업 (Google Drive/Backblaze)

### 백업 대상 선정

**필수 백업**:
- Docker 볼륨 데이터
- 설정 파일 (docker-compose.yml, .env)
- 데이터베이스 덤프
- 개인 파일 (문서, 사진, 동영상)

**백업 불필요**:
- Docker 이미지 (재다운로드 가능)
- 캐시 파일
- 임시 파일

## 15.2 자동 백업 시스템

### Duplicati 설정

[Duplicati](https://www.duplicati.com/)는 증분 백업을 지원하는 도구입니다.

```yaml
version: '3.8'

services:
  duplicati:
    image: lscr.io/linuxserver/duplicati:latest
    container_name: duplicati
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Asia/Seoul
    volumes:
      - ./duplicati/config:/config
      - /mnt/data:/source:ro
      - /mnt/backup:/backups
    ports:
      - 8200:8200
    restart: unless-stopped
```

웹 UI: `http://homeserver:8200`

**백업 작업 생성**:
1. **Add backup** 클릭
2. Source: `/source`
3. Destination: Google Drive / S3 / FTP
4. Schedule: 매일 새벽 2시
5. Retention: 30일치 보관

### Restic 활용

[Restic](https://restic.net/)은 CLI 기반 백업 도구입니다.

```bash
# Restic 설치
sudo apt install restic -y

# 백업 저장소 초기화
restic init --repo /mnt/backup/restic

# 백업 실행
restic -r /mnt/backup/restic backup /mnt/data

# 백업 목록
restic -r /mnt/backup/restic snapshots

# 복원
restic -r /mnt/backup/restic restore latest --target /mnt/restore
```

**자동화 스크립트**:

```bash
#!/bin/bash
# ~/scripts/restic-backup.sh

export RESTIC_REPOSITORY=/mnt/backup/restic
export RESTIC_PASSWORD=your_password

# 백업
restic backup /mnt/data \
  --exclude /mnt/data/cache \
  --exclude /mnt/data/tmp

# 오래된 스냅샷 정리
restic forget --keep-daily 7 --keep-weekly 4 --keep-monthly 6 --prune

# 상태 확인
restic check
```

Cron 등록:
```
0 2 * * * ~/scripts/restic-backup.sh >> ~/logs/restic.log 2>&1
```

## 15.3 클라우드 백업 연동

### Rclone으로 다양한 클라우드 지원

[Rclone](https://rclone.org/)은 클라우드 스토리지 동기화 도구입니다.

```bash
# Rclone 설치
curl https://rclone.org/install.sh | sudo bash

# 클라우드 설정
rclone config

# Google Drive 동기화
rclone sync /mnt/data gdrive:backup --progress

# Backblaze B2 동기화
rclone sync /mnt/data b2:mybucket/backup --progress
```

**자동화**:

```bash
#!/bin/bash
# ~/scripts/cloud-backup.sh

rclone sync /mnt/data/important gdrive:backup/important \
  --exclude "*.tmp" \
  --exclude "cache/**" \
  --log-file ~/logs/rclone.log
```

## 15.4 시스템 복구 절차

### 재난 시나리오별 복구

**시나리오 1: 컨테이너 손상**

```bash
# 컨테이너 재생성
docker compose down
docker compose up -d
```

**시나리오 2: 데이터 손실**

```bash
# Restic에서 복원
restic -r /mnt/backup/restic restore latest --target /mnt/data
```

**시나리오 3: 시스템 전체 재설치**

1. Ubuntu Server 재설치
2. Docker 설치
3. Git 저장소에서 설정 파일 클론
```bash
git clone https://github.com/username/homeserver-config.git
cd homeserver-config
./restore.sh
```
4. 백업에서 데이터 복원

### 복구 테스트

정기적으로 복구 절차 테스트:

```bash
# 테스트 복원
mkdir -p /tmp/restore-test
restic -r /mnt/backup/restic restore latest --target /tmp/restore-test

# 파일 무결성 확인
diff -r /mnt/data /tmp/restore-test
```

## 15.5 재해 복구 계획

### 문서화

`disaster-recovery.md` 작성:

```markdown
# 재해 복구 계획

## 긴급 연락처
- IT 담당자: 010-XXXX-XXXX
- 클라우드 서비스: support@provider.com

## 복구 우선순위
1. 비밀번호 관리자 (Vaultwarden)
2. 파일 스토리지 (Nextcloud)
3. 미디어 서버 (Jellyfin)
4. 기타 서비스

## 복구 절차
1. 시스템 재설치
2. Docker 환경 구성
3. 백업 복원
4. 서비스 확인

## 백업 위치
- 로컬: /mnt/backup
- 클라우드: gdrive:backup, b2:homeserver-backup
```

### 오프사이트 백업

최소 월 1회 외장 HDD를 다른 장소에 보관:
- 친구/가족 집
- 사무실
- 안전 금고

---

**다음 장에서는**: 시스템 성능 최적화 방법을 알아보겠습니다.
