# 🚀 배포 체크리스트

GitHub Actions를 통한 Firebase 자동 배포 전 확인해야 할 사항들입니다.

## 🔐 GitHub Secrets 설정 ✅

- [ ] `FIREBASE_PROJECT_ID` 설정
- [ ] `FIREBASE_SERVICE_ACCOUNT` 설정 (base64 인코딩된 JSON)
- [ ] `VITE_FIREBASE_API_KEY` 설정
- [ ] `VITE_FIREBASE_AUTH_DOMAIN` 설정
- [ ] `VITE_FIREBASE_PROJECT_ID` 설정
- [ ] `VITE_FIREBASE_APP_ID` 설정
- [ ] `VITE_APPCHECK_SITE_KEY` 설정

## 🔥 Firebase 프로젝트 설정 ✅

- [ ] Firebase 프로젝트 생성 완료
- [ ] Hosting 서비스 활성화
- [ ] Functions 서비스 활성화
- [ ] Firestore 서비스 활성화
- [ ] 서비스 계정 키 생성 및 다운로드
- [ ] 프로젝트 ID 확인

## 📁 프로젝트 구조 ✅

- [ ] `.github/workflows/deploy.yml` 파일 존재
- [ ] `apps/web/env.example` 파일 존재
- [ ] `firebase.json` 설정 완료
- [ ] `.gitignore` 파일 설정 완료
- [ ] 모든 필요한 의존성 설치 완료

## 🧪 테스트 ✅

- [ ] 로컬 빌드 테스트 (`npm run build`)
- [ ] 로컬 개발 서버 테스트 (`npm run dev`)
- [ ] Firebase 에뮬레이터 테스트 (`npm run dev:emulators`)
- [ ] API 엔드포인트 테스트
- [ ] 환경 변수 주입 테스트

## 🔄 배포 설정 ✅

- [ ] GitHub 저장소에 코드 푸시 완료
- [ ] main/master 브랜치 설정 확인
- [ ] GitHub Actions 워크플로우 파일 검증
- [ ] Firebase CLI 로그인 완료 (로컬 테스트용)

## 📱 모바일 최적화 ✅

- [ ] 반응형 디자인 테스트
- [ ] 모바일 브라우저 호환성 확인
- [ ] 터치 인터페이스 테스트
- [ ] 성능 최적화 적용

## 🔒 보안 설정 ✅

- [ ] Firebase App Check 설정
- [ ] reCAPTCHA v3 통합
- [ ] Firestore 보안 규칙 설정
- [ ] Origin Whitelisting 설정
- [ ] Rate Limiting 설정

## 🚀 배포 실행 ✅

- [ ] main/master 브랜치에 푸시
- [ ] GitHub Actions 워크플로우 실행 확인
- [ ] 빌드 단계 성공 확인
- [ ] 배포 단계 성공 확인
- [ ] Firebase Console에서 배포 상태 확인

## 🌐 배포 후 확인 ✅

- [ ] 웹 앱 접근 가능 확인
- [ ] API 엔드포인트 동작 확인
- [ ] 모바일 디바이스에서 테스트
- [ ] 성능 메트릭 확인
- [ ] 에러 로그 확인

## 📊 모니터링 설정 ✅

- [ ] Firebase Analytics 설정
- [ ] Performance Monitoring 설정
- [ ] Error Reporting 설정
- [ ] 사용자 피드백 수집 방법 설정

## 🔧 문제 해결 도구 ✅

- [ ] 로그 확인 방법 숙지
- [ ] 롤백 방법 숙지
- [ ] 긴급 연락처 준비
- [ ] 백업 전략 수립

---

## 🚨 긴급 상황 대응

### 배포 실패 시
1. GitHub Actions 로그 확인
2. Firebase Console 에러 확인
3. 환경 변수 설정 재확인
4. 필요시 이전 버전으로 롤백

### 서비스 중단 시
1. Firebase Console에서 상태 확인
2. Functions 로그 확인
3. Firestore 상태 확인
4. 사용자에게 공지

### 성능 문제 시
1. Firebase Performance Monitoring 확인
2. Core Web Vitals 측정
3. 네트워크 요청 분석
4. 최적화 적용

---

**마지막 업데이트**: 배포 전 최종 확인 완료 시 체크
