# 제8장: 미디어 서버 구축

## 왜 개인 미디어 서버가 필요한가?

### 구독 경제의 함정과 프라이버시 문제

7장에서 파일 스토리지로 비용을 절감했습니다. 이제 미디어 관리: **구독료와 프라이버시, 어디까지 감당할 것인가?**

#### 상황 분석: 미디어 구독 비용 폭증

**가족/소규모 조직의 미디어 구독 현황 (2025년)**:

```
💰 월간 구독료 합계:
- Netflix Premium (4K): $23/월
- Disney+: $14/월
- Spotify Family (6인): $17/월
- YouTube Premium Family: $23/월
- Google Photos (2TB): $10/월

합계: $87/월 = $1,044/년

5년 총 비용: $5,220
10년 총 비용: $10,440
```

**Google Photos의 숨겨진 비용**:

```
현재 상황 (20명 조직):
- 사진/동영상: 개인당 평균 50GB
- 총 필요 용량: 1TB
- Google Photos 100GB: $2/월 (부족)
- Google Photos 2TB: $10/월 (필요)

문제점:
1. 개인 사진이 Google 서버에 업로드됨
2. Google AI가 모든 사진 분석 (프라이버시?)
3. 가족 행사 사진, 회사 워크샵 사진 등 민감 정보 노출
4. Google 계정 해킹 시 모든 사진 유출 위험
```

**회사 차원의 문제**:

```
상황 1: 팀 워크샵 사진 공유
팀원A: "워크샵 사진 공유 좀 해주세요"
당신: Google Photos 링크 공유

보안팀: "회사 행사 사진을 외부 서버에 올렸나요?"
당신: "다들 Google Photos 쓰는데..."
보안팀: "내부 정보 유출 위험이 있습니다. 즉시 삭제하고 사내 서버로 옮기세요"

상황 2: 교육/세미나 동영상 보관
- 강의 녹화 영상 20GB
- Netflix? (업로드 불가)
- YouTube? (비공개 설정해도 불안)
- Google Drive? (스트리밍 품질 낮음)
- 외장 HDD? (공유 불편, 분실 위험)
```

💼 **소규모 조직 관점 - 구독 피로와 프라이버시 리스크**:
- **구독 피로**: 연간 $1,000+ 지출에도 콘텐츠는 내 것이 아님
- **프라이버시 우려**: 개인/회사 사진이 빅테크 서버에 업로드됨
- **종속성**: 구독 취소 시 모든 콘텐츠 접근 불가
- **광고**: YouTube Premium 없으면 광고 시청 필수
- **콘텐츠 삭제**: 스트리밍 서비스가 라이선스 종료 시 영상 사라짐

#### 도입 후 변화: Jellyfin + Immich

**비용 비교 (5년 기준)**:

| 항목 | 구독 서비스 | 자체 구축 | 절감 |
|------|------------|----------|------|
| **영상 스트리밍** | Netflix $1,380 (5년) | Jellyfin $0 | **$1,380** |
| **음악 스트리밍** | Spotify $1,020 (5년) | Navidrome $0 | **$1,020** |
| **사진 백업** | Google Photos $600 (5년) | Immich $0 | **$600** |
| **YouTube 광고 제거** | YouTube Premium $1,380 (5년) | - | **$1,380** |
| **저장 공간** | - | HDD 4TB $100 (1회) | - |
| **총 비용** | $4,380 (5년) | $100 (HDD만) | **$4,280 절감** |

**실제 사용 시나리오**:

```
✅ 회사 교육 자료 스트리밍:
   - 세미나 영상 업로드
   - 팀원들이 언제든 시청
   - 외부 유출 걱정 없음

✅ 워크샵 사진 자동 백업:
   - Immich 앱으로 촬영 즉시 자동 백업
   - AI 얼굴 인식으로 사람별 자동 분류
   - 회사 서버에 안전하게 보관
   - Google에 업로드하지 않음 (프라이버시 확보)

✅ 개인 음악 라이브러리:
   - 구매한 음악 파일을 Navidrome에 업로드
   - Spotify 없이도 어디서든 스트리밍
   - 음악 구독료 절약

✅ 가족 공유:
   - 집에서 Jellyfin으로 영화/드라마 시청
   - Netflix 구독 불필요
   - 4K HDR 지원 (N100 하드웨어 가속)
```

💡 **핵심 가치**:
- **비용 절감**: 5년간 $4,000+ 절약
- **프라이버시**: 사진/영상이 외부 서버에 업로드되지 않음
- **영구 소유**: 구독 취소해도 콘텐츠는 내 것
- **무제한 저장**: HDD 추가만 하면 무제한 확장
- **광고 없음**: 모든 콘텐츠를 광고 없이 시청
- **회사 콘텐츠**: 교육 자료, 워크샵 영상 등 안전하게 보관

**프라이버시 비교**:

```
Google Photos:
❌ 모든 사진이 Google 서버로 업로드
❌ Google AI가 사진 내용 분석 (얼굴, 장소, 사물 인식)
❌ 광고 타겟팅에 활용 가능
❌ 정부 요청 시 사진 제공 가능
❌ 서비스 약관 변경 시 어쩔 수 없이 동의

Immich (자체 구축):
✅ 모든 사진이 내 서버에만 보관
✅ AI 분석도 내 서버에서만 실행
✅ 외부 유출 없음
✅ 완전한 통제권
✅ 언제든 데이터 백업/이동 가능
```

**회사 활용 사례**:

```
시나리오 1: 신입 교육 시스템
- 온보딩 교육 영상을 Jellyfin에 업로드
- 신입사원에게 링크 공유
- 원격근무자도 언제든 시청 가능
- YouTube에 비공개로 올릴 필요 없음

시나리오 2: 팀 빌딩 사진 공유
- 워크샵 사진 500장을 Immich에 자동 백업
- AI가 자동으로 사람별 분류
- 팀원들: "제 사진만 모아서 다운로드할 수 있네요!"
- Google Photos 구독료 절약

시나리오 3: 회사 소개 영상 보관
- 홍보 영상, 제품 데모 영상 등
- 고객사 방문 시 태블릿으로 바로 재생
- 인터넷 연결 없이도 재생 가능 (로컬 캐시)
```

---

## 8.1 Jellyfin 미디어 서버

[Jellyfin](https://jellyfin.org/)은 Netflix와 같은 개인 미디어 서버입니다.

### 설치와 라이브러리 구성

```yaml
version: '3.8'

services:
  jellyfin:
    image: jellyfin/jellyfin:latest
    container_name: jellyfin
    user: 1000:1000
    network_mode: 'host'
    volumes:
      - ./config:/config
      - ./cache:/cache
      - /mnt/data/media/movies:/media/movies
      - /mnt/data/media/tv:/media/tv
      - /mnt/data/media/music:/media/music
    restart: unless-stopped
    devices:
      - /dev/dri:/dev/dri  # Intel QuickSync
```

웹 UI: `http://homeserver:8096`

### 하드웨어 가속 설정 (Intel QuickSync)

N100 프로세서의 내장 GPU를 활용한 트랜스코딩:

1. **Dashboard** → **Playback** 이동
2. **Hardware acceleration**: Intel QuickSync (QSV) 선택
3. 인코딩 옵션 체크:
   - Enable hardware decoding for: H264, HEVC, VP9
   - Enable hardware encoding
   - Enable VPP Tone mapping

### 모바일/TV 앱 설정

- iOS/Android: "Jellyfin" 앱 다운로드
- Smart TV: 앱 스토어에서 Jellyfin 검색
- 서버 주소 입력: `http://homeserver:8096`

## 8.2 음악 스트리밍 (Navidrome)

[Navidrome](https://www.navidrome.org/)는 개인 Spotify 서버입니다.

```yaml
version: '3.8'

services:
  navidrome:
    image: deluan/navidrome:latest
    container_name: navidrome
    user: 1000:1000
    ports:
      - 4533:4533
    environment:
      ND_SCANSCHEDULE: 1h
      ND_LOGLEVEL: info
      ND_SESSIONTIMEOUT: 24h
      ND_BASEURL: ""
    volumes:
      - ./data:/data
      - /mnt/data/media/music:/music:ro
    restart: unless-stopped
```

Subsonic 호환 앱으로 접속 가능:
- iOS: play:Sub
- Android: DSub, Ultrasonic

## 8.3 사진 관리 (Immich)

[Immich](https://immich.app/)은 Google Photos 대체 솔루션입니다.

```yaml
version: '3.8'

services:
  immich-server:
    container_name: immich_server
    image: ghcr.io/immich-app/immich-server:release
    command: ['start.sh', 'immich']
    volumes:
      - /mnt/data/photos:/usr/src/app/upload
      - /etc/localtime:/etc/localtime:ro
    env_file:
      - .env
    depends_on:
      - redis
      - database
    restart: unless-stopped

  immich-microservices:
    container_name: immich_microservices
    image: ghcr.io/immich-app/immich-server:release
    command: ['start.sh', 'microservices']
    volumes:
      - /mnt/data/photos:/usr/src/app/upload
      - /etc/localtime:/etc/localtime:ro
    env_file:
      - .env
    depends_on:
      - redis
      - database
    restart: unless-stopped

  immich-machine-learning:
    container_name: immich_machine_learning
    image: ghcr.io/immich-app/immich-machine-learning:release
    volumes:
      - model-cache:/cache
    env_file:
      - .env
    restart: unless-stopped

  immich-web:
    container_name: immich_web
    image: ghcr.io/immich-app/immich-web:release
    env_file:
      - .env
    restart: unless-stopped

  redis:
    container_name: immich_redis
    image: redis:6.2-alpine
    restart: unless-stopped

  database:
    container_name: immich_postgres
    image: tensorchord/pgvecto-rs:pg14-v0.2.0
    env_file:
      - .env
    environment:
      POSTGRES_PASSWORD: ${DB_PASSWORD}
      POSTGRES_USER: ${DB_USERNAME}
      POSTGRES_DB: ${DB_DATABASE_NAME}
    volumes:
      - pgdata:/var/lib/postgresql/data
    restart: unless-stopped

  immich-proxy:
    container_name: immich_proxy
    image: ghcr.io/immich-app/immich-proxy:release
    environment:
      IMMICH_SERVER_URL: http://immich-server:3001
      IMMICH_WEB_URL: http://immich-web:3000
    ports:
      - 2283:8080
    depends_on:
      - immich-server
      - immich-web
    restart: unless-stopped

volumes:
  pgdata:
  model-cache:
```

`.env`:
```env
DB_PASSWORD=postgres
DB_USERNAME=postgres
DB_DATABASE_NAME=immich
```

### AI 얼굴 인식 설정

Immich는 자동으로 얼굴을 인식하고 그룹화합니다. 추가 설정 불필요.

### 모바일 자동 백업

- iOS/Android 앱 설치
- 설정 → Background backup 활성화
- 자동으로 사진/비디오 업로드

## 8.4 전자책 관리 (Calibre-Web)

[Calibre-Web](https://github.com/janeczku/calibre-web)으로 전자책 라이브러리 관리:

```yaml
version: '3.8'

services:
  calibre-web:
    image: lscr.io/linuxserver/calibre-web:latest
    container_name: calibre-web
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Asia/Seoul
    volumes:
      - ./config:/config
      - /mnt/data/books:/books
    ports:
      - 8083:8083
    restart: unless-stopped
```

기본 로그인: `admin` / `admin123`

---

**다음 장에서는**: qBittorrent와 Sonarr/Radarr를 활용한 다운로드 자동화 시스템을 알아보겠습니다.
