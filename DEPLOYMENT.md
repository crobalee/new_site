# 🚀 GitHub 배포 가이드

이 문서는 Anonymous Board MVP 프로젝트를 GitHub Actions를 통해 Firebase에 자동 배포하는 방법을 설명합니다.

## 📋 사전 준비사항

### 1. Firebase 프로젝트 설정
- [Firebase Console](https://console.firebase.google.com/)에서 새 프로젝트 생성
- Hosting, Functions, Firestore 서비스 활성화
- 프로젝트 ID 확인

### 2. GitHub 저장소 생성
- GitHub에서 새 저장소 생성
- 로컬 프로젝트를 저장소에 연결

## 🔐 GitHub Secrets 설정

### 1. Firebase 서비스 계정 키 생성

1. **Firebase Console > 프로젝트 설정 > 서비스 계정**
2. **"새 비공개 키 생성"** 클릭
3. JSON 파일 다운로드
4. JSON 파일을 base64로 인코딩:

#### Windows PowerShell
```powershell
[Convert]::ToBase64String([IO.File]::ReadAllBytes("C:\path\to\serviceAccountKey.json"))
```

#### macOS/Linux
```bash
base64 -i path/to/serviceAccountKey.json
```

### 2. GitHub Secrets 설정

저장소의 **Settings > Secrets and variables > Actions**에서 다음 시크릿들을 설정:

| Secret 이름 | 설명 | 예시 값 |
|------------|------|---------|
| `FIREBASE_PROJECT_ID` | Firebase 프로젝트 ID | `my-project-123` |
| `FIREBASE_SERVICE_ACCOUNT` | base64 인코딩된 서비스 계정 JSON | `eyJ0eXBlIjoic2VydmljZV9hY2NvdW50Iiw...` |
| `VITE_FIREBASE_API_KEY` | Firebase API 키 | `AIzaSyC...` |
| `VITE_FIREBASE_AUTH_DOMAIN` | Firebase 인증 도메인 | `my-project-123.firebaseapp.com` |
| `VITE_FIREBASE_PROJECT_ID` | Firebase 프로젝트 ID | `my-project-123` |
| `VITE_FIREBASE_APP_ID` | Firebase 앱 ID | `1:123456789:web:abc123` |
| `VITE_APPCHECK_SITE_KEY` | reCAPTCHA v3 사이트 키 | `6Lc...` |

## 🔄 자동 배포 설정

### 1. 워크플로우 파일 확인

`.github/workflows/deploy.yml` 파일이 올바르게 설정되어 있는지 확인:

```yaml
name: Deploy to Firebase

on:
  push:
    branches: [ main, master ]
  pull_request:
    branches: [ main, master ]

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    
    steps:
    - name: Checkout code
      uses: actions/checkout@v4

    - name: Setup Node.js
      uses: actions/setup-node@v4
      with:
        node-version: '20'
        cache: 'npm'

    - name: Install dependencies
      run: npm ci

    - name: Build web app
      run: npm run build:web
      env:
        VITE_FIREBASE_API_KEY: ${{ secrets.VITE_FIREBASE_API_KEY }}
        VITE_FIREBASE_AUTH_DOMAIN: ${{ secrets.VITE_FIREBASE_AUTH_DOMAIN }}
        VITE_FIREBASE_PROJECT_ID: ${{ secrets.VITE_FIREBASE_PROJECT_ID }}
        VITE_FIREBASE_APP_ID: ${{ secrets.VITE_FIREBASE_APP_ID }}
        VITE_APPCHECK_SITE_KEY: ${{ secrets.VITE_APPCHECK_SITE_KEY }}

    - name: Build functions
      run: npm run build:functions

    - name: Deploy to Firebase
      uses: FirebaseExtended/action-hosting-deploy@v0
      with:
        repoToken: '${{ secrets.GITHUB_TOKEN }}'
        firebaseServiceAccount: '${{ secrets.FIREBASE_SERVICE_ACCOUNT }}'
        channelId: live
        projectId: ${{ secrets.FIREBASE_PROJECT_ID }}
        entryPoint: '.'
```

### 2. 브랜치 설정

- `main` 또는 `master` 브랜치에 푸시하면 자동 배포
- 다른 브랜치는 Pull Request 생성 시 빌드만 실행

## 🚀 배포 프로세스

### 1. 자동 배포
```bash
# main 브랜치에 푸시
git add .
git commit -m "Add new feature"
git push origin main
```

### 2. 수동 배포
```bash
# 로컬에서 빌드 및 배포
npm run deploy:github
```

### 3. 개별 서비스 배포
```bash
# Hosting만 배포
npm run deploy:hosting

# Functions만 배포
npm run deploy:functions
```

## 📊 배포 모니터링

### 1. GitHub Actions
- **Actions** 탭에서 워크플로우 실행 상태 확인
- 빌드 로그 및 에러 메시지 확인

### 2. Firebase Console
- **Hosting** > **배포** 탭에서 배포 상태 확인
- **Functions** > **로그** 탭에서 함수 실행 로그 확인

## 🔧 문제 해결

### 1. 빌드 실패
- Node.js 버전 확인 (20.x 이상 필요)
- 의존성 설치 문제 확인
- 환경 변수 설정 확인

### 2. 배포 실패
- Firebase 서비스 계정 키 확인
- Firebase 프로젝트 ID 확인
- Firebase CLI 권한 확인

### 3. 환경 변수 문제
- GitHub Secrets 설정 확인
- 환경 변수 이름 확인 (VITE_ 접두사)
- 빌드 시 환경 변수 주입 확인

## 📝 환경별 설정

### 개발 환경
```bash
# .env.local 파일 생성
cp apps/web/env.example apps/web/.env.local

# 환경 변수 설정
VITE_FIREBASE_API_KEY=your_api_key
VITE_FIREBASE_AUTH_DOMAIN=your_project.firebaseapp.com
VITE_FIREBASE_PROJECT_ID=your_project_id
VITE_FIREBASE_APP_ID=your_app_id
VITE_APPCHECK_SITE_KEY=your_recaptcha_key
```

### 프로덕션 환경
- GitHub Secrets를 통한 자동 설정
- 빌드 시 환경 변수 주입
- Firebase Hosting을 통한 정적 파일 서빙

## 🌐 도메인 설정

### 1. Firebase Hosting 도메인
- 기본 도메인: `https://your-project-id.web.app`
- 커스텀 도메인 설정 가능

### 2. 커스텀 도메인 설정
1. Firebase Console > Hosting > 사용자 정의 도메인
2. 도메인 추가 및 DNS 설정
3. SSL 인증서 자동 발급

## 📱 모바일 최적화

- PWA 설정 (manifest.json, service worker)
- 모바일 터치 최적화
- 반응형 디자인 적용
- 성능 최적화 (코드 스플리팅, 지연 로딩)

## 🔒 보안 설정

### 1. Firebase App Check
- reCAPTCHA v3 통합
- 클라이언트 앱 검증

### 2. Firestore 보안 규칙
- 읽기 전용 설정
- 서버 전용 쓰기 권한

### 3. Functions 보안
- Origin Whitelisting
- Rate Limiting
- reCAPTCHA 검증

## 📈 성능 모니터링

### 1. Core Web Vitals
- LCP (Largest Contentful Paint)
- FID (First Input Delay)
- CLS (Cumulative Layout Shift)

### 2. Firebase Performance
- 네트워크 요청 모니터링
- 사용자 경험 메트릭
- 에러 추적

## 🎯 다음 단계

1. **모니터링 설정**: Firebase Analytics, Performance Monitoring
2. **CI/CD 개선**: 테스트 자동화, 품질 게이트
3. **스테이징 환경**: 개발/스테이징/프로덕션 분리
4. **백업 전략**: 데이터 백업 및 복구 계획
5. **확장성 계획**: 트래픽 증가 대응 방안
