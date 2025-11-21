# 제7장: 파일 관리와 클라우드 스토리지

## 왜 자체 파일 스토리지가 필요한가?

### 상용 클라우드 비용의 현실

6장에서 모니터링으로 안정성을 확보했습니다. 이제 새로운 질문: **회사 파일을 어디에 저장할 것인가?**

#### 1단계: Google Drive로 시작 (팀 3-5명)

처음에는 Google Workspace(구 G Suite)로 충분합니다:

```
구성원 현황:
- 대표 1명
- 개발자 2명
- 디자이너 1명
- 마케팅 1명
= 총 5명

Google Workspace 비용:
$12/user/month × 5명 = $60/month ($720/year)
```

**이 단계에서는 괜찮습니다**:
- 월 $60는 감당할 만한 비용
- 자동 백업, 모바일 앱, 협업 기능 제공
- 관리할 서버가 없어 편리함

#### 2단계: 비용 증가와 제약 (팀 10-15명)

**실제로 겪는 문제**:

**상황 1: 월말, 대표실**
```
대표: "이번 달 Google 비용이 왜 이렇게 나왔어?"

청구서:
Google Workspace Business Standard
$12 × 15명 = $180/month

당신: "팀원이 늘어나서..."
대표: "연간 $2,160인데, 이거 우리가 계속 낼 수 있어?"
```

**상황 2: 저장 공간 부족**
```
팀원A: "Google Drive 용량 초과됐어요"

현황:
- Business Standard: 2TB/user
- 디자이너: 디자인 파일로 1.8TB 사용
- 개발자: Docker 이미지 백업 1.5TB 사용
- 마케팅: 동영상 소스 파일 2.3TB 사용 (초과!)

대표: "용량 늘리려면 Business Plus로 업그레이드?"
당신: "그럼 $18/user로 올라갑니다"
대표: "$18 × 15명 = $270/month... 연간 $3,240?"
```

**상황 3: 보안 요구사항**
```
대기업 고객사 보안팀:
"귀사는 고객 데이터를 어디에 보관하나요?"

당신: "Google Drive에 보관하고 있습니다"

고객사 보안팀:
"Google 서버는 해외(미국)에 있죠?
 GDPR과 개인정보보호법 준수 확인서 제출해주세요"

당신: "...Google에 문의해봐야 할 것 같은데요"

고객사:
"데이터 주권 문제가 있습니다.
 국내 서버에 보관해주시거나, 온프레미스 환경 구축 필요합니다"
```

💼 **소규모 조직 관점 - 비용과 제약의 시작**:
- **비용 증가**: 팀원 증가에 비례해서 월 비용 증가 (15명 = $2,160/년)
- **용량 제한**: 팀원당 2TB 제한, 추가 시 더 비싼 플랜 필요
- **데이터 주권**: 대기업/공공기관 고객은 해외 서버 사용 꺼림
- **종속성**: Google 정책 변경 시 영향 받음 (가격 인상, 약관 변경)

#### 3단계: 비용 폭발과 통제 불가 (팀 20명+)

**상황 1: 연간 비용 검토 회의**
```
대표: "올해 클라우드 비용 총정리 좀 해줘"

당신의 집계:
Google Workspace: $12 × 20명 × 12개월 = $2,880/년
추가 스토리지: 10TB × $100/월 = $1,200/년
총계: $4,080/년

대표: "와... 연간 500만원이 클라우드 저장소에만?"
당신: "앞으로 팀원 더 늘면 계속 올라갑니다"
대표: "다른 방법 없어?"
```

**상황 2: 파일 공유의 비효율**
```
협력사 A: "설계 문서 공유해주세요"
당신: (Google Drive 공유 링크 발송)

협력사 A: "회사 보안정책상 외부 클라우드 접속이 막혀있어요.
          이메일로 보내주시거나 FTP 서버 주소 주세요"

당신: "파일이 5GB인데 이메일은 안되고...
      FTP 서버는 없는데..."

결국:
- USB에 담아서 택배 발송
- 또는 협력사가 직접 방문해서 복사
```

**상황 3: 협업 도구 통합 불가**
```
당신: "사내 시스템에서 자동으로 Google Drive에 백업하고 싶은데"

Google Drive API 제약사항:
- OAuth 인증 필요 (복잡함)
- 업로드 속도 제한 (초당 10MB)
- API 호출 할당량 제한 (하루 20,000회)

팀원: "우리 CI/CD 파이프라인에서 빌드 결과물을
      Google Drive에 자동 업로드하려 했는데
      API 할당량 초과로 실패했어요"

당신: "할당량 늘리려면 Enterprise 플랜으로..."
대표: "또 돈?"
```

**상황 4: 내부 백업 서버 필요**
```
대표: "Google Drive만 믿어도 되나?
      혹시 Google 장애나면 우리 일 못하는 거 아니야?"

당신: "그래서 중요 파일은 로컬에도 백업해야 합니다"

현재 백업 방법:
- 각 팀원이 주요 폴더를 수동으로 로컬에 복사
- 일주일에 한 번씩? (안 하는 사람도 많음)
- 버전 관리 없음

실제 장애 상황 (2024년 11월):
- Google Drive 2시간 다운
- 팀원들: "파일 접근 안돼요!"
- 업무 마비 2시간
```

💼 **소규모 조직 관점 - 비즈니스 리스크 발생**:
- **비용 폭증**: 20명 × $12 + 추가 스토리지 = 연간 $4,000+ (500만원+)
- **종속성 위험**: Google 장애 시 전사 업무 마비
- **통합 불가**: 사내 시스템과 자동화 어려움
- **보안 요구사항**: 대기업 고객 확보 시 온프레미스 요구
- **협업 제약**: 외부 협력사가 Google Drive 접근 못하는 경우 많음

#### 4단계: Nextcloud - 완전한 통제권 확보

**도입 후 변화**:

| 항목 | Google Workspace | Nextcloud (자체 구축) | 절감/개선 |
|------|------------------|----------------------|-----------|
| **비용** | $12/user × 20명 = $240/월 | $0/월 | **$2,880/년 절감** |
| **저장 공간** | 2TB/user (제한) | 무제한 (HDD 용량만큼) | **제한 없음** |
| **추가 스토리지** | 10TB = $100/월 | 4TB HDD = $100 (1회) | **$1,200/년 절감** |
| **API 제약** | 할당량 제한 | 제한 없음 | **자동화 자유** |
| **데이터 위치** | 미국 Google 서버 | 사내 서버 (완전 통제) | **컴플라이언스 충족** |
| **백업 통제** | Google에 의존 | 직접 관리 (안심) | **리스크 제거** |
| **협력사 공유** | 계정 필요 | WebDAV/Samba로 자유 | **협업 자유도 ↑** |

**비용 절감 계산**:

```
💰 연간 구독료 절감:
   Google Workspace: $2,880/년
   추가 스토리지: $1,200/년
   합계: $4,080/년 → Nextcloud로 $0/년
   절감: $4,080/년

💰 초기 투자:
   HDD 4TB × 2개 (RAID 미러): $200
   1년이면 투자금 회수: $4,080 - $200 = $3,880 절감

💰 5년 총 비용 비교:
   Google: $4,080 × 5년 = $20,400
   Nextcloud: $200 (HDD) + $100 (전기료/5년) = $300
   절감: $20,100 (약 2,500만원)
```

**실제 사용 예시**:

```
도입 전 - Google Workspace:
- 월 $240 고정 비용
- 2TB 제한에 걸려서 파일 삭제 반복
- 대기업 고객: "해외 서버는 곤란합니다"
- 협력사: "외부 클라우드 접속 안됩니다"
- Google 장애 시 업무 마비

도입 후 - Nextcloud:
- 월 비용 $0
- 10TB 저장 공간 (HDD만 추가하면 무제한 확장)
- 대기업 고객: "온프레미스라 승인 가능합니다"
- 협력사: WebDAV로 연결 또는 공유 링크 제공
- 백업 직접 관리 (NAS로 자동 백업)
```

**Nextcloud 추가 기능**:

```
✅ Google Drive 대체:
   - 웹 인터페이스로 파일 업/다운로드
   - Windows/Mac/Linux 동기화 클라이언트
   - iOS/Android 앱

✅ Google Docs 대체:
   - OnlyOffice 연동으로 워드/엑셀/PPT 편집
   - 실시간 협업 편집 가능
   - MS Office와 100% 호환

✅ 추가 혜택:
   - 캘린더/연락처 동기화
   - 화상회의 (Nextcloud Talk)
   - 태스크 관리
   - 2단계 인증 (보안 강화)
   - 파일 버전 관리 (실수로 삭제해도 복구)
```

**보안 및 컴플라이언스**:

```
대기업 고객사 보안팀: "데이터 보관 위치는?"
당신: "자사 서버에 온프레미스로 보관하고 있습니다"

고객사: "백업은?"
당신: "매일 자동 백업되며, 백업 데이터도 국내 서버에 보관합니다"

고객사: "접근 제어는?"
당신: "2단계 인증, IP 제한, 역할 기반 권한 관리 적용 중입니다"

고객사: "승인합니다 ✓"

👉 대기업/공공기관 프로젝트 수주 가능!
```

**협력사 파일 공유**:

```
도입 전:
협력사: "Google Drive 접속 안됩니다"
→ USB 택배 발송 (3일 소요)

도입 후:
협력사: "어떻게 받죠?"
당신: "https://cloud.company.com/s/abc123 클릭하세요"
협력사: "바로 다운로드 됐습니다!"

또는:
협력사: "FTP 주소 주세요"
당신: "WebDAV 연결 주소입니다: webdav://cloud.company.com"
협력사: (탐색기에서 네트워크 드라이브로 바로 연결)
```

💡 **핵심 가치**:
- **비용 절감**: 연간 $4,000 이상 절감 (5년이면 $20,000)
- **무제한 확장**: HDD만 추가하면 저장 공간 무제한
- **완전한 통제**: 백업, 보안, 정책 모두 직접 관리
- **컴플라이언스**: 데이터 주권 확보로 대기업/공공기관 프로젝트 수주 가능
- **협업 자유**: WebDAV, Samba, 공유 링크로 다양한 방식 지원
- **종속성 제거**: Google 정책 변경, 가격 인상에 영향 없음
- **오피스 통합**: OnlyOffice로 MS Office 문서 실시간 협업 편집

---

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
