# MODAK 법인 설립 후 서비스 가입 가이드(MODAK Group Pty Ltd)

> 법인 설립 후 MODAK 앱 운영에 필요한 서비스 가입 순서와 비용 정리

---

## STEP 1 — 법인 설립 (가장 먼저)

| 항목 | 기관 | 비용 | 처리시간 |
|------|------|------|---------|
| ABN 신청 | business.gov.au | 무료 | 즉시 |
| ACN + Pty Ltd 등록 | ASIC | $597 AUD | 1~3일 |
| TFN 법인용 | ATO | 무료 | 수일 |
| GST 등록 | ATO | 무료 | 즉시 |
| 법인 은행 계좌 | CBA / ANZ / NAB | 무료 | ABN 후 |

---

## STEP 2 — 법인 이메일 + 도메인 (법인 설립 직후 바로)

> ⚠️ 이후 모든 서비스는 이 법인 이메일로 가입

| 서비스 | 용도 | 비용 |
|--------|------|------|
| 도메인 등록 | modak.com.au 또는 modak.app | $15~30 AUD/년 |
| Google Workspace | 법인 이메일 (admin@modak.com.au) | $14 USD/월 |

---

## STEP 3 — 개발 인프라 (개발 시작 시)

| 서비스 | 용도 | 비용 | 우선순위 |
|--------|------|------|---------|
| Firebase | DB + 백엔드 + 인증 + 푸시 전체 | 무료 시작 | 최우선 |
| Google Cloud Console | 구글 지도 API | 월 $200 무료 크레딧 | 최우선 |
| GitHub | 코드 저장소 / 개발자 협업 | 무료 | 최우선 |
| Apple Developer | iOS 앱 배포 | $99 USD/년 | 개발 초기 |
| Google Play | Android 앱 배포 | $25 USD 일회성 | 개발 초기 |

### Firebase가 커버하는 것들

```
Firebase
├── Firestore        → DB 서버 (게시물, 유저, 스폰서, 딜 데이터)
├── Authentication   → 로그인 서버 (구글, 전화번호)
├── Storage          → 파일 서버 (사진 업로드)
├── Cloud Messaging  → 푸시 알림 (FCM)
├── Cloud Functions  → 백엔드 서버 (결제, 알림 발송)
└── Hosting          → 관리자 페이지 배포
```

> Firebase 하나로 백엔드 + DB 서버 + 스토리지 + 인증 + 푸시 알림 전부 대체 가능
> 별도 AWS, 백엔드 서버 불필요 (초기)

---

## STEP 4 — 로그인 연동 (개발 중)

| 서비스 | 용도 | 비용 | 비고 |
|--------|------|------|------|
| 구글 로그인 | Firebase에 내장 | 무료 | 별도 가입 불필요 |
| 전화번호 SMS 인증 | Firebase에 내장 | 무료~소액 | 별도 가입 불필요 |
| 카카오 개발자 | 카카오 로그인 | 무료 | developers.kakao.com |

### 카카오 로그인 가입 순서

```
1. developers.kakao.com 가입 (개인 카카오 계정으로)
2. 앱 생성 → 테스트 앱으로 개발 진행
3. 법인 설립 후 → 호주 법인 서류로 비즈 앱 전환 시도
4. 안 되면 → 가족/지인 한국 간이사업자로 등록
```

> 카카오는 해외 사업자 라이센스로도 비즈 앱 등록 가능
> 개발 기간에는 테스트 앱으로 충분

---

## STEP 5 — 결제 처리 (런칭 직전)

| 서비스 | 용도 | 비용 | 필요 서류 |
|--------|------|------|---------|
| Stripe (stripe.com/au) | 광고료 결제 처리 | 1.7% + 30c | ABN + 법인 계좌 |

---

## STEP 6 — 운영 관리 (런칭 후)

| 서비스 | 용도 | 비용 |
|--------|------|------|
| Cloudflare | CDN + DNS 관리 | 무료 |
| Sentry | 앱 에러 모니터링 | 무료 시작 |
| Xero | 법인 회계 소프트웨어 | $32~85 AUD/월 |

---

## 전체 타임라인

```
법인 설립 완료
    │
    ▼
도메인 + Google Workspace
(법인 이메일 확보 — 모든 가입의 기준)
    │
    ▼
Firebase + Google Cloud Console + GitHub
(개발 시작)
    │
    ▼
Apple Developer + Google Play
(앱 배포 준비)
    │
    ▼
카카오 개발자 가입
(로그인 구현)
    │
    ▼
Stripe 가입
(런칭 직전)
    │
    ▼
Cloudflare + Sentry + Xero
(런칭 후 운영)
```

---

## 초기 1년 비용 합계

| 항목 | 금액 |
|------|------|
| Pty Ltd 등록 | $597 AUD |
| 도메인 | ~$25 AUD |
| Google Workspace 12개월 | ~$200 AUD |
| Apple Developer | ~$150 AUD |
| Google Play | ~$38 AUD |
| **합계** | **~$1,010 AUD** |

> Firebase, GitHub, Cloudflare, Sentry, 카카오, 구글 로그인, SMS 인증은 초기 무료

---

## Flutter + Firebase 전체 구조

```
Flutter 앱 (유저)
    ↕ Firebase SDK
Firebase
├── 로그인/회원가입    → Authentication
├── 게시물 저장/조회   → Firestore (DB)
├── 사진 업로드       → Storage
├── 푸시 알림        → FCM
├── 광고 결제 처리    → Cloud Functions (백엔드)
└── 관리자 페이지     → Hosting

관리자 페이지 (현재 HTML)
    ↕ Firebase SDK
Firebase (동일 프로젝트 공유)
```

---

## 개발 순서 추천

```
1. Firebase 프로젝트 생성 (법인 Google 계정으로)
2. Firestore 데이터 구조 설계
3. Authentication 설정 (구글 + 전화번호 + 카카오)
4. 관리자 페이지 Firebase 연동
5. Flutter 앱 개발 시작
```

---

*작성일: 2026년 4월*
*대상: MODAK Australia Pty Ltd*
