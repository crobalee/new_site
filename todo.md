# 모바일 익명 게시판 (MVP)

**Version:** 1.0.0

Firebase 일체형 아키텍처(React+Vite SPA, Firestore, Firebase Functions v2) 기반의 익명 게시판 MVP. 로그인/닉네임 없이 글 목록/상세/작성/신고 및 보안(App Check, reCAPTCHA), 레이트리밋과 자동 숨김 정책을 포함하여 1주 내 배포를 목표로 한다.

## 목차

- [프로젝트 개요](#프로젝트-개요)
  - [프로젝트 현황](#프로젝트-현황)
  - [우선순위별 작업 분포](#우선순위별-작업-분포)
- [작업 목록](#작업-목록)
- [구현 가이드](#구현-가이드)
- [전역 고려사항](#전역-고려사항)
- [실행 순서](#실행-순서)
- [메타데이터](#메타데이터)

## 프로젝트 개요

Firebase 일체형 아키텍처(React+Vite SPA, Firestore, Firebase Functions v2) 기반의 익명 게시판 MVP. 로그인/닉네임 없이 글 목록/상세/작성/신고 및 보안(App Check, reCAPTCHA), 레이트리밋과 자동 숨김 정책을 포함하여 1주 내 배포를 목표로 한다.

### 프로젝트 현황

- **전체 작업:** 30개
- **세부 작업:** 35개

### 우선순위별 작업 분포

- **P0 (필수):** 29개
- **P1 (중요):** 1개
- **P2 (선택):** 0개

## 작업 목록

### Firebase 프로젝트/앱 설정 및 환경 준비 (task-setup-firebase-project)

- **우선순위:** P0
- **상태:** not_started
- **실행 순서:** 1

**설명:**
Firebase 콘솔에서 dev/prod 프로젝트를 생성하고 Firestore, Hosting, Functions, App Check(reCAPTCHA v3) 활성화. 로컬 개발자 환경(Firebase CLI, Node 20, pnpm/npm) 준비.

**관련 파일:**
- `.firebaserc` (create): dev/prod 별 Firebase 프로젝트 alias 등록
- `firebase.json` (create): Hosting/Functions/Emulators 설정의 기본 골격
- `.github/workflows/deploy.yml` (create): CI 파이프라인 자리표시자(workflow scaffold)
- `docs/SETUP.md` (create): 프로젝트/환경 설정 절차 문서

**세부 작업:**

#### Firebase 콘솔 리소스 생성 (setup-firebase-console)

- **카테고리:** 운영
**설명:** Project 2개(dev, prod), Firestore 네이티브 모드, Hosting, Functions v2 활성화. App Check(reCAPTCHA v3 사이트 키) 등록.

**구현 참고사항:**
- App Check: Web App에 reCAPTCHA v3 연결 후 강제 모드엔 Functions에서 enforceAppCheck 사용

#### 로컬 도구 설치 (setup-local-cli)

- **카테고리:** 운영
**설명:** Node.js 20 LTS, Firebase CLI, pnpm/npm 설치 및 로그인

**구현 참고사항:**
- firebase login
- firebase use dev

**완료 기준:**
- Firebase 콘솔에 dev/prod 두 프로젝트 존재
- App Check 사이트 키 발급 완료
- CLI에서 firebase projects:list 확인

**위험 요소:**
- **위험:** App Check site key 오구성
- **완화 방안:** 환경변수와 콘솔 키 값 상호 검증, 샘플 호출로 403 기대 동작 확인

---

### 모노레포 구조 초기화 (task-repo-init)

- **우선순위:** P0
- **상태:** not_started
- **실행 순서:** 2

**설명:**
apps/web(React+Vite+TS), functions(Firebase Functions v2+TS) 디렉터리 구조 생성 및 공통 루트 설정.

**종속성:**
- task-setup-firebase-project

**관련 파일:**
- `apps/web/package.json` (create): 웹 앱 종속성/스크립트
- `apps/web/tsconfig.json` (create): 웹 TypeScript 설정
- `apps/web/index.html` (create): 루트 HTML 및 reCAPTCHA v2 스크립트 로더 자리
- `apps/web/src/main.tsx` (create): React 진입점
- `apps/web/src/app/App.tsx` (create): App Shell 및 라우팅 골격
- `apps/web/tailwind.config.js` (create): Tailwind 설정
- `apps/web/postcss.config.js` (create): Tailwind/PostCSS 설정
- `apps/web/src/styles/index.css` (create): Tailwind base 스타일
- `functions/package.json` (create): Functions 종속성/스크립트
- `functions/tsconfig.json` (create): Functions TS 설정
- `functions/src/index.ts` (create): Functions v2 export 엔트리

**세부 작업:**

#### Git 초기화 (repo-init-git)

- **카테고리:** 내부구현
**설명:** git init, 기본 .gitignore 추가(node_modules, build)

**구현 참고사항:**
- 루트 README.md 작성
- Commit: chore: repo scaffold

**완료 기준:**
- 로컬에서 웹/함수 빌드 스크립트가 동작
- 디렉터리 구조가 TRD 구조와 일치

---

### CI/CD 스캐폴드 구성 (task-ci-cd-setup)

- **우선순위:** P0
- **상태:** not_started
- **실행 순서:** 3

**설명:**
GitHub Actions 워크플로우 기본 구축. main push 시 웹 빌드/테스트/Hosting Preview 및 Functions를 스테이징에 배포. 승인 후 prod 배포 단계 포함.

**종속성:**
- task-repo-init

**관련 파일:**
- `.github/workflows/deploy.yml` (modify): 빌드 → 테스트 → 프리뷰 → 스테이징 배포 → 승인 후 프로덕션

**세부 작업:**

#### CI 인증 구성 (ci-setup-service-accounts)

- **카테고리:** 운영
**설명:** Firebase CI 토큰 또는 Workload Identity 설정

**구현 참고사항:**
- Firebase CLI token을 GitHub Secret에 저장
- 프로젝트별 target 설정

**완료 기준:**
- main 브랜치 push 시 Preview URL 생성
- 수동 승인 후 prod 배포 단계 표시

**위험 요소:**
- **위험:** CI 권한 부족으로 배포 실패
- **완화 방안:** 서비스 계정 권한 firebasehosting.admin, cloudfunctions.admin 확인

---

### Firebase Emulator Suite 설정 (task-emulator-setup)

- **우선순위:** P0
- **상태:** not_started
- **실행 순서:** 4

**설명:**
Functions, Firestore 에뮬레이터 설정 및 로컬 개발 스크립트 구성.

**종속성:**
- task-repo-init

**관련 파일:**
- `firebase.json` (modify): emulators 섹션 추가(Functions, Firestore), Hosting rewrite 프록시
- `firestore.rules` (create): Firestore 보안 규칙
- `firestore.indexes.json` (create): 필수/보조 인덱스 정의

**세부 작업:**

#### 로컬 실행 스크립트 (emulators-start-script)

- **카테고리:** 내부구현
**설명:** web dev 서버와 emulators를 동시 기동하는 npm script 추가(concurrently)

**구현 참고사항:**
- pnpm -w script: dev:web, emulators:start

**완료 기준:**
- firebase emulators:start 시 Functions/Firestore 기동
- apps/web 로컬에서 /api/health 프록시 접근

---

### Functions v2 + Express 스캐폴드 (task-functions-scaffold)

- **우선순위:** P0
- **상태:** not_started
- **실행 순서:** 5

**설명:**
Express 앱 생성, /api 베이스 라우팅, v2 https.onRequest로 내보내기. trust proxy=1 설정.

**종속성:**
- task-emulator-setup

**관련 파일:**
- `functions/src/app.ts` (create): Express 인스턴스, 공통 미들웨어 적용, 라우터 마운트
- `functions/src/handlers/health.ts` (create): GET /health 핸들러
- `functions/src/index.ts` (modify): v2 https.onRequest({enforceAppCheck:true})로 app export

**세부 작업:**

#### Express 초기화 (express-init)

- **카테고리:** 내부구현
**설명:** app.set('trust proxy', 1), /health 라우트 연결

**구현 참고사항:**
- CORS는 사용하지 않고 Origin Guard로 제한

**완료 기준:**
- 에뮬레이터에서 GET /api/health → {ok:true, ts:number}
- App Check 미설정 시 403(프로덕션 옵션) 확인

---

### 핵심 미들웨어 구현(appCheck/origin/json/schema) (task-middleware-core)

- **우선순위:** P0
- **상태:** not_started
- **실행 순서:** 6

**설명:**
App Check(v2 옵션), Origin 화이트리스트 검사, Content-Type 검사 및 JSON 파싱, 요청 스키마 길이/형식 체크 구현.

**종속성:**
- task-functions-scaffold

**관련 파일:**
- `functions/src/middleware/appCheckGuard.ts` (create): App Check 유효성 확인(보조, v2 옵션과 병행)
- `functions/src/middleware/originGuard.ts` (create): ORIGIN_WHITELIST 기반 Origin 검증
- `functions/src/middleware/jsonGuard.ts` (create): Content-Type, JSON 바디 파서 및 크기 제한
- `functions/src/middleware/schemaValidator.ts` (create): 요청 스키마 수동 검증 유틸(길이/enum)

**세부 작업:**

#### Origin 화이트리스트 검사 (origin-whitelist)

- **카테고리:** 보안
**설명:** 환경변수 ORIGIN_WHITELIST(쉼표분리)와 req.headers.origin 매칭

**구현 참고사항:**
- 불일치 시 403 반환(APP_CHECK_FAILED 준용 아님, 별도 code 사용 가능)

#### 스키마 검증기 (schema-validators)

- **카테고리:** API
**설명:** posts/create 및 reports/create 입력값 길이/형식/enum 검사

**구현 참고사항:**
- 400 VALIDATION_ERROR와 메시지 표준 매핑

**완료 기준:**
- 유효하지 않은 Origin에서 요청 시 403
- 잘못된 Content-Type 또는 스키마 위반 시 400

---

### Firestore 규칙 구성 (task-firestore-rules)

- **우선순위:** P0
- **상태:** not_started
- **실행 순서:** 7

**설명:**
posts 읽기 공개, posts/reports/rateLimits 쓰기 전면 차단 규칙 작성.

**종속성:**
- task-emulator-setup

**관련 파일:**
- `firestore.rules` (modify): TRD 명시 규칙 반영

**세부 작업:**

#### 규칙 시뮬레이션 (rules-simulate)

- **카테고리:** 유효성검사
**설명:** 에뮬레이터에서 클라이언트 SDK로 쓰기 시도 실패, 읽기 성공 확인

**구현 참고사항:**
- allow read: true on posts, others deny all

**완료 기준:**
- 클라이언트에서 posts 읽기 OK
- 모든 컬렉션에 대한 클라이언트 write 거부

---

### Firestore 인덱스 정의 (task-firestore-indexes)

- **우선순위:** P0
- **상태:** not_started
- **실행 순서:** 8

**설명:**
posts: status ASC + createdAt DESC 복합 인덱스, createdAt DESC 단일 인덱스(필요 시). 신고 중복 검사 최적화를 위해 reports: postId ASC + reporterHash ASC + createdAt DESC 복합 인덱스 추가.

**종속성:**
- task-emulator-setup

**관련 파일:**
- `firestore.indexes.json` (modify): 인덱스 스키마 반영

**세부 작업:**

#### 인덱스 배포 및 준비 대기 (deploy-indexes)

- **카테고리:** 운영
**설명:** firebase deploy --only firestore:indexes 실행 후 준비 완료 확인

**구현 참고사항:**
- 콘솔에서 빌드 완료 상태 확인

**완료 기준:**
- 목록 쿼리 p95 < 300ms
- 신고 중복 검사 쿼리 p95 < 300ms

**위험 요소:**
- **위험:** 인덱스 빌드 지연
- **완화 방안:** 개발 초기 배포 후 빌드 대기, 에뮬레이터로 우회 테스트

---

### 서버 서비스 유틸 구현(recaptcha/ratelimit/hash/firestore/contentPolicy/config) (task-services-base)

- **우선순위:** P0
- **상태:** not_started
- **실행 순서:** 9

**설명:**
서버 내부 서비스 모듈 구현: reCAPTCHA 검증, 레이트리밋, SHA-256 해시, Firestore 래퍼, 콘텐츠 정책 검증, 설정 로더.

**종속성:**
- task-middleware-core
- task-firestore-indexes

**관련 파일:**
- `functions/src/services/recaptcha.ts` (create): Google verify API 호출(undici 또는 fetch) 및 점수/체크박스 검증
- `functions/src/services/ratelimit.ts` (create): sliding window + daily cap 구현(rateLimits 컬렉션)
- `functions/src/services/hash.ts` (create): SHA-256 해시 유틸(crypto)
- `functions/src/services/firestore.ts` (create): CRUD/트랜잭션 헬퍼, posts/reports 접근 래퍼
- `functions/src/services/contentPolicy.ts` (create): 금지어/길이 정책 검사
- `functions/src/types/api.ts` (create): 요청/응답 타입 정의
- `functions/src/config.ts` (create): 환경변수/정책 상수 로딩

**세부 작업:**

#### reCAPTCHA 서버 검증 (recaptcha-verify)

- **카테고리:** API
**설명:** secret, token, remoteip로 verify, 실패 시 403 코드(CAPTCHA_FAILED)

**구현 참고사항:**
- Node 20 fetch 사용 가능
- v2 checkbox는 success boolean만 확인

#### 레이트리밋 구현 (ratelimit-impl)

- **카테고리:** 데이터처리
**설명:** type별(windowSec, dailyCap) 체크. 문서 구조: rateLimits/{key_type}에 windowEnd, count, dayCount, dayKey, ttlAt.

**구현 참고사항:**
- now > windowEnd면 window 리셋
- dayKey=YYYYMMDD, dayCount 업데이트
- ttlAt=now+7d 설정(프로젝트 TTL 활성화 필요)

#### 해시 유틸 (hashing)

- **카테고리:** 보안
**설명:** reporterHash = SHA256(ip + ua + salt), rateKey = SHA256(ip + salt)

**구현 참고사항:**
- salt는 POLICY_SALT 사용, 32바이트 이상

**완료 기준:**
- unit 테스트로 성공/실패 케이스 통과
- TTL 필드 설정 및 문서 생성 시 반영

**위험 요소:**
- **위험:** 동시성으로 인한 rateLimits 경합
- **완화 방안:** runTransaction으로 CAS 보장, 재시도 로직 포함

---

### POST /posts/create 핸들러 (task-endpoint-posts-create)

- **우선순위:** P0
- **상태:** not_started
- **실행 순서:** 10

**설명:**
글 생성 API 구현: 스키마/콘텐츠 정책/레이트리밋/캡차 검증 후 posts 문서 생성.

**종속성:**
- task-services-base

**관련 파일:**
- `functions/src/handlers/postsCreate.ts` (create): 핸들러 구현 및 라우팅 연결
- `functions/src/app.ts` (modify): /posts/create 라우트 마운트

**세부 작업:**

#### 입력/정책 검증 (posts-validate)

- **카테고리:** API
**설명:** title/body 길이, 금지어, captchaVerifier, rateLimiter(post) 호출

**구현 참고사항:**
- 에러 코드 맵핑: 400/403/429

#### Firestore 문서 생성 (posts-create-firestore)

- **카테고리:** 데이터처리
**설명:** createdAt=serverTimestamp, status='active', reportCount=0

**구현 참고사항:**
- 성공 시 {id} 반환

**완료 기준:**
- 정상 요청 p95 <300ms 응답
- 레이트리밋/캡차 실패 시 적절한 오류 코드 반환

---

### POST /reports/create 핸들러 (task-endpoint-reports-create)

- **우선순위:** P0
- **상태:** not_started
- **실행 순서:** 11

**설명:**
신고 API 구현: post 존재/상태 확인, 중복 신고 제한(reporterHash+post+day), 레이트리밋(report) 적용, reportCount 증가 및 임계치 초과 시 status='removed'.

**종속성:**
- task-services-base

**관련 파일:**
- `functions/src/handlers/reportsCreate.ts` (create): 핸들러 구현 및 라우팅 연결
- `functions/src/app.ts` (modify): /reports/create 라우트 마운트

**세부 작업:**

#### 중복 신고 제한 (reports-dedupe)

- **카테고리:** 데이터처리
**설명:** 트랜잭션에서 reporterHash, postId로 당일(createdAt>=00:00) 존재 여부 조회 후 있으면 409 반환

**구현 참고사항:**
- 복합 인덱스(reports: postId+reporterHash+createdAt) 사용

#### reportCount 증가 및 자동 숨김 (reports-increment)

- **카테고리:** 데이터처리
**설명:** reports 생성 후 posts.reportCount 증가, 임계치 이상 시 status='removed'

**구현 참고사항:**
- runTransaction으로 원자성 보장
- 阈値: REPORT_HIDE_THRESHOLD

**완료 기준:**
- 중복 신고 시 409 DUPLICATE_REPORT
- 임계치 도달 시 해당 post status='removed'로 변경

**위험 요소:**
- **위험:** 트랜잭션 충돌
- **완화 방안:** Firestore 트랜잭션 재시도 허용

---

### 서버 로깅/모니터링 훅 (task-logging-monitoring)

- **우선순위:** P1
- **상태:** not_started
- **실행 순서:** 12

**설명:**
요청 ID, IP 해시, 결과 코드, 처리 시간 로깅. CAPTCHA 실패/레이트리밋 트리거 카운트.

**종속성:**
- task-functions-scaffold

**관련 파일:**
- `functions/src/middleware/requestLogger.ts` (create): 요청 전/후 시간 측정 및 구조화 로그

**세부 작업:**

#### 로거 연결 (logger-integration)

- **카테고리:** 내부구현
**설명:** app.use(requestLogger), Google Cloud Logging 포맷

**구현 참고사항:**
- console.log JSON 직렬화

**완료 기준:**
- Cloud Logging에서 코드/지연시간 필드 조회 가능
- CAPTCHA 실패/429 발생 카운트 확인 가능

---

### Functions 단위 테스트 (task-tests-unit-functions)

- **우선순위:** P0
- **상태:** not_started
- **실행 순서:** 13

**설명:**
콘텐츠 정책, 레이트리밋, 캡차 분기, 신고 중복 제한 단위 테스트.

**종속성:**
- task-endpoint-posts-create
- task-endpoint-reports-create

**관련 파일:**
- `functions/test/contentPolicy.test.ts` (create): 금지어/길이 테스트
- `functions/test/ratelimit.test.ts` (create): 윈도우/데일리 캡 테스트
- `functions/test/reports.test.ts` (create): 중복/임계치 테스트

**세부 작업:**

#### 에뮬레이터 기반 테스트 (emulator-tests)

- **카테고리:** 유효성검사
**설명:** Firestore 에뮬레이터를 이용한 CRUD/트랜잭션 검증

**구현 참고사항:**
- test 환경변수로 캡차 verify 목(mock) 주입

**완료 기준:**
- 핵심 테스트 케이스 전부 통과
- CI에서 테스트 단계 성공

---

### Hosting SPA/리라이트/캐시 헤더 구성 (task-hosting-config)

- **우선순위:** P0
- **상태:** not_started
- **실행 순서:** 14

**설명:**
SPA rewrite(all → /index.html), API rewrite(/api/** → functions). HTML no-cache, assets immutable 캐싱 헤더.

**종속성:**
- task-ci-cd-setup
- task-web-scaffold

**관련 파일:**
- `firebase.json` (modify): hosting rewrites/headers 설정

**세부 작업:**

#### Hosting Preview 배포 (hosting-deploy-preview)

- **카테고리:** 운영
**설명:** Preview 채널 배포 및 헤더 동작 확인

**구현 참고사항:**
- Cache-Control: public, max-age=0, must-revalidate (HTML)
- Cache-Control: public, max-age=31536000, immutable (assets)

**완료 기준:**
- SPA 라우팅 정상
- /api/** 가 Functions로 프록시

---

### 웹 앱 스캐폴드(React+Vite+TS+Tailwind+Router) (task-web-scaffold)

- **우선순위:** P0
- **상태:** not_started
- **실행 순서:** 15

**설명:**
라우팅(/, /posts/:id, /compose, /guidelines), 레이아웃, 기본 스타일, Skeleton 컴포넌트 틀.

**종속성:**
- task-repo-init

**관련 파일:**
- `apps/web/vite.config.ts` (create): Vite 설정 및 코드 스플리팅 옵션
- `apps/web/src/app/router.tsx` (create): React Router 라우팅 정의
- `apps/web/src/components/Layout/Header.tsx` (create): 타이틀/글쓰기 버튼
- `apps/web/src/components/Skeleton.tsx` (create): 로딩 Skeleton UI

**세부 작업:**

#### Tailwind 설정 (tailwind-setup)

- **카테고리:** UI
**설명:** 폰트 최소 14px, 포커스 링 표준 유지

**구현 참고사항:**
- focus-visible 스타일 유지
- prefers-reduced-motion 고려

**완료 기준:**
- 라우팅 이동 가능
- 레이아웃/스켈레톤 표시

---

### App Check(reCAPTCHA v3) 초기화 (task-appcheck-init)

- **우선순위:** P0
- **상태:** not_started
- **실행 순서:** 16

**설명:**
클라이언트 앱에서 Firebase App Check 초기화 및 강제 사용.

**종속성:**
- task-web-scaffold

**관련 파일:**
- `apps/web/src/services/firebase.ts` (create): Firebase 앱/App Check 초기화

**세부 작업:**

#### App Check 구성 (appcheck-configure)

- **카테고리:** 보안
**설명:** VITE_APPCHECK_SITE_KEY 사용, 자동 토큰 갱신

**구현 참고사항:**
- self SDK provider: ReCaptchaV3Provider

**완료 기준:**
- 유효 토큰이 요청 헤더에 포함
- 토큰 미존재 시 서버 403 동작

---

### Firestore 읽기 래퍼/쿼리 구현 (task-firestore-reads-wrapper)

- **우선순위:** P0
- **상태:** not_started
- **실행 순서:** 17

**설명:**
목록/상세 읽기 전용 쿼리 래퍼 구현. 커서 기반 페이징(startAfter).

**종속성:**
- task-appcheck-init
- task-firestore-indexes

**관련 파일:**
- `apps/web/src/services/firestore.ts` (create): getPostsPage, getPostById 구현
- `apps/web/src/types/models.ts` (create): Post/Report 타입 정의

**세부 작업:**

#### 커서 기반 페이징 (pagination-cursor)

- **카테고리:** 데이터처리
**설명:** status=='active' 필터 + createdAt desc + limit 20 + startAfter

**구현 참고사항:**
- 빈 상태/오류 throw로 상위에서 처리

**완료 기준:**
- 최초 20개, 더보기 시 다음 20개
- 중복 없이 커서 진행

---

### Functions API 래퍼(web) (task-api-wrapper)

- **우선순위:** P0
- **상태:** not_started
- **실행 순서:** 18

**설명:**
/api/posts/create, /api/reports/create 호출 래퍼 구현. 공통 응답 포맷 처리 및 오류 맵핑.

**종속성:**
- task-web-scaffold

**관련 파일:**
- `apps/web/src/services/api.ts` (create): fetch 래퍼, 오류 코드 한국어 메시지 매핑

**세부 작업:**

#### 에러 메시지 표준 맵핑 (api-error-map)

- **카테고리:** API
**설명:** VALIDATION_ERROR/429/409/403/404/500 → 한국어 문구

**구현 참고사항:**
- TRD 19번 기준

**완료 기준:**
- 성공/오류 케이스에서 일관된 메시지 반환

---

### 목록 페이지 구현 (task-list-page)

- **우선순위:** P0
- **상태:** not_started
- **실행 순서:** 19

**설명:**
최신순 20개 로드, 더보기 버튼으로 다음 20개 로드, Skeleton/빈 상태/오류 재시도.

**종속성:**
- task-firestore-reads-wrapper

**관련 파일:**
- `apps/web/src/pages/ListPage.tsx` (create): 목록 페이지(페이징/더보기)
- `apps/web/src/components/PostListItem.tsx` (create): 제목 1줄/본문 2줄 프리뷰/상대시각(dayjs)

**세부 작업:**

#### Skeleton/재시도 (list-skeleton-retry)

- **카테고리:** UI
**설명:** 로딩 Skeleton 3–5개, 오류 시 재시도 버튼

**구현 참고사항:**
- 중복 제거: 마지막 커서 기준 startAfter 유지

**완료 기준:**
- 0건 시 빈 문구 표시
- 더보기 클릭 시 중복 없음

---

### 상세 페이지 구현 (task-detail-page)

- **우선순위:** P0
- **상태:** not_started
- **실행 순서:** 20

**설명:**
문서 fetch 후 status 검사. 제목/본문/작성시각 표시. 미존재/removed 시 404 UI.

**종속성:**
- task-firestore-reads-wrapper

**관련 파일:**
- `apps/web/src/pages/DetailPage.tsx` (create): 상세 페이지

**세부 작업:**

#### 404 처리 (detail-404)

- **카테고리:** UI
**설명:** 미존재/removed 시 404 안내 및 목록 이동 버튼

**구현 참고사항:**
- 새로고침에도 동일 URL 접근 가능

**완료 기준:**
- 정상 문서 표시/404 케이스 정상
- 상대 시각 표기

---

### 글쓰기 페이지 및 reCAPTCHA v2 연동 (task-compose-page)

- **우선순위:** P0
- **상태:** not_started
- **실행 순서:** 21

**설명:**
제목/내용 폼, 길이/금지어 클라 검증, reCAPTCHA v2(체크박스) 토큰 획득 후 /api/posts/create 호출.

**종속성:**
- task-api-wrapper

**관련 파일:**
- `apps/web/src/pages/ComposePage.tsx` (create): 글쓰기 폼 및 제출 흐름
- `apps/web/src/services/validators.ts` (create): 클라이언트 길이/금지어/스키마 검사 유틸

**세부 작업:**

#### reCAPTCHA v2 위젯 (captcha-v2-widget)

- **카테고리:** 보안
**설명:** grecaptcha.render/execute로 토큰 획득

**구현 참고사항:**
- VITE_RECAPTCHA_V2_SITE_KEY 사용
- 토큰 미존재 시 제출 비활성화

**완료 기준:**
- 필수/길이 오류 즉시 표시
- 성공 시 상세 페이지로 이동
- 레이트리밋 초과 시 안내 문구

---

### 신고 모달 구현 (task-report-modal)

- **우선순위:** P0
- **상태:** not_started
- **실행 순서:** 22

**설명:**
사유 선택 라디오, reCAPTCHA v2 토큰 획득 후 /api/reports/create 호출. 성공/실패 토스트.

**종속성:**
- task-api-wrapper
- task-detail-page

**관련 파일:**
- `apps/web/src/components/ReportModal.tsx` (create): 모달 UI/로직

**세부 작업:**

#### 사유 필수 검사 (report-validation)

- **카테고리:** UI
**설명:** reason enum 체크 및 비활성화 상태 처리

**구현 참고사항:**
- 중복 신고(409) 시 안내

**완료 기준:**
- 성공 토스트 표시
- 중복/제한 시 오류 토스트

---

### 이용 안내/고지 (task-guidelines)

- **우선순위:** P0
- **상태:** not_started
- **실행 순서:** 23

**설명:**
상/하단 링크 항상 노출, 모달 또는 /guidelines 라우트에 내용 제공.

**종속성:**
- task-web-scaffold

**관련 파일:**
- `apps/web/src/pages/GuidelinesPage.tsx` (create): 가이드라인 콘텐츠
- `apps/web/src/components/GuidelinesLink.tsx` (create): 고정 링크 컴포넌트

**세부 작업:**

#### 접근성 라벨 (guidelines-accessibility)

- **카테고리:** 접근성
**설명:** 모달 접근성 속성(aria-labelledby/aria-describedby) 설정

**구현 참고사항:**
- 키보드 포커스 트랩

**완료 기준:**
- 모든 페이지에서 링크 접근 가능
- 모달 내 텍스트 제공

---

### 토스트/에러 바운더리/재시도 (task-toast-error-boundary)

- **우선순위:** P0
- **상태:** not_started
- **실행 순서:** 24

**설명:**
글쓰기/신고 성공/실패 메시지 토스트, ErrorBoundary로 네트워크 오류 재시도 제공.

**종속성:**
- task-api-wrapper

**관련 파일:**
- `apps/web/src/components/Toast.tsx` (create): 토스트/알림 컴포넌트
- `apps/web/src/components/ErrorBoundary.tsx` (create): 에러 바운더리 및 재시도 UI

**세부 작업:**

#### 오류 코드→문구 맵 (error-messages-map)

- **카테고리:** UX
**설명:** TRD 19 표준에 따른 한국어 메시지 표시

**구현 참고사항:**
- API 래퍼와 메시지 일관성

**완료 기준:**
- 서버 오류/검증 오류 케이스에서 올바른 문구 표시
- 재시도 동작

---

### GA4 분석 이벤트 연동 (task-analytics-ga4)

- **우선순위:** P0
- **상태:** not_started
- **실행 순서:** 25

**설명:**
gtag.js 초기화 및 page_view, post/report attempt/success/fail, pagination_click 이벤트 전송.

**종속성:**
- task-web-scaffold

**관련 파일:**
- `apps/web/src/services/analytics.ts` (create): GA4 초기화/이벤트 유틸

**세부 작업:**

#### 주요 이벤트 계측 (analytics-instrument)

- **카테고리:** 데이터처리
**설명:** 페이지 전환, 더보기 클릭, 작성/신고 시도/성공/실패

**구현 참고사항:**
- 전송 성공률 >95% 모니터

**완료 기준:**
- GA4 실시간 대시보드에 이벤트 수신 확인

---

### 접근성 준수(WCAG AA 대비) (task-accessibility)

- **우선순위:** P0
- **상태:** not_started
- **실행 순서:** 26

**설명:**
폰트 ≥14px, 포커스 링, 스크린리더 라벨, 키보드 탐색.

**종속성:**
- task-list-page
- task-detail-page
- task-compose-page
- task-report-modal
- task-guidelines

**세부 작업:**

#### Axe/WAVE 스캔 (axe-scan)

- **카테고리:** 접근성
**설명:** 기본 위반 0 확인

**구현 참고사항:**
- aria-* 속성 점검
- 색 대비 확인

**완료 기준:**
- 키보드 탭 포커스 가능
- 기본 위반 0

---

### 성능 최적화(FCP/TTI 목표) (task-performance-tuning)

- **우선순위:** P0
- **상태:** not_started
- **실행 순서:** 27

**설명:**
코드 스플리팅(라우트), 프리로드, Skeleton 300ms 이내 노출, 의존성 최소화.

**종속성:**
- task-web-scaffold

**세부 작업:**

#### 라우트 레벨 코드 스플리팅 (split-routes)

- **카테고리:** 성능
**설명:** List/Detail/Compose 지연 로딩

**구현 참고사항:**
- Vite dynamic import

**완료 기준:**
- p75 FCP <1.5s, TTI <2.5s (4G 프로파일에서)
- 최초 JS <120KB gzip 목표 충족

---

### 웹 단위/통합 테스트 (task-web-tests)

- **우선순위:** P0
- **상태:** not_started
- **실행 순서:** 28

**설명:**
폼 검증/목록 페이징/상세 404/토스트/재시도 테스트(vitest).

**종속성:**
- task-list-page
- task-detail-page
- task-compose-page
- task-report-modal

**관련 파일:**
- `apps/web/src/pages/__tests__/ListPage.test.tsx` (create): 목록/더보기/중복 없음
- `apps/web/src/pages/__tests__/DetailPage.test.tsx` (create): 상세/404 처리
- `apps/web/src/pages/__tests__/ComposePage.test.tsx` (create): 폼 검증/성공 플로우

**세부 작업:**

#### Playwright 스모크(E2E 선택) (playwright-smoke)

- **카테고리:** 유효성검사
**설명:** 접속→목록→상세→글쓰기→상세 이동→신고 흐름

**구현 참고사항:**
- 선택(P0 내 스모크 최소화)

**완료 기준:**
- 주요 테스트 케이스 통과
- CI에서 웹 테스트 성공

---

### firebase.json API/SPA rewrite 및 헤더 최종화 (task-firebase-json-rewrites)

- **우선순위:** P0
- **상태:** not_started
- **실행 순서:** 29

**설명:**
API와 SPA 라우팅/캐시 정책을 최종 반영.

**종속성:**
- task-hosting-config

**관련 파일:**
- `firebase.json` (modify): rewrites, headers, ignore, public 디렉토리 설정

**완료 기준:**
- 모든 경로가 SPA로 동작, /api/**는 Functions 경유

---

### 프로덕션 배포 및 스모크 테스트 (task-prod-deploy-smoke)

- **우선순위:** P0
- **상태:** not_started
- **실행 순서:** 30

**설명:**
prod 환경으로 Hosting+Functions 배포. KPI 체크리스트 기반 스모크 수행.

**종속성:**
- task-web-tests
- task-tests-unit-functions
- task-performance-tuning
- task-accessibility
- task-logging-monitoring

**세부 작업:**

#### KPI 체크 (kpi-checklist)

- **카테고리:** 유효성검사
**설명:** 성능/접근성/보안/기능 기준 충족 확인

**구현 참고사항:**
- Lighthouse 4G, Cloud Monitoring p95

**완료 기준:**
- KPI 수용 기준 충족
- 롤백 전략 검증

**위험 요소:**
- **위험:** 프로덕션 환경변수 누락
- **완화 방안:** 배포 전 config:list 체크, 부트 시 유효성 검사

---

## 구현 가이드

### Firestore 커서 기반 페이지네이션 (guide-firestore-pagination)

**카테고리:** 데이터처리

status=='active' 필터 + createdAt desc 정렬에서 limit(20) 후 마지막 문서를 커서로 보관하여 startAfter로 다음 페이지를 질의한다. 동일 쿼리 파라미터를 유지해야 중복을 방지할 수 있다.

**관련 작업:**
- task-firestore-reads-wrapper
- task-list-page

**코드 예시:**

**커서 유지 및 다음 페이지 로드**

```typescript
import { collection, query, where, orderBy, limit, startAfter, getDocs } from 'firebase/firestore';

async function getPostsPage(db, after?: any) {
  const base = [
    where('status', '==', 'active'),
    orderBy('createdAt', 'desc'),
    limit(20)
  ];
  const q = after ? query(collection(db, 'posts'), ...base, startAfter(after)) : query(collection(db, 'posts'), ...base);
  const snap = await getDocs(q);
  const items = snap.docs.map(d => ({ id: d.id, ...d.data() }));
  const nextCursor = snap.docs[snap.docs.length - 1];
  return { items, nextCursor };
}
```

---

### Functions v2 App Check 강제 (guide-functions-app-check)

**카테고리:** 보안

https.onRequest 핸들러 생성 시 enforceAppCheck:true 옵션을 설정하면 유효한 App Check 토큰 없는 요청은 403으로 거부된다. Express 내부 미들웨어에서 추가 검증/로그를 보강할 수 있다.

**관련 작업:**
- task-functions-scaffold
- task-middleware-core

**코드 예시:**

**App Check 강제 예시**

```typescript
import { https } from 'firebase-functions/v2';
import app from './app';

export const api = https.onRequest({ region: 'us-central1', enforceAppCheck: true }, app);
```

---

### reCAPTCHA 서버 검증 (guide-recaptcha-verify)

**카테고리:** 보안

클라이언트에서 받은 토큰을 서버에서 Google verify API로 검증한다. v2 체크박스는 success 플래그를, v3 점수는 score 임계값(예: ≥0.5)을 확인한다.

**관련 작업:**
- task-services-base
- task-endpoint-posts-create
- task-endpoint-reports-create

**코드 예시:**

**v2 체크박스 검증**

```typescript
export async function verifyRecaptcha(token: string, remoteip?: string) {
  const params = new URLSearchParams({ secret: process.env.RECAPTCHA_SECRET_KEY!, response: token });
  if (remoteip) params.append('remoteip', remoteip);
  const res = await fetch('https://www.google.com/recaptcha/api/siteverify', { method: 'POST', body: params });
  const data = await res.json();
  return !!data.success;
}
```

---

### Firestore 기반 레이트리밋(sliding window + daily cap) (guide-ratelimit-firestore)

**카테고리:** 데이터처리

rateLimits/{key_type} 문서에 windowEnd와 count를 저장하여 슬라이딩 윈도우를 구현하고, dayKey/da yCount로 일일 상한을 관리한다. TTL 필드(ttlAt)로 자동 정리를 활성화한다.

**관련 작업:**
- task-services-base
- task-endpoint-posts-create
- task-endpoint-reports-create

**코드 예시:**

**트랜잭션을 통한 한계 체크/증가**

```typescript
await db.runTransaction(async (tx) => {
  const ref = doc(db, 'rateLimits', `${key}_${type}`);
  const snap = await tx.get(ref);
  const now = Date.now();
  const dayKey = new Date().toISOString().slice(0,10).replace(/-/g,'');
  let data = snap.exists() ? snap.data() : { count: 0, windowEnd: 0, dayKey, dayCount: 0 };
  if (data.dayKey !== dayKey) { data.dayKey = dayKey; data.dayCount = 0; }
  if (now > data.windowEnd) { data.count = 0; data.windowEnd = now + windowSec*1000; }
  if (data.dayCount >= dailyCap) throw new Error('DAILY_CAP');
  if (data.count >= 1) throw new Error('WINDOW_CAP');
  data.count += 1; data.dayCount += 1; data.ttlAt = new Date(now + 7*24*3600*1000);
  tx.set(ref, data, { merge: true });
});
```

---

### Origin 화이트리스트 검사 (guide-origin-check)

**카테고리:** 보안

ORIGIN_WHITELIST 환경변수에 정의된 출처만 허용한다. 동일 출처 요청과 Hosting 프록시를 고려한다.

**관련 작업:**
- task-middleware-core

**코드 예시:**

**간단한 Origin 검사**

```typescript
const allowed = process.env.ORIGIN_WHITELIST!.split(',').map(s=>s.trim());
export function originGuard(req,res,next){
  const origin = req.headers.origin as string | undefined;
  if (!origin || !allowed.includes(origin)) return res.status(403).json({ success:false, error:{ code:'APP_CHECK_FAILED', message:'Forbidden origin' }, data:null });
  next();
}
```

---

### 서버 오류 코드→클라이언트 메시지 맵핑 (guide-error-mapping)

**카테고리:** 사용자경험

TRD 19번 표준에 따라 서버의 error.code를 한국어 메시지로 일관되게 매핑한다.

**관련 작업:**
- task-api-wrapper
- task-toast-error-boundary

**코드 예시:**

**문구 매핑 유틸**

```typescript
const messages = {
  VALIDATION_ERROR: '입력값을 확인하세요.',
  CAPTCHA_FAILED: 'CAPTCHA 검증에 실패했습니다.',
  APP_CHECK_FAILED: 'CAPTCHA 검증에 실패했습니다.',
  NOT_FOUND: '존재하지 않는 글입니다.',
  DUPLICATE_REPORT: '하루에 한 번만 신고할 수 있습니다.',
  RATE_LIMITED: '요청이 많습니다. 잠시 후 다시 시도하세요.',
  SERVER_ERROR: '일시적인 오류가 발생했습니다.'
};
export function mapError(e:{code:string,message?:string}){return messages[e.code] || messages.SERVER_ERROR;}
```

---

## 전역 고려사항

### 브라우저 호환성
- 모바일 Chrome/Android WebView 최신 2개 메이저, iOS Safari 최신 2개
- ES2019+ 타겟, Vite 모던 번들

### 성능 요구사항
- 모바일 4G p75 FCP < 1.5s
- 모바일 4G p75 TTI < 2.5s
- 서버 정상 케이스 p95 < 300ms
- 최초 JS < 120KB gzip

### 접근성 표준
- WCAG AA 대비
- 폰트 최소 14px
- 브라우저 기본 포커스 링 유지
- 키보드 탭 탐색 가능

### 오류 처리 정책
- 일관된 JSON 응답: {success,data,error}
- 서버 오류 코드 표준(VALIDATION_ERROR, CAPTCHA_FAILED 등)
- 클라이언트 토스트로 사용자 친화 메시지 표시
- ErrorBoundary로 네트워크 오류 재시도

### 메모리 관리
- 페이지 언마운트 시 구독 정리(onSnapshot 미사용, get 단발성)
- 대량 데이터 보관 금지(PII 저장 금지)
- GA4 큐 누수 방지

## 실행 순서

**요약:** 환경 구성 → 백엔드 스캐폴드/보안 → 데이터/인덱스 → 서버 서비스/엔드포인트 → 프론트 스캐폴드/데이터 → UI/플로우 → 분석/접근성/성능 → 테스트 → 배포/스모크 순으로 진행한다.

### 단계 1: 기반 설정

프로젝트/리포/CI/에뮬레이터 기반 구성 완료

**포함 작업:**
- task-setup-firebase-project
- task-repo-init
- task-ci-cd-setup
- task-emulator-setup

### 단계 2: 백엔드 골격/보안

API 프레임과 보안, 규칙/인덱스 준비

**포함 작업:**
- task-functions-scaffold
- task-middleware-core
- task-firestore-rules
- task-firestore-indexes

### 단계 3: 서버 도메인 기능

캡차/레이트리밋/콘텐츠 정책과 핵심 엔드포인트 구현 및 테스트

**포함 작업:**
- task-services-base
- task-endpoint-posts-create
- task-endpoint-reports-create
- task-logging-monitoring
- task-tests-unit-functions

### 단계 4: 프론트 스캐폴드/데이터

SPA 라우팅과 데이터 접근 래퍼 구현, Hosting 설정

**포함 작업:**
- task-web-scaffold
- task-appcheck-init
- task-firestore-reads-wrapper
- task-api-wrapper
- task-hosting-config
- task-firebase-json-rewrites

### 단계 5: UI/플로우 구현

목록/상세/작성/신고/고지 UI 및 사용자 피드백 흐름 완성

**포함 작업:**
- task-list-page
- task-detail-page
- task-compose-page
- task-report-modal
- task-guidelines
- task-toast-error-boundary

### 단계 6: 품질/계측

분석 이벤트, 접근성, 성능 최적화 및 테스트 정비

**포함 작업:**
- task-analytics-ga4
- task-accessibility
- task-performance-tuning
- task-web-tests

### 단계 7: 배포/스모크

프로덕션 배포 및 KPI 스모크 검증

**포함 작업:**
- task-prod-deploy-smoke

## 메타데이터

- **작성자:** AI 개발 워크플로우 설계 어시스턴트
- **버전:** 1.0.0
- **생성일:** 2025-08-24T00:00:00Z
- **수정일:** 2025-08-24T00:00:00Z
