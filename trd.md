# 기술 요구 사항 문서(TRD) — 모바일 익명 게시판(MVP)

본 문서는 AI 코딩 에이전트가 1주 내 프로토타입을 구현·배포할 수 있도록, 모듈·작업·인터페이스·검증 기준을 기계가독형으로 명확히 정의합니다.

## 📋 목차

- [1. 개요 및 범위](#1-개요-및-범위)
- [2. 아키텍처 개요](#2-아키텍처-개요)
- [3. 기술 스택 선택](#3-기술-스택-선택)
- [4. 데이터 모델 및 인덱스](#4-데이터-모델-및-인덱스)
- [5. 보안 설계](#5-보안-설계)
- [6. API 명세](#6-api-명세)
- [7. 클라이언트 앱 설계](#7-클라이언트-앱-설계)
- [8. 금지어/검증/레이트리밋 정책](#8-금지어검증레이트리밋-정책)
- [9. 접근성/국제화/법적 고지](#9-접근성국제화법적-고지)
- [10. 프로젝트 구조](#10-프로젝트-구조)
- [11. 환경 변수/설정](#11-환경-변수설정)
- [12. 빌드/배포 파이프라인](#12-빌드배포-파이프라인)
- [13. 기능 모듈 및 검증](#13-기능-모듈-및-검증)
- [14. 서버 내부 컴포넌트 설계](#14-서버-내부-컴포넌트-설계)
- [15. 테스트 계획](#15-테스트-계획)
- [16. 분석/모니터링](#16-분석모니터링)
- [17. 성능/품질 목표 및 검증](#17-성능품질-목표-및-검증)
- [18. 작업 분해 및 우선순위](#18-작업-분해-및-우선순위)
- [19. 에러 처리/사용자 메시지 표준](#19-에러-처리사용자-메시지-표준)
- [20. 유지보수/운영](#20-유지보수운영)
- [21. 정의/타입](#21-정의타입)
- [22. KPI 수용 기준 체크리스트](#22-kpi-수용-기준-체크리스트)
- [23. 위험 및 완화](#23-위험-및-완화)
- [24. 구현 가이드](#24-구현-가이드)

------------------------------------------------------------

## 1. 개요 및 범위

- 제품명: 모바일 익명 게시판 (MVP)
- 목표: 로그인/닉네임 없이 글 목록/상세/작성/신고 기능 구현, Firebase 일체형 아키텍처로 1주 내 배포
- 플랫폼: 모바일 웹 우선, 단일 페이지 앱(SPA)
- 범위(P0)
  - 글 목록(최신순, 페이지네이션 20개, 더보기)
  - 글 상세(제목/본문/작성시각/신고 버튼)
  - 글 작성(제목/본문/길이/금지어/레이트리밋/reCAPTCHA/App Check)
  - 신고(사유 선택, 중복 신고 제한, CAPTCHA/App Check)
  - 이용 안내/고지(링크/모달)
- 비기능 요구
  - 성능: FCP <1.5s, TTI <2.5s(모바일 4G p75)
  - 서버 p95 <300ms
  - 에러율 <1%
  - 접근성: WCAG AA 대비, 포커스 링
  - 보안: 완전 익명, Firestore 읽기만 공개, 쓰기 서버 전용

------------------------------------------------------------

## 2. 아키텍처 개요

- 프론트엔드: React + TypeScript + Vite(SPA), Firebase Web SDK
- 백엔드: Google Cloud Functions for Firebase (v2, Node.js 20, TypeScript, Express)
- 데이터: Firestore(문서형, 규칙: Read 공개, Write 서버만)
- 배포: Firebase Hosting(정적 SPA), Functions(HTTPS), GitHub Actions CI/CD
- 보안: Firebase App Check(reCAPTCHA v3) 강제 + reCAPTCHA v2(체크박스) 또는 v3(점수) 토큰 서버검증 병행
- 로깅/모니터링: Cloud Logging, Error Reporting, GA4
- 페이지네이션: Firestore 커서 기반(startAfter), 20개 단위
- 성능: 코드 스플리팅, 프리로드, Skeleton UI, 최소 의존성

------------------------------------------------------------

## 3. 기술 스택 선택

- 프론트엔드
  - React 18 + TypeScript
  - Vite 5
  - React Router 6
  - Firebase Web SDK (app, firestore, app-check)
  - dayjs(상대시간 플러그인 포함)
  - Tailwind CSS(경량 스타일링, 다크 모드 대응)
  - GA4 gtag.js
- 백엔드
  - Firebase Functions v2 (Node 20, TypeScript)
  - Express 4
  - cross-fetch 또는 undici(서버측 reCAPTCHA 검증용 HTTP)
  - crypto 내장 모듈(SHA-256)
- 테스트/개발
  - Firebase Emulator Suite(Functions+Firestore)
  - vitest(웹 단위), jest or vitest(Functions 단위)
  - Playwright(선택, 스모크 e2e)
- CI/CD
  - GitHub Actions + Firebase CLI

선정 근거: Firebase 일체형 단순 아키텍처, React/Vite의 성능 및 DX, 폭넓은 커뮤니티·문서.

------------------------------------------------------------

## 4. 데이터 모델 및 인덱스

- Firestore 컬렉션/문서 구조
  - posts
    - id: string (auto)
    - title: string (<=60)
    - body: string (<=2000)
    - createdAt: timestamp (server)
    - status: string in ["active","removed"] (default "active")
    - reportCount: number (default 0)
  - reports
    - id: string (auto)
    - postId: string (참조)
    - reason: string in ["spam","abuse","illegal","other"]
    - createdAt: timestamp (server)
    - reporterHash: string (SHA256(ip + ua + salt))
  - rateLimits
    - key: string (hash(IP) 또는 device)
    - type: string in ["post","report"]
    - count: number
    - windowEnd: timestamp
    - ttlAt: timestamp (TTL 7일)

- 필수 인덱스
  - posts: composite index on status(ASC) + createdAt(DESC)
  - 필요 시 단일 인덱스 createdAt(DESC)도 생성
- 쿼리 정책
  - 목록: where status == "active", orderBy createdAt desc, limit 20, startAfter 커서
  - 상세: doc by id(존재 확인 + status == "active" 확인)

검증 기준
- 인덱스 미스 시 콘솔 경고 없이 프로덕션 쿼리 p95 <300ms
- 목록/상세 쿼리 모두 status 필터 적용

------------------------------------------------------------

## 5. 보안 설계

- Firestore Rules
  - posts: allow read: true; write/update/delete: false
  - reports: allow read: false; write/update/delete: false
  - rateLimits: allow read/write: false
- Functions 보호
  - App Check enforcement(Functions v2 옵션): 유효한 App Check 토큰 없으면 403
  - reCAPTCHA 토큰 서버 검증 필수(작성/신고)
  - Origin 화이트리스트 검증(Firebase Hosting 도메인 및 커스텀 도메인)
  - X-Forwarded-For로 클라이언트 IP 파싱, 내부 프록시 신뢰 단계 1
  - 해시: reporterHash = SHA256(ip + ua + salt)
- 개인정보/보존
  - PII 저장 금지, IP raw 미저장
  - rateLimits TTL 7일 자동 삭제
- 남용 대응
  - 서버 레이트 리밋(기본값): 글쓰기 30초당 1건, 1일 10건/IP; 신고 15초당 1건, 1일 50건/IP
  - 신고 중복 제한: 동일 reporterHash-포스트-일자 1회
  - 자동 숨김: reportCount >= 5 시 status="removed" (정책 플래그로 조정 가능)

검증 기준
- 클라이언트에서 Firestore write 시도 실패(권한 오류)
- App Check 비활성/위조 토큰 요청 403
- reCAPTCHA 실패 시 403 + 에러 메시지
- 레이트리밋 초과 시 429 반환

------------------------------------------------------------

## 6. API 명세(서버 Functions, HTTPS)

공통
- Base URL: https://<hosting-domain>/api
- 요청 헤더: Content-Type: application/json; X-App-Client: web; Origin 검사
- App Check: 자동 검증(Functions v2), 실패 시 403
- 응답 포맷(JSON)
  - success: boolean
  - data: object | null
  - error: { code: string, message: string } | null

오류 코드
- VALIDATION_ERROR (400)
- CAPTCHA_FAILED (403)
- APP_CHECK_FAILED (403)
- RATE_LIMITED (429)
- DUPLICATE_REPORT (409)
- NOT_FOUND (404)
- SERVER_ERROR (500)

엔드포인트
1) POST /posts/create
- 요청: { title: string, body: string, captchaToken: string }
- 검증:
  - 길이/필수/금지어(서버 리스트) 체크
  - reCAPTCHA 서버 검증
  - 레이트리밋(post: 30s/1, 1d/10 by IP hash)
- 처리:
  - Firestore posts 문서 생성(createdAt serverTimestamp, status="active", reportCount=0)
- 응답: { id: string }
- 응답 시간 목표: p95 <300ms

2) POST /reports/create
- 요청: { postId: string, reason: "spam"|"abuse"|"illegal"|"other", captchaToken: string }
- 검증:
  - 존재하는 postId이며 status != "removed"
  - reCAPTCHA 서버 검증
  - reporterHash 계산(ip+ua), 동일 글 하루 1회 제한
  - 레이트리밋(report: 15s/1, 1d/50 by IP hash)
- 처리:
  - reports 문서 생성
  - posts.reportCount 원자 증가
  - reportCount 임계치 이상 시 posts.status="removed"
- 응답: { reported: true }

3) GET /health
- 응답: { ok: true, ts: number }

검증 기준
- 스키마 불일치/누락 시 400
- CAPTCHA 실패 403
- 중복 신고 시 409
- 존재하지 않는 postId 404

------------------------------------------------------------

## 7. 클라이언트 앱 설계

라우팅
- "/": 목록 페이지(ListPage)
- "/posts/:id": 상세 페이지(DetailPage)
- "/compose": 글쓰기 페이지(ComposePage)
- "/guidelines": 가이드라인 모달 라우트(또는 페이지)
- 404 라우트 및 유지보수 페이지(오류 플래그 시)

주요 화면/컴포넌트
- Layout/Header: 타이틀, 글쓰기 버튼
- PostList: 20개 로드, 더보기 버튼, Skeleton 3–5개
- PostListItem: 제목(1줄), 본문 2줄 프리뷰, 상대시각
- PostDetail: 제목/본문/작성시각, 신고 버튼, 404 처리
- ComposeForm: 제목/내용 입력, 길이/금지어 클라 검증, reCAPTCHA v2 위젯, 제출
- ReportModal: 사유 선택 라디오, reCAPTCHA v2 위젯, 제출
- Toast/Notification: 성공/실패 메시지
- ErrorBoundary/Retry: 네트워크 실패 재시도
- GuidelinesModal/Link: 상단/하단 고정 노출 링크

상태/데이터 패턴
- Firestore 읽기 전용:
  - 목록: query(posts where status=="active" orderBy createdAt desc limit 20), startAfter로 추가 로드
  - 상세: doc(posts/:id) 읽기 후 status 검사, removed 또는 미존재 시 404 UI
- 캐싱: 간단 캐시(메모리) 또는 React Query 사용(선택). MVP: Firestore SDK onSnapshot 단발성 get (최신성 우선), React Query는 선택(P1).
- 유효성(클라)
  - 제목/본문 필수, 길이 제한
  - 금지어 기본 리스트(클라 선제 검사), 실패 시 비활성화 및 안내
- 보안(클라)
  - Firebase App Check 초기화(reCAPTCHA v3 key)
  - reCAPTCHA v2(checkbox) 위젯 토큰으로 제출

UX/성능
- Skeleton, Lazy route, Code splitting
- 모바일 뷰포트 최적화(360–430px)
- 다크 모드(P1): prefers-color-scheme 대응

검증 기준
- 목록 최초 20개, 더보기 시 중복 없이 다음 20개
- 빈 상태 문구/오류 재시도 동작
- 상세 새로고침 시 동일 URL 접근 가능
- 작성 완료 시 상세 페이지로 이동
- 신고 성공/실패 토스트

------------------------------------------------------------

## 8. 금지어/검증/레이트리밋 정책

- 금지어(서버): 기본 리스트를 환경변수/파일로 로드(한글 및 영문, 소량). 클라에도 동일 리스트 번들(P1: 원격 관리).
- 레이트리밋 윈도우
  - post: sliding window(30초 1회), daily cap 10
  - report: sliding window(15초 1회), daily cap 50
- 해시 키
  - key = SHA256(ip + salt) for rateLimits
  - reporterHash = SHA256(ip + ua + salt)
- TTL
  - rateLimits.ttlAt = now + 7d (Firestore TTL 기능)

검증 기준
- 30초 내 2회 글쓰기 시 429
- 동일 글 하루 2회 신고 시 1회만 성공, 이후 409

------------------------------------------------------------

## 9. 접근성/국제화/법적 고지

- 접근성: WCAG AA 대비, 폰트 최소 14px, 포커스 링 기본 브라우저 스타일 유지
- 마이크로카피: PRD 문구 준수
- 국제화: 한국어만(MVP)
- 고지: 가이드라인 및 신고 고지 링크 항상 노출, 모달 내 내용 제공

검증 기준
- 키보드 탭 포커스 가능
- 다크 모드 시 대비(P1)

------------------------------------------------------------

## 10. 프로젝트 구조

- 루트
  - apps/
    - web/ (React + Vite + TS)
      - src/
        - app/ (Routing, App shell)
        - components/ (UI 컴포넌트)
        - pages/ (List, Detail, Compose, Guidelines)
        - services/
          - firebase.ts (SDK init, App Check)
          - firestore.ts (읽기 쿼리 래퍼)
          - api.ts (Functions 호출 래퍼: /api)
          - analytics.ts (GA4 이벤트)
          - validators.ts (길이/금지어/스키마)
        - styles/ (Tailwind, 전역 CSS)
        - utils/ (시간 포맷 등)
      - public/ (favicon, 로고)
      - index.html
      - vite.config
  - functions/ (Firebase Functions v2 + TS)
    - src/
      - index.ts (exported handlers)
      - app.ts (Express 인스턴스, 라우팅)
      - middleware/ (app-check, origin, schema validate)
      - services/
        - recaptcha.ts
        - ratelimit.ts
        - hash.ts
        - firestore.ts (CRUD 래퍼, 트랜잭션)
        - contentPolicy.ts (금지어/길이)
      - handlers/
        - postsCreate.ts
        - reportsCreate.ts
        - health.ts
      - types/ (요청/응답 스키마)
      - config.ts (상수/정책)
    - package.json, tsconfig.json
  - firestore.rules
  - firestore.indexes.json
  - firebase.json
  - .firebaserc
  - .github/workflows/deploy.yml
  - docs/ (운영/런북/정책)

검증 기준
- 로컬 에뮬레이터로 apps/web, functions 실행 가능
- firebase.json 라우트 프록시: /api/* → Functions

------------------------------------------------------------

## 11. 환경 변수/설정

- Functions 환경(config)
  - RECAPTCHA_SECRET_KEY (필수)
  - POLICY_SALT (필수, 32바이트 이상)
  - ORIGIN_WHITELIST (쉼표 구분): https://<project>.web.app, https://<custom-domain>
  - RATE_LIMIT_POST_WINDOW_SECS=30
  - RATE_LIMIT_POST_DAILY=10
  - RATE_LIMIT_REPORT_WINDOW_SECS=15
  - RATE_LIMIT_REPORT_DAILY=50
  - REPORT_HIDE_THRESHOLD=5
- Web .env
  - VITE_FIREBASE_API_KEY, AUTH_DOMAIN, PROJECT_ID, APP_ID
  - VITE_APPCHECK_SITE_KEY (reCAPTCHA v3)
  - VITE_RECAPTCHA_V2_SITE_KEY (체크박스)
  - VITE_GA_MEASUREMENT_ID

검증 기준
- 누락 시 빌드/부트 시 명확한 오류 로그

------------------------------------------------------------

## 12. 빌드/배포 파이프라인

- GitHub Actions
  - on: push to main → build web → test → deploy hosting preview → deploy functions to staging
  - manual approval → prod deploy (hosting + functions)
- Firebase Hosting
  - SPA rewrite: all to /index.html
  - API rewrite: "/api/**" → functions api
  - 캐싱: HTML no-cache, assets immutable
- Functions
  - 최소 동시 <100, 자동 스케일 기본
- 환경 분리
  - firebase project: dev/prod 별도
  - 환경값 Secret/Config 분리

검증 기준
- main 머지 시 Preview URL 자동 생성
- 롤백: Hosting 채널/버전 롤백 가능, Functions 이전 버전 재활성화 확인

------------------------------------------------------------

## 13. 기능 모듈 및 검증

### 모듈 A: 글 목록 뷰 (P0)
- 입력/의존: Firestore posts(status="active"), 인덱스, 네트워크
- 처리: 최초 20개 로드, 더보기로 다음 20개, 중복 제거(startAfter 커서)
- 출력/UI: 리스트, Skeleton, 빈 상태/오류 재시도
- 인터페이스:
  - getPostsPage(afterCursor?: DocumentSnapshot): { items: Post[], nextCursor? }
- 검증:
  - 0건 시 “아직 글이 없습니다...” 문구
  - 더보기 클릭 시 중복 없음
  - 네트워크 실패 시 재시도 버튼

### 모듈 B: 글 상세 뷰 (P0)
- 입력: posts/:id
- 처리: 문서 fetch, status 확인
- 출력/UI: 제목/본문/상대시각, 404 안내+목록 이동
- 인터페이스:
  - getPostById(id: string): Post | null
- 검증:
  - 존재X 또는 removed 시 404 UI
  - 새로고침 시 동일 URL 접근 정상

### 모듈 C: 글 작성 (P0)
- 입력: 제목/본문, reCAPTCHA v2 토큰(App Check는 자동)
- 처리: 클라 검증 → /api/posts/create POST
- 출력/UI: 성공 시 상세 이동, 실패 메시지
- 인터페이스:
  - createPost({title, body, captchaToken}): { id }
- 검증:
  - 필수/길이 오류 즉시 표시
  - 레이트리밋 초과 시 “잠시 후 다시 시도” 문구

### 모듈 D: 신고 (P0)
- 입력: postId, reason, reCAPTCHA v2 토큰
- 처리: /api/reports/create POST
- 출력/UI: 성공 토스트, 중복/제한 안내
- 인터페이스:
  - reportPost({postId, reason, captchaToken}): { reported: true }
- 검증:
  - 사유 필수, 중복 신고 제한 적용
  - 성공 시 “신고가 접수되었습니다”

### 모듈 E: 이용 안내/고지 (P0)
- 출력/UI: 상/하단 링크 항상 노출, 모달 내 텍스트
- 검증:
  - 모든 페이지에서 접근 가능

### 모듈 F: 성능/접근성 (P0)
- 처리: 코드 스플리팅, 이미지 미사용, Skeleton, 폰트/대비/포커스
- 검증:
  - p75 FCP <1.5s, TTI <2.5s(중급 기기/4G)
  - 포커스 이동/스크린리더 라벨 확인

### 모듈 G: 보안/남용 방지 (P0)
- 처리: App Check 강제, reCAPTCHA 서버 검증, 레이트리밋, Origin 검증
- 검증:
  - 위/위 조건 미충족 시 요청 거부(403/429)
  - Firestore direct write 차단 확인

------------------------------------------------------------

## 14. 서버 내부 컴포넌트 설계

### 미들웨어

- **appCheckGuard:** Functions v2 App Check enforcement 사용
- **originGuard:** ORIGIN_WHITELIST 매칭
- **jsonGuard:** Content-Type 확인 및 JSON 파싱
- **schemaValidator:** 요청 스키마 길이/형식 체크
- **captchaVerifier:** reCAPTCHA token → Google verify API
- **rateLimiter:** rateLimits 컬렉션 기반 카운트/윈도우

### 서비스

- **recaptcha.verify(token, remoteip):** boolean
- **ratelimit.checkAndIncrement(key, type, windowSec, dailyCap):** { allowed:boolean, retryAfterSec?:number }
- **hash.sha256(input):** hex
- **firestore.createPost(data), incrementReport(postId), createReport(data)**
- **contentPolicy.validate(title, body):** { ok:boolean, reason?:string }

### 트랜잭션

- **신고 처리:** reports 생성 + posts.reportCount 증가 + 임계치 시 status 변경(원자)

### 검증 기준

- **단위 테스트:** 각 서비스 성공/실패 케이스
- **임계치 교차조건 테스트:** reportCount==threshold-1 → +1 시 removed

------------------------------------------------------------

## 15. 테스트 계획

### 단위 테스트 (Functions)
- **금지어/길이/스키마** 검증
- **reCAPTCHA 실패/성공** 분기 처리
- **레이트리밋 윈도우/데일리 캡** 로직
- **신고 중복 제한** 정책
### 통합 테스트 (Emulator)
- **posts/create** 후 Firestore 문서값 검증
- **reports/create** 후 reportCount 증가/자동 숨김
- **Firestore 규칙** (클라 write 실패)
### 웹 단위/통합 테스트
- **폼 검증** 메시지 표시
- **목록 더보기** 중복 없음 확인
- **상세 404** 처리 동작
- **토스트/재시도** UI 동작
### E2E 테스트 (스모크)
- **핵심 플로우:** 접속→목록→상세→글쓰기→상세 이동→신고
### 성능 테스트
- **Lighthouse 모바일** p75 FCP/TTI 측정
### 접근성 테스트
- **Axe/WAVE** 기본 위반 0, 대비 AA 준수

### 수락 기준 (샘플 시나리오)

- **목록:** 첫 페이지 20개, 더보기 시 다음 20개, 중복 없음
- **작성:** 실패(길이/금지어/레이트리밋)는 적절한 메시지
- **신고:** 중복 시 409 처리 및 안내
- **보안:** 클라 Firestore write 금지 확인

------------------------------------------------------------

## 16. 분석/모니터링

### GA4 이벤트
- **page_view:** list/detail 페이지 전환
- **post_attempt/success/fail:** 글 작성 시도/성공/실패
- **report_attempt/success/fail:** 신고 시도/성공/실패
- **pagination_click:** 더보기 클릭
### 서버 로깅
- **요청 추적:** 요청 ID, IP 해시, 결과 코드, 처리 시간
- **보안 모니터링:** 레이트리밋 트리거, CAPTCHA 실패 카운트
### 알림 (P2)
- **장애 대응:** 임계 장애 시 대시보드 확인, 향후 이메일/Alerting 연결

### 검증 기준
- **이벤트 전송:** 성공률 >95%
- **로그 분석:** 에러 로그 샘플링 검토 가능

------------------------------------------------------------

## 17. 성능/품질 목표 및 검증

### 웹 번들
- **최초 JS:** < 120KB gzip 목표
- **코드 스플리팅:** 라우트 기준
### 캐싱
- **정적 자산:** immutable, HTML no-cache
### 서버
- **응답 시간:** p95 <300ms (정상 케이스)
### UX
- **로딩:** Skeleton UI 노출 ≤300ms
### 가용성
- **Uptime:** ≥99.5%

### 검증 도구
- **웹 성능:** Lighthouse CI, WebPageTest 4G
- **서버 모니터링:** Cloud Monitoring 지표

------------------------------------------------------------

## 18. 작업 분해 및 우선순위

D1 설계/환경(P0)
- Firebase 프로젝트/앱 생성, App Check(reCAPTCHA v3) 설정
- 리포지토리 초기화, CI 골격, Hosting/Functions 연결
- 검증: 로컬 에뮬레이터 기동, 빈 페이지 배포 확인

D2 백엔드 스캐폴드(P0)
- Functions v2 + Express 앱, /health
- 미들웨어: origin/json/appCheck 골격
- 검증: /api/health 200 응답

D2 데이터/보안(P0)
- Firestore rules(read-only), 인덱스(status+createdAt)
- 검증: 규칙 시뮬레이션/에뮬레이터 통과

D3 posts/create(P0)
- 스키마/금지어/길이/레이트리밋/recaptcha/트랜잭션 없는 단일 생성
- 검증: 올바른 요청 시 id 반환, 오류 코드 처리

D4 reports/create(P0)
- 신고 중복 제한(reporterHash+post+day), reportCount 증가/임계치 숨김
- 검증: 중복 409, 임계치 도달 시 status=removed

D3–D4 웹 목록/상세(P0)
- 목록 20개 + 더보기, 상세 존재/removed→404
- 검증: 커서·중복·빈 상태/오류 재시도

D4–D5 웹 작성/신고(P0)
- 폼 검증, reCAPTCHA v2 위젯, 성공 플로우
- 검증: 토스트/에러 핸들링/상세 이동

D5 가이드라인/GA4(P0)
- 상/하단 링크, 모달, 이벤트 바인딩
- 검증: 이벤트 송신/링크 노출

D6 성능/접근성 튜닝(P0)
- 번들 최적화, Skeleton, 포커스 링
- 검증: FCP/TTI 목표, Axe 오류 0

D7 배포/스모크(P0)
- 프로덕션 배포, 스모크 플로우, 모니터링 점검
- 검증: KPI 체크리스트 통과

선택 작업(P1)
- 다크 모드, 금지어 원격 관리, React Query 도입
- 검증: 대비 기준 충족

후순위(P2)
- 작성자 자가 삭제(10분 토큰)
- 검증: 동일 브라우저 내 10분 이내 삭제 가능

------------------------------------------------------------

## 19. 에러 처리/사용자 메시지 표준

- 400 VALIDATION_ERROR: “입력값을 확인하세요.”
- 403 CAPTCHA_FAILED/APP_CHECK_FAILED: “CAPTCHA 검증에 실패했습니다.”
- 404 NOT_FOUND: “존재하지 않는 글입니다.”
- 409 DUPLICATE_REPORT: “하루에 한 번만 신고할 수 있습니다.”
- 429 RATE_LIMITED: “요청이 많습니다. 잠시 후 다시 시도하세요.”
- 500 SERVER_ERROR: “일시적인 오류가 발생했습니다.”

검증 기준
- 서버 코드→클라 메시지 맵핑 일관, 한국어 표시

------------------------------------------------------------

## 20. 유지보수/운영

- 운영 플래그
  - REPORT_HIDE_THRESHOLD 동적 조정 가능
  - ORIGIN_WHITELIST 추가
- 롤백 전략
  - Hosting 버전 롤백
  - Functions 이전 릴리스 재활성화
- 데이터 보관
  - posts/reports 영구
  - rateLimits TTL 7일

검증 기준
- 플래그 변경 후 무중단 반영(재배포 시)

------------------------------------------------------------

## 21. 정의/타입(참조 스키마)

- Post
  - id, title, body, createdAt, status, reportCount
- Report
  - id, postId, reason, createdAt, reporterHash
- API 요청 스키마
  - posts/create: { title: string(1..60), body: string(1..2000), captchaToken: string(>0) }
  - reports/create: { postId: string, reason in enum, captchaToken: string }

검증 기준
- 길이/enum 위반 시 400

------------------------------------------------------------

## 22. KPI 수용 기준 체크리스트

- 목록: 20개 로드, 더보기 20개, 중복 없음, 빈/에러 처리 OK
- 상세: 제목/본문/시간 표시, 존재X/removed 시 404 안내
- 작성: 필수/길이/금지어/레이트리밋/캡차 검증, 성공 후 상세 이동
- 신고: 사유 필수, 중복 제한, 성공 토스트
- 보안: 클라 Firestore write 불가, Functions만 쓰기 성공
- 성능: FCP<1.5s, TTI<2.5s(p75), 서버 p95 <300ms
- 접근성: AA 대비, 폰트 ≥14px, 포커스 링 제공

------------------------------------------------------------

## 23. 위험 및 완화

- 스팸/봇: App Check + reCAPTCHA + 레이트리밋 + Origin 검사 + IP 해시 블록리스트(P1)
- 악성 콘텐츠: 신고 임계치 자동 숨김 + 가이드라인/신고 고지
- 트래픽 급증: Hosting 캐싱, Functions 자동 스케일, 텍스트 위주
- 무인 운영: 자동 규칙 우선, 알림은 P2

검증 기준
- 스팸 시나리오 시 차단율 ≥90% 목표(로그 기반)

------------------------------------------------------------

## 24. 구현 가이드

### 핵심 구현 패턴

#### Firestore 커서 기반 페이지네이션
```typescript
async function getPostsPage(after?: DocumentSnapshot) {
  const base = [
    where('status', '==', 'active'),
    orderBy('createdAt', 'desc'),
    limit(20)
  ];
  const q = after 
    ? query(collection(db, 'posts'), ...base, startAfter(after))
    : query(collection(db, 'posts'), ...base);
  
  const snap = await getDocs(q);
  const items = snap.docs.map(d => ({ id: d.id, ...d.data() }));
  const nextCursor = snap.docs[snap.docs.length - 1];
  
  return { items, nextCursor };
}
```

#### Functions v2 App Check 강제
```typescript
import { https } from 'firebase-functions/v2';

export const api = https.onRequest(
  { 
    region: 'us-central1', 
    enforceAppCheck: true 
  }, 
  app
);
```

#### reCAPTCHA 서버 검증
```typescript
export async function verifyRecaptcha(token: string, remoteip?: string) {
  const params = new URLSearchParams({
    secret: process.env.RECAPTCHA_SECRET_KEY!,
    response: token
  });
  if (remoteip) params.append('remoteip', remoteip);
  
  const res = await fetch('https://www.google.com/recaptcha/api/siteverify', {
    method: 'POST',
    body: params
  });
  const data = await res.json();
  return !!data.success;
}
```

#### Firestore 기반 레이트리밋
```typescript
async function checkRateLimit(key: string, type: string) {
  return await db.runTransaction(async (tx) => {
    const ref = doc(db, 'rateLimits', `${key}_${type}`);
    const snap = await tx.get(ref);
    const now = Date.now();
    
    let data = snap.exists() ? snap.data() : { 
      count: 0, 
      windowEnd: 0, 
      dayCount: 0 
    };
    
    if (now > data.windowEnd) {
      data.count = 0;
      data.windowEnd = now + 30000; // 30초
    }
    
    if (data.count >= 1) throw new Error('RATE_LIMITED');
    
    data.count += 1;
    data.ttlAt = new Date(now + 7 * 24 * 3600 * 1000);
    
    tx.set(ref, data, { merge: true });
    return true;
  });
}
```

### 보안 구현 체크리스트

- [ ] App Check 토큰 검증 (Functions v2 enforceAppCheck)
- [ ] Origin 화이트리스트 검사
- [ ] reCAPTCHA 서버 검증 (모든 쓰기 작업)
- [ ] IP 기반 레이트리밋 (해시화)
- [ ] Firestore Rules: 읽기만 공개, 쓰기 서버 전용
- [ ] Content-Type 및 JSON 스키마 검증
- [ ] 에러 메시지에서 민감 정보 노출 금지

### 성능 최적화 체크리스트

- [ ] 코드 스플리팅 (라우트 기준)
- [ ] Firestore 인덱스 최적화
- [ ] Skeleton UI (300ms 이내 표시)
- [ ] 이미지 최적화 (텍스트 중심)
- [ ] 번들 크기 < 120KB gzip
- [ ] 캐싱 전략 (HTML no-cache, assets immutable)

------------------------------------------------------------

본 TRD는 모듈, 인터페이스, 정책, 검증 기준을 결정적이고 원자적으로 정의하여, AI 코딩 에이전트가 1주 내 MVP를 구현·배포할 수 있도록 합니다.