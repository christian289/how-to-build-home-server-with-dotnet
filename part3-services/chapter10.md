# 제10장: 개인 생산성 도구

## 왜 자체 생산성 도구가 필요한가?

### SaaS 생산성 도구의 비용 폭탄

9장에서 콘텐츠 자동화를 구축했습니다. 이제 업무 생산성: **구독 중인 SaaS 비용, 계속 낼 것인가?**

#### 생산성 SaaS 구독 비용 현황 (20명 조직)

```
💰 월간 생산성 도구 구독료:
- Notion Team: $15/user × 20명 = $300/월 ($3,600/년)
- 1Password Teams: $8/user × 20명 = $160/월 ($1,920/년)
- Todoist Business: $6/user × 20명 = $120/월 ($1,440/년)
- Evernote Teams: $15/user × 20명 = $300/월 ($3,600/년)

합계: $880/월 = $10,560/년
5년 총 비용: $52,800 (약 6,600만원)
```

**실제로 겪는 문제**:

```
상황 1: 분기 예산 회의
대표: "생산성 도구에 월 80만원씩 나가는데,
      이거 꼭 다 필요해?"

당신: "다들 Notion으로 문서 작업하고,
      1Password로 비밀번호 관리하고..."

대표: "오픈소스 대안 없어?
      계속 이렇게 구독료 내면 연간 1천만원이야"

상황 2: 데이터 주권 문제
보안감사: "회사 기술 문서가 어디에 있나요?"
당신: "Notion 서버에..."

보안감사: "Notion은 미국 서버죠?
         고객사 기밀이 해외 서버에 있는 거예요?"

당신: "...그렇네요"

상황 3: 데이터 Export 제약
팀원: "Notion에서 우리 3년치 문서 백업 받으려는데
      Export하면 형식이 다 깨지네요"

당신: "Notion 전용 형식이라..."
팀원: "나중에 다른 도구로 옮기려면?"
당신: "...어렵습니다"
```

💼 **소규모 조직 관점 - 구독 피로와 종속성**:
- **비용 폭증**: 팀 성장에 따라 구독료 기하급수적 증가
- **데이터 종속**: 플랫폼에 데이터가 묶여 이동 어려움
- **가격 인상**: SaaS 업체가 가격 올리면 어쩔 수 없이 수용
- **보안 우려**: 회사 기밀이 외부 서버에 저장됨

#### 도입 후 변화: 자체 생산성 스택

**비용 비교 (5년 기준)**:

| 도구 | SaaS 구독 | 자체 구축 | 절감 |
|------|----------|----------|------|
| **문서/위키** | Notion $18,000 (5년) | WikiJS $0 | **$18,000** |
| **비밀번호 관리** | 1Password $9,600 (5년) | Vaultwarden $0 | **$9,600** |
| **할 일 관리** | Todoist $7,200 (5년) | Vikunja $0 | **$7,200** |
| **노트 앱** | Evernote $18,000 (5년) | Joplin $0 | **$18,000** |
| **RSS 리더** | Feedly Pro $720 (5년) | FreshRSS $0 | **$720** |
| **총 비용** | **$53,520** | **$0** | **$53,520** |

**실제 활용 사례**:

```
✅ Wiki (지식 관리):
   - 신입 온보딩 가이드
   - API 문서
   - 트러블슈팅 가이드
   - 회의록
   → Notion 대비 연간 $3,600 절감
   → 데이터 완전 통제

✅ Vaultwarden (비밀번호 관리):
   - 팀 계정 공유 (AWS, GitHub, 서버 SSH)
   - 2FA 백업
   - 안전한 비밀번호 생성
   → 1Password 대비 연간 $1,920 절감
   → 자체 서버에 암호화 저장 (해외 서버 X)

✅ Joplin (노트):
   - 개인 업무 노트
   - 회의록
   - 아이디어 메모
   - 모바일 앱으로 동기화
   → Evernote 대비 연간 $3,600 절감

✅ Vikunja (할 일 관리):
   - 개인 작업 관리
   - 프로젝트 보드 (Kanban)
   - 마일스톤 관리
   → Todoist 대비 연간 $1,440 절감
```

**비즈니스 임팩트**:

```
도입 전:
- 연간 구독료: $10,560
- 데이터: 외부 서버에 의존
- 가격 정책: SaaS 업체가 일방적으로 결정
- 데이터 백업: Export 형식 제한적

도입 후:
- 연간 비용: $0
- 데이터: 자체 서버에 완전 통제
- 가격 정책: 없음 (오픈소스)
- 데이터 백업: 전체 데이터베이스 백업 가능

비즈니스 가치:
💰 5년 절감: $53,520 (약 6,700만원)
🔒 보안 강화: 기밀 정보가 해외 서버로 나가지 않음
📈 확장성: 팀 증가해도 비용 증가 없음
🔓 자유: 언제든 다른 도구로 전환 가능 (데이터 소유)
```

**실제 시나리오**:

```
시나리오 1: 신입 온보딩
신입: "회사 시스템 어떻게 쓰나요?"
당신: "Wiki (http://wiki.company.com) 들어가서
      '신입 온보딩' 페이지 보세요"

신입: (30분 후) "다 이해했습니다!"

👉 Notion 구독 없이도 체계적인 문서 관리

시나리오 2: 비밀번호 공유
팀원: "AWS 콘솔 비밀번호 뭐예요?"
당신: "Vaultwarden에서 'AWS Production' 항목 보세요"

팀원: "1Password 계정 없는데요"
당신: "우리는 자체 서버라 계정 만들어드릴게요. 무료예요"

👉 1Password 구독 없이도 안전한 비밀번호 공유

시나리오 3: 프로젝트 관리
PM: "이번 스프린트 작업 어디서 보나요?"
당신: "Vikunja 보드에 다 정리했습니다"

PM: "Jira는 너무 비싸고, Trello는 기능이 부족한데
    Vikunja는 딱 좋네요!"

👉 Jira/Trello 구독 없이도 프로젝트 관리
```

💡 **핵심 가치**:
- **비용 절감**: 5년간 $53,000+ 절약 (약 6,700만원)
- **데이터 주권**: 모든 기업 지식이 자체 서버에 안전하게 보관
- **확장성**: 팀 규모 증가해도 비용 증가 없음
- **통합**: 모든 도구가 한 서버에 있어 연동 쉬움
- **백업**: 전체 시스템 백업 및 복구 용이
- **커스터마이징**: 필요에 따라 자유롭게 수정 가능

---

## 10.1 개인 위키 (WikiJS/BookStack)

### WikiJS

[Wiki.js](https://js.wiki/)는 강력한 오픈소스 위키 시스템입니다.

```yaml
version: '3.8'

services:
  wikijs-db:
    image: postgres:15-alpine
    container_name: wikijs-db
    environment:
      POSTGRES_DB: wiki
      POSTGRES_PASSWORD: wikijsrocks
      POSTGRES_USER: wikijs
    volumes:
      - wikijs-db-data:/var/lib/postgresql/data
    restart: unless-stopped

  wikijs:
    image: requarks/wiki:2
    container_name: wikijs
    depends_on:
      - wikijs-db
    environment:
      DB_TYPE: postgres
      DB_HOST: wikijs-db
      DB_PORT: 5432
      DB_USER: wikijs
      DB_PASS: wikijsrocks
      DB_NAME: wiki
    restart: unless-stopped
    ports:
      - "3003:3000"

volumes:
  wikijs-db-data:
```

초기 설정: `http://homeserver:3003`

### BookStack

[BookStack](https://www.bookstackapp.com/)은 사용하기 쉬운 위키 플랫폼입니다.

```yaml
version: '3.8'

services:
  bookstack-db:
    image: mariadb:10
    container_name: bookstack-db
    environment:
      - MYSQL_ROOT_PASSWORD=secret
      - MYSQL_DATABASE=bookstack
      - MYSQL_USER=bookstack
      - MYSQL_PASSWORD=secret
    volumes:
      - bookstack-db-data:/var/lib/mysql
    restart: unless-stopped

  bookstack:
    image: lscr.io/linuxserver/bookstack:latest
    container_name: bookstack
    environment:
      - PUID=1000
      - PGID=1000
      - APP_URL=http://bookstack.homelab.local
      - DB_HOST=bookstack-db
      - DB_PORT=3306
      - DB_USER=bookstack
      - DB_PASS=secret
      - DB_DATABASE=bookstack
    volumes:
      - ./bookstack:/config
    ports:
      - 6875:80
    restart: unless-stopped
    depends_on:
      - bookstack-db

volumes:
  bookstack-db-data:
```

기본 로그인: `admin@admin.com` / `password`

## 10.2 노트 앱 (Joplin Server)

[Joplin](https://joplinapp.org/)은 Evernote 대체 노트 앱입니다.

```yaml
version: '3.8'

services:
  joplin-db:
    image: postgres:16-alpine
    container_name: joplin-db
    volumes:
      - joplin-db:/var/lib/postgresql/data
    ports:
      - "5432:5432"
    restart: unless-stopped
    environment:
      - POSTGRES_PASSWORD=joplin
      - POSTGRES_USER=joplin
      - POSTGRES_DB=joplin

  joplin:
    image: joplin/server:latest
    container_name: joplin
    depends_on:
      - joplin-db
    ports:
      - "22300:22300"
    restart: unless-stopped
    environment:
      - APP_PORT=22300
      - APP_BASE_URL=https://joplin.example.com
      - DB_CLIENT=pg
      - POSTGRES_PASSWORD=joplin
      - POSTGRES_DATABASE=joplin
      - POSTGRES_USER=joplin
      - POSTGRES_PORT=5432
      - POSTGRES_HOST=joplin-db

volumes:
  joplin-db:
```

데스크톱/모바일 앱에서 동기화 설정.

## 10.3 할 일 관리 (Vikunja)

[Vikunja](https://vikunja.io/)는 Todoist 대체 할 일 관리 도구입니다.

```yaml
version: '3.8'

services:
  vikunja-db:
    image: postgres:16-alpine
    container_name: vikunja-db
    environment:
      POSTGRES_PASSWORD: secret
      POSTGRES_USER: vikunja
      POSTGRES_DB: vikunja
    volumes:
      - vikunja-db:/var/lib/postgresql/data
    restart: unless-stopped

  vikunja:
    image: vikunja/vikunja:latest
    container_name: vikunja
    environment:
      VIKUNJA_DATABASE_HOST: vikunja-db
      VIKUNJA_DATABASE_PASSWORD: secret
      VIKUNJA_DATABASE_TYPE: postgres
      VIKUNJA_DATABASE_USER: vikunja
      VIKUNJA_DATABASE_DATABASE: vikunja
      VIKUNJA_SERVICE_JWTSECRET: random_secret_key
      VIKUNJA_SERVICE_FRONTENDURL: http://homeserver:3456
    ports:
      - 3456:3456
    volumes:
      - vikunja-files:/app/vikunja/files
    depends_on:
      - vikunja-db
    restart: unless-stopped

volumes:
  vikunja-db:
  vikunja-files:
```

## 10.4 비밀번호 관리자 (Vaultwarden)

[Vaultwarden](https://github.com/dani-garcia/vaultwarden)은 Bitwarden 호환 비밀번호 관리자입니다.

```yaml
version: '3.8'

services:
  vaultwarden:
    image: vaultwarden/server:latest
    container_name: vaultwarden
    restart: unless-stopped
    environment:
      - WEBSOCKET_ENABLED=true
      - SIGNUPS_ALLOWED=true
      - INVITATIONS_ALLOWED=true
      - DOMAIN=https://vault.example.com
      - SMTP_HOST=smtp.gmail.com
      - SMTP_FROM=your-email@gmail.com
      - SMTP_PORT=587
      - SMTP_SECURITY=starttls
      - SMTP_USERNAME=your-email@gmail.com
      - SMTP_PASSWORD=your-app-password
    volumes:
      - ./vaultwarden-data:/data
    ports:
      - 8087:80
      - 3012:3012
```

**보안 권장사항**:
- HTTPS 필수 (Nginx Proxy Manager로 설정)
- 2FA 활성화
- 첫 계정 생성 후 `SIGNUPS_ALLOWED=false`로 변경

## 10.5 RSS 리더 (FreshRSS)

[FreshRSS](https://freshrss.org/)로 뉴스 피드 통합 관리:

```yaml
version: '3.8'

services:
  freshrss-db:
    image: postgres:16-alpine
    container_name: freshrss-db
    environment:
      POSTGRES_USER: freshrss
      POSTGRES_PASSWORD: freshrss
      POSTGRES_DB: freshrss
    volumes:
      - freshrss-db:/var/lib/postgresql/data
    restart: unless-stopped

  freshrss:
    image: freshrss/freshrss:latest
    container_name: freshrss
    restart: unless-stopped
    ports:
      - 8084:80
    environment:
      - TZ=Asia/Seoul
      - CRON_MIN=*/15
    volumes:
      - ./freshrss/data:/var/www/FreshRSS/data
      - ./freshrss/extensions:/var/www/FreshRSS/extensions
    depends_on:
      - freshrss-db

volumes:
  freshrss-db:
```

웹 UI: `http://homeserver:8084`

모바일 앱:
- iOS: Reeder, NetNewsWire
- Android: FeedMe, News+

---

**다음 장에서는**: Home Assistant를 활용한 스마트홈 자동화 시스템을 구축해보겠습니다.
