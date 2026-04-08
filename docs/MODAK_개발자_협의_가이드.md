# MODAK 개발자 협의 가이드

> Flutter 앱 개발 착수 전 개발자와 협의할 항목 전체 정리
> 기준일: 2026년 4월 7일

---

## 기본 원칙

```
모든 계정 = 김주영 호주 법인 명의로 가입
개발자는 팀원으로 초대하는 방식
소스코드 소유권 = 김주영 (MODAK Group Pty Ltd)
```

---

## 1. 추가 개발 항목 — 비용/일정 확인

### 현재 견적서 기준 (3,000만원 / 16주 / 75 MD)

| 항목 | 현재 견적 | 상태 |
|------|---------|------|
| 전화번호 OTP 인증 | ✅ 포함 | 유지 |
| 구글 로그인 | ❌ 미포함 | 추가 요청 |
| 카카오 로그인 | ❌ 미포함 | 추가 요청 |
| 관리자 페이지 연동 | ❌ 미포함 | 추가 요청 |

### 추가 요청 항목

```
① 구글 로그인 추가
   - 추가 비용 / 일정 영향?

② 카카오 로그인 추가
   - 추가 비용 / 일정 영향?
   - 호주 법인으로 비즈 앱 등록 안 될 경우
     한국 사업자 번호 별도 안내 예정

③ 관리자 페이지 Supabase 연동
   - UI 완성본 있음 (HTML 파일 전달 가능)
   - 앱과 동일한 DB 공유 필요
   - 관리자 권한 체계 (슈퍼/에디터/뷰어) 포함
   - 추가 비용 / 일정?
```

---

## 2. 기술 스택 확인

### 견적서 기준 스택

| 영역 | 구성 |
|------|------|
| 앱 | Flutter (iOS/Android) |
| 인증 | Firebase Auth (전화번호 OTP) |
| 서버 | Node.js API |
| DB | Supabase (PostgreSQL) |
| 파일 | Firebase Storage |
| 알림 | Firebase Cloud Messaging (FCM) |
| 결제 | Stripe |
| 배포 | AWS ECS |

### 확인할 것들

```
① AWS ECS 월 운영 비용 예상
② Supabase 월 비용 예상
③ Firebase 대신 Supabase 선택 이유
④ 런칭 후 DB 마이그레이션 가능한지 (Firebase Firestore로)
```

### 월 운영 비용 예상 (런칭 후)

| 항목 | 예상 비용 |
|------|---------|
| AWS ECS | $50~150 USD/월 |
| Supabase Pro | $25 USD/월 |
| Firebase (Blaze) | $0~50 USD/월 |
| Google Maps API | $0 (무료 크레딧 내) |
| **합계** | **$75~225 USD/월** |

---

## 3. 계정 / 접근 권한

> 모든 계정은 김주영 법인 명의로 가입 후 개발자 초대

| 서비스 | 개발자 권한 | 가입 주체 |
|--------|-----------|---------|
| GitHub | Admin | 김주영 계정 |
| Firebase | Editor | 김주영 법인 계정 |
| Supabase | Admin | 김주영 법인 계정 |
| AWS | IAM 계정 생성 후 전달 | 김주영 법인 계정 |
| Google Cloud Console | Editor | 김주영 법인 계정 |
| Apple Developer | Admin | 김주영 법인 계정 |
| Google Play | Release Manager | 김주영 법인 계정 |
| Stripe | Developer | 김주영 법인 계정 |

---

## 4. 계약 관련

```
① 소스코드 소유권 — 김주영 (MODAK Group Pty Ltd)
② GitHub 레포 — 김주영 계정 기준으로 관리
③ 계약서 작성 가능한지
④ 선금 / 중도금 / 잔금 비율 협의 (일반적으로 30 / 40 / 30) or (50/50)
⑤ 런칭 후 무상 버그 수정 기간
⑥ 장기 유지보수 단가 기준
```

---

## 5. 개발 진행 방식

```
① 4월 15일 시작 가능한지
③ 소통 채널 (카카오톡 / 슬랙 / 이메일)
④ 산출물 목록
   ├── 소스코드 (GitHub)
   ├── 앱 바이너리 (iOS .ipa / Android .apk)
   ├── DB 스키마 문서
   ├── API 문서
   └── 배포 가이드
```

---

## 6. 개발 시작 전 준비할 것(김주영)

### 1순위 — 개발 시작 전 필수

```
├── 호주 법인 설립 완료 (MODAK Group Pty Ltd)
├── Google Workspace 가입 (법인 이메일)
│     예: admin@modak.com.au
├── 도메인 등록
│     modak.com.au / modak.app
├── Firebase 프로젝트 생성
├── Supabase 프로젝트 생성
├── GitHub 레포 생성
└── AWS 계정 생성
```

### 2순위 — 개발 중

```
├── Apple Developer 등록 ($99 USD/년)
├── Google Play 등록 ($25 USD 일회성)
├── Google Cloud Console (지도 API 활성화)
└── 카카오 개발자 가입
    ├── 호주 법인으로 비즈 앱 전환 시도
    └── 안 되면 한국 사업자로 진행
```

### 3순위 — 런칭 직전

```
└── Stripe 가입
    ├── ABN 필수
    └── 법인 은행 계좌 연결
```

---------------------------------------------------------------
## 초기 1년 계정/서비스 비용 합계

| 항목 | 금액 |
|------|------|
| Pty Ltd 등록 | $1597 AUD |
| 도메인 (2개) | ~$40 AUD |
| Google Workspace 12개월 | ~$200 AUD |
| Apple Developer | ~$150 AUD |
| Google Play | ~$38 AUD |
| **법인/계정 합계** | **~$2,025 AUD** |
| **개발 비용** | **3,000만원** |
| **월 운영 비용** | **$75~225 USD/월** |

---

*작성일: 2026년 4월*
*대상: MODAK Group Pty Ltd*
