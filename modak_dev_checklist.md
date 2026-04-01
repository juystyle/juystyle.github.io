# 🚀 모닥(MODAK) 앱 개발 전 준비 최종 목록

## ✅ 법인 설립 전 (지금 바로 가능)

| 서비스 | 가입 위치 | 개발자에게 전달할 것 | 비용 |
|--------|----------|-------------------|------|
| **Firebase** | console.firebase.google.com | google-services.json, GoogleService-Info.plist, Server Key | 무료 |
| **카카오 개발자** | developers.kakao.com | REST API 키, Native 앱 키 | 무료 |
| **카카오 오픈채팅** | 카카오톡 앱 | 오픈채팅 링크 URL | 무료 완료 |

---

## 🏢 법인 설립 후 (Pty Ltd + 비즈니스 카드 필요)

| 서비스 | 가입 위치 | 개발자에게 전달할 것 | 비용 |
|--------|----------|-------------------|------|
| **Google Maps** | console.cloud.google.com | API Key | 종량제 (월 $200 무료 크레딧) |
| **Google 로그인** | console.cloud.google.com | Client ID, Client Secret | 무료 |
| **카카오 로그인** | developers.kakao.com | REST API 키, Native 앱 키 | 무료 |
| **Apple Developer** | developer.apple.com | Team ID, Bundle ID, Key ID, .p8 파일 | $99 USD/년 |
| **Google Play Store** | play.google.com/console | 패키지명 (예: com.modak.app) | $25 USD 1회 |
| **SMS 인증 (Twilio)** | twilio.com | Account SID, Auth Token, 발신번호 | 종량제 |
| **Stripe (결제)** | stripe.com | Publishable Key, Secret Key | 수수료 1.7% + 30c |
| **AWS S3 (파일저장)** | aws.amazon.com | Access Key ID, Secret Access Key, 버킷명, 리전 | 종량제 |

---

## 🏢 호주 법인 설립 순서

1. **ABN 신청** → abr.gov.au (무료, 온라인 즉시)
2. **ACN + Pty Ltd 등록** → asic.gov.au (약 $576 AUD)
3. **비즈니스 은행 계좌 개설** → ANZ, NAB, Westpac 등
4. **비즈니스 카드 발급**
5. 이후 모든 API 서비스 법인 카드로 가입

---

## 📋 개발자에게 넘기는 순서 추천

### 1단계 - 지금 바로
- [ ] Firebase 가입 및 프로젝트 생성
- [ ] 카카오 개발자 등록 및 앱 키 발급
- [ ] 카카오 오픈채팅 링크 전달 (완료)

### 2단계 - 법인 설립 후
- [ ] Google Cloud (Maps + 로그인) 가입 및 API Key 발급
- [ ] Apple Developer 등록 ($99/년)
- [ ] Google Play Console 등록 ($25)
- [ ] Twilio 가입 및 발신번호 등록
- [ ] Stripe 가입 및 은행 계좌 연결
- [ ] AWS S3 버킷 생성

---

## 💡 참고사항

- **Stripe**: 호주 법인 + 호주 은행 계좌 필수 (수익 정산용)
- **Apple Developer**: 앱스토어에 법인명으로 표시됨 (예: MODAK Pty Ltd)
- **Google Maps**: 월 $200 무료 크레딧 있어서 초기엔 사실상 무료
- **SMS 인증**: 호주 번호는 Twilio 추천, 한국 번호면 알리고(aligo.in.kr) 추천
- **모든 API 비용**: 법인 카드로 결제 시 비즈니스 비용 처리 + GST 환급 가능
