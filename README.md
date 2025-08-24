# Anonymous Board MVP

모바일 익명 게시판 MVP - Firebase 일체형 아키텍처

## 🚀 배포 방법

### 1. GitHub Secrets 설정

GitHub 저장소의 Settings > Secrets and variables > Actions에서 다음 시크릿을 설정하세요:

#### Firebase 설정
- `FIREBASE_PROJECT_ID`: Firebase 프로젝트 ID
- `FIREBASE_SERVICE_ACCOUNT`: Firebase 서비스 계정 JSON (base64 인코딩)

#### 환경 변수
- `VITE_FIREBASE_API_KEY`: Firebase API 키
- `VITE_FIREBASE_AUTH_DOMAIN`: Firebase 인증 도메인
- `VITE_FIREBASE_PROJECT_ID`: Firebase 프로젝트 ID
- `VITE_FIREBASE_APP_ID`: Firebase 앱 ID
- `VITE_APPCHECK_SITE_KEY`: reCAPTCHA v3 사이트 키

### 2. Firebase 서비스 계정 키 생성

1. Firebase Console > 프로젝트 설정 > 서비스 계정
2. "새 비공개 키 생성" 클릭
3. 다운로드된 JSON 파일을 base64로 인코딩:

```bash
# Windows PowerShell
[Convert]::ToBase64String([IO.File]::ReadAllBytes("path/to/serviceAccountKey.json"))

# macOS/Linux
base64 -i path/to/serviceAccountKey.json
```

4. 인코딩된 값을 `FIREBASE_SERVICE_ACCOUNT` 시크릿에 설정

### 3. 자동 배포

- `main` 또는 `master` 브랜치에 푸시하면 자동으로 Firebase에 배포됩니다
- GitHub Actions에서 배포 진행 상황을 확인할 수 있습니다

### 4. 수동 배포

```bash
# 의존성 설치
npm install

# 웹 앱 빌드
npm run build:web

# Functions 빌드
npm run build:functions

# Firebase 배포
npm run deploy
```

## 🛠️ 개발 환경 설정

### 1. 환경 변수 설정

`apps/web/env.example` 파일을 복사하여 `.env.local` 파일을 생성하고 실제 값으로 설정:

```bash
cp apps/web/env.example apps/web/.env.local
```

### 2. 개발 서버 실행

```bash
# 전체 개발 환경 (웹 + Firebase 에뮬레이터)
npm run dev

# 웹 앱만 실행
npm run dev:web

# Firebase 에뮬레이터만 실행
npm run dev:emulators
```

## 📁 프로젝트 구조

```
├── apps/
│   └── web/                 # React 웹 앱
│       ├── src/
│       ├── dist/            # 빌드 결과물
│       └── package.json
├── functions/               # Firebase Functions
├── firebase.json           # Firebase 설정
└── package.json            # 루트 패키지 설정
```

## 🔧 주요 스크립트

- `npm run dev`: 개발 환경 실행
- `npm run build`: 전체 프로젝트 빌드
- `npm run deploy`: Firebase에 배포
- `npm run test`: 테스트 실행

## 🌐 배포 URL

배포가 완료되면 다음 URL에서 접근할 수 있습니다:
- 웹 앱: `https://your-project-id.web.app`
- API: `https://your-project-id.web.app/api`

## 📝 주의사항

1. **환경 변수**: 프로덕션 배포 전 모든 환경 변수가 올바르게 설정되었는지 확인
2. **Firebase 설정**: Firebase 프로젝트가 올바르게 구성되었는지 확인
3. **도메인 설정**: Firebase Hosting에서 커스텀 도메인을 사용하는 경우 추가 설정 필요
