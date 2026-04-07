# MODAK Admin — Firebase & Firestore 연동 가이드

> Firebase Authentication + Firestore 연동 완전 가이드  
> 현재 더미 데이터 → 실데이터로 교체하는 모든 단계를 다룹니다.

---

## 목차

1. [사전 준비](#1-사전-준비)
2. [Firebase 프로젝트 설정](#2-firebase-프로젝트-설정)
3. [SDK 초기화](#3-sdk-초기화)
4. [Authentication 연동](#4-authentication-연동)
5. [Firestore 컬렉션 구조](#5-firestore-컬렉션-구조)
6. [화면별 데이터 연동 가이드](#6-화면별-데이터-연동-가이드)
7. [보안 규칙 (Security Rules)](#7-보안-규칙-security-rules)
8. [배포 (Firebase Hosting)](#8-배포-firebase-hosting)
9. [연동 체크리스트](#9-연동-체크리스트)
10. [FAQ](#10-faq)

---

## 1. 사전 준비

### 필요한 것

- [ ] 기존 MODAK Flutter 앱이 사용 중인 **Firebase 프로젝트**
- [ ] Firebase Console 접근 권한 (프로젝트 오너 또는 편집자)
- [ ] 개발자에게 아래 3가지 수령:
  1. **Firebase Config 객체** (`apiKey`, `projectId` 등)
  2. **Firestore 컬렉션 구조** (Flutter 앱이 사용하는 컬렉션명)
  3. **관리자 UID** (Auth에서 관리자로 등록할 계정)

### 개발자에게 요청할 Config 형태

```js
const firebaseConfig = {
  apiKey:            "AIzaSy...",
  authDomain:        "modak-sydney.firebaseapp.com",
  projectId:         "modak-sydney",
  storageBucket:     "modak-sydney.appspot.com",
  messagingSenderId: "123456789",
  appId:             "1:123456789:web:abcdef"
};
```

---

## 2. Firebase 프로젝트 설정

### 2-1. 웹 앱 추가

1. [Firebase Console](https://console.firebase.google.com) → 기존 MODAK 프로젝트 선택
2. **프로젝트 설정** → **앱 추가** → 웹(`</>`) 선택
3. 앱 이름: `MODAK Admin`
4. **Firebase Hosting 설정도 함께** 체크 (권장)
5. Config 코드 복사 보관

### 2-2. Authentication 설정

**Firebase Console → Authentication → Sign-in method:**

```
✅ 이메일/비밀번호   — 사용 설정
```

### 2-3. Firestore 확인

**Firebase Console → Firestore Database** → 기존 컬렉션 구조 확인.

---

## 3. SDK 초기화

`modak_admin.html`의 기존 `<script>` 태그 **바로 위**에 추가합니다.

```html
<!-- Firebase SDK v10 (CDN) -->
<script type="module">
  import { initializeApp } from
    'https://www.gstatic.com/firebasejs/10.7.0/firebase-app.js';
  import {
    getFirestore, collection, getDocs, doc,
    getDoc, updateDoc, deleteDoc, addDoc,
    onSnapshot, query, orderBy, where, limit,
    Timestamp
  } from 'https://www.gstatic.com/firebasejs/10.7.0/firebase-firestore.js';
  import {
    getAuth, signInWithEmailAndPassword,
    signOut, onAuthStateChanged
  } from 'https://www.gstatic.com/firebasejs/10.7.0/firebase-auth.js';

  // ── Firebase 초기화 ──
  const firebaseConfig = {
    apiKey:            "여기에_apiKey",
    authDomain:        "여기에_authDomain",
    projectId:         "여기에_projectId",
    storageBucket:     "여기에_storageBucket",
    messagingSenderId: "여기에_messagingSenderId",
    appId:             "여기에_appId"
  };

  const app  = initializeApp(firebaseConfig);
  const db   = getFirestore(app);
  const auth = getAuth(app);

  // 전역 노출 — 기존 JS에서 window._db, window._auth 로 사용
  window._db   = db;
  window._auth = auth;
  window._fb   = {
    collection, getDocs, doc, getDoc,
    updateDoc, deleteDoc, addDoc,
    onSnapshot, query, orderBy, where, limit,
    Timestamp,
    signInWithEmailAndPassword, signOut, onAuthStateChanged
  };
</script>
```

---

## 4. Authentication 연동

### 4-1. doLogin() 교체

현재 더미 계정 방식을 Firebase Auth로 교체합니다.

```js
// ── 기존 코드 (삭제) ──────────────────────────────────────────
// const account = ADMIN_ACCOUNTS.find(a => a.email === email && a.pw === pw);

// ── 교체 코드 ────────────────────────────────────────────────
async function doLogin() {
  const email = document.getElementById('login-email')?.value?.trim();
  const pw    = document.getElementById('login-pw')?.value;
  const err   = document.getElementById('login-err');
  const btn   = document.querySelector('.login-btn');

  if (!email || !pw) {
    err.textContent = '이메일과 비밀번호를 입력해주세요';
    return;
  }

  btn.textContent = '로그인 중...';
  btn.disabled    = true;

  try {
    // 1. Firebase Auth 로그인
    const cred = await window._fb.signInWithEmailAndPassword(
      window._auth, email, pw
    );

    // 2. Firestore에서 관리자 권한 조회
    const adminSnap = await window._fb.getDoc(
      window._fb.doc(window._db, 'admins', cred.user.uid)
    );

    if (!adminSnap.exists()) {
      err.textContent = '관리자 권한이 없습니다';
      await window._auth.signOut();
      btn.textContent = '로그인';
      btn.disabled    = false;
      return;
    }

    const d = adminSnap.data();
    _session = {
      uid:    cred.user.uid,
      email:  cred.user.email,
      name:   d.name   || email.split('@')[0],
      role:   d.role   || 'viewer',   // 'super' | 'manager' | 'viewer'
      avatar: d.avatar || cred.user.email[0].toUpperCase(),
      color:  d.color  || '#1E6FFF',
    };

    // 3. 로그인 성공 처리
    const screen = document.getElementById('login-screen');
    if (screen) {
      screen.style.opacity    = '0';
      screen.style.transition = 'opacity .3s';
      setTimeout(() => { screen.style.display = 'none'; }, 300);
    }
    renderSideUser();
    updateNotifDot();
    showToast(`👋 환영합니다, ${_session.name}님!`);

  } catch (e) {
    const msgs = {
      'auth/user-not-found':     '등록되지 않은 이메일입니다',
      'auth/wrong-password':     '비밀번호가 올바르지 않습니다',
      'auth/invalid-credential': '이메일 또는 비밀번호가 올바르지 않습니다',
      'auth/too-many-requests':  '로그인 시도가 너무 많습니다. 잠시 후 다시 시도해주세요',
      'auth/user-disabled':      '비활성화된 계정입니다',
    };
    err.textContent = msgs[e.code] || `오류: ${e.message}`;
    btn.textContent = '로그인';
    btn.disabled    = false;
  }
}
```

### 4-2. doLogout() 교체

```js
async function doLogout() {
  if (!confirm('로그아웃 하시겠습니까?')) return;
  try {
    await window._fb.signOut(window._auth);
  } catch (e) {
    console.error('로그아웃 오류:', e);
  }
  _session = null;
  renderSideUser();
  const screen = document.getElementById('login-screen');
  if (screen) {
    screen.style.display    = 'flex';
    screen.style.opacity    = '1';
    screen.style.transition = 'opacity .3s';
  }
}
```

### 4-3. 세션 자동 복원 (onAuthStateChanged)

즉시실행 블록을 아래로 교체합니다.

```js
(function () {
  renderSideUser();   // 초기 UI

  if (!window._auth) return;

  window._fb.onAuthStateChanged(window._auth, async (user) => {
    if (!user) return;

    try {
      const snap = await window._fb.getDoc(
        window._fb.doc(window._db, 'admins', user.uid)
      );
      if (!snap.exists()) return;

      const d  = snap.data();
      _session = {
        uid: user.uid, email: user.email,
        name: d.name, role: d.role,
        avatar: d.avatar, color: d.color,
      };

      const screen = document.getElementById('login-screen');
      if (screen) screen.style.display = 'none';
      renderSideUser();
      updateNotifDot();
    } catch (e) {
      console.error('세션 복원 오류:', e);
    }
  });
})();
```

---

## 5. Firestore 컬렉션 구조

앱(Flutter)과 관리자 웹이 **동일한 Firestore**를 공유합니다.

```
modak-sydney (Firestore)
│
├── users/                    # 앱 회원
│   └── {uid}
│       ├── name:        string
│       ├── email:       string
│       ├── phone:       string | null
│       ├── region:      string          # "Strathfield" 등
│       ├── auth:        "verified" | "signed-in" | "guest"
│       ├── method:      "kakao" | "google" | "phone"
│       ├── status:      "active" | "suspended"
│       ├── warning:     number          # 경고 누적 횟수
│       ├── joinDate:    Timestamp
│       └── postCount:   number
│
├── posts/                    # 게시물
│   └── {postId}
│       ├── cat:         "알바·구인" | "렌트·쉐어" | "중고거래"
│       ├── title:       string
│       ├── body:        string
│       ├── authorId:    string          # users/{uid}
│       ├── authorName:  string
│       ├── region:      string
│       ├── price:       string
│       ├── status:      "active" | "hidden" | "done"
│       ├── reported:    number
│       ├── views:       number
│       ├── contacts:    number
│       ├── images:      string[]        # Storage URLs
│       └── createdAt:   Timestamp
│
├── reports/                  # 신고
│   └── {reportId}
│       ├── type:        "사기의심" | "허위정보" | "비매너" | "스팸" | "기타"
│       ├── sev:         "high" | "mid" | "low"
│       ├── title:       string
│       ├── reporterId:  string
│       ├── reporterName: string
│       ├── targetId:    string
│       ├── targetName:  string
│       ├── postId:      string
│       ├── detail:      string
│       ├── status:      "pending" | "done"
│       ├── action:      string | null
│       └── createdAt:   Timestamp
│
├── sponsors/                 # 스폰서 업체
│   └── {sponsorId}
│       ├── nm:          string          # 업체명
│       ├── e:           string          # 이모지
│       ├── tag:         string          # "" | "Premium" | "NEW"
│       ├── ring:        string[]        # 그라데이션 2색
│       ├── phone:       string
│       ├── addr:        string
│       ├── desc:        string
│       ├── plan:        "Premium 고정" | "일반 랜덤"
│       ├── fee:         number
│       ├── status:      "active" | "expiring" | "expired"
│       ├── ver:         boolean         # 검증 여부
│       ├── startDate:   string          # YYYY-MM-DD
│       └── endDate:     string
│
├── deals/                    # 딜·혜택
│   └── {dealId}
│       ├── nm:          string
│       ├── e:           string
│       ├── cat:         string
│       ├── sponsor:     string
│       ├── discount:    number
│       ├── type:        "%" | "$" | "free" | "gift"
│       ├── code:        string
│       ├── region:      string
│       ├── startDate:   string
│       ├── endDate:     string
│       ├── used:        number
│       ├── limit:       number
│       └── status:      "active" | "expiring" | "expired"
│
├── orders/                   # 광고 주문
│   └── {orderId}
│       ├── sponsor:     string
│       ├── plan:        string
│       ├── fee:         number
│       ├── period:      string
│       ├── requestDate: string
│       ├── status:      "pending" | "approved" | "expiring" | "expired" | "refunded"
│       ├── payMethod:   "bank" | "card" | "paypal"
│       ├── payRef:      string
│       └── note:        string
│
├── notices/                  # 공지사항
│   └── {noticeId}
│       ├── type:        "notice" | "event" | "policy"
│       ├── title:       string
│       ├── body:        string
│       ├── region:      string
│       ├── popup:       boolean
│       ├── pinned:      boolean
│       ├── status:      "published" | "draft"
│       ├── publishDate: string
│       ├── expireDate:  string
│       ├── views:       number
│       └── author:      string
│
├── inquiries/                # 문의
│   └── {inquiryId}
│       ├── type:        "bug" | "feature" | "report" | "partner" | "etc"
│       ├── title:       string
│       ├── author:      string
│       ├── email:       string
│       ├── status:      "pending" | "replied"
│       ├── reply:       string
│       └── createdAt:   Timestamp
│
├── admins/                   # 관리자 계정 (Auth UID 기준)
│   └── {uid}
│       ├── name:        string
│       ├── email:       string
│       ├── role:        "super" | "manager" | "viewer"
│       ├── status:      "active" | "inactive"
│       ├── avatar:      string
│       ├── color:       string
│       ├── twoFA:       boolean
│       ├── joinDate:    string
│       └── lastLogin:   Timestamp
│
└── pushHistory/              # 푸시 알림 이력
    └── {pushId}
        ├── title:       string
        ├── body:        string
        ├── target:      string          # "전체" | 지역명
        ├── sent:        number
        ├── opened:      number
        ├── openRate:    number
        ├── sentAt:      Timestamp
        └── status:      "sent" | "scheduled"
```

---

## 6. 화면별 데이터 연동 가이드

### 공통 유틸 함수

JS 최상단에 추가합니다.

```js
// ── Firestore 헬퍼 ───────────────────────────────────────────
async function fetchCol(colName, ...constraints) {
  const { collection, getDocs, query } = window._fb;
  const q = constraints.length
    ? query(collection(window._db, colName), ...constraints)
    : collection(window._db, colName);
  const snap = await getDocs(q);
  return snap.docs.map(d => ({ id: d.id, ...d.data() }));
}

async function updateDoc(colName, docId, data) {
  await window._fb.updateDoc(
    window._fb.doc(window._db, colName, docId), data
  );
}

async function deleteDocById(colName, docId) {
  await window._fb.deleteDoc(
    window._fb.doc(window._db, colName, docId)
  );
}

async function addNewDoc(colName, data) {
  const ref = await window._fb.addDoc(
    window._fb.collection(window._db, colName),
    { ...data, createdAt: window._fb.Timestamp.now() }
  );
  return ref.id;
}
```

---

### 6-1. 회원 관리

```js
let MEMBERS = [];   // 기존 더미 배열 선언 교체

async function loadMembers() {
  const { orderBy } = window._fb;
  MEMBERS = await fetchCol('users', orderBy('joinDate', 'desc'));
  reloadMembers();
}

async function handleMemberAction(id, action) {
  if (!checkWriteAccess()) return;
  const m = MEMBERS.find(x => x.id === id);
  if (!m) return;

  if (action === 'warn') {
    const w = (m.warning || 0) + 1;
    await updateDoc('users', id, { warning: w });
    m.warning = w;

  } else if (action === 'suspend') {
    await updateDoc('users', id, { status: 'suspended' });
    m.status = 'suspended';

  } else if (action === 'unsuspend') {
    await updateDoc('users', id, { status: 'active' });
    m.status = 'active';

  } else if (action === 'delete') {
    if (!confirm(`${m.name}님을 탈퇴 처리하시겠습니까?`)) return;
    await deleteDocById('users', id);
    MEMBERS.splice(MEMBERS.findIndex(x => x.id === id), 1);
    mSelected = null;
  }

  reloadMembers();
  showToast('✅ 처리 완료');
}

loadMembers();  // 페이지 로드 시 호출
```

---

### 6-2. 게시물 관리

```js
let POSTS = [];

async function loadPosts() {
  const { orderBy } = window._fb;
  POSTS = await fetchCol('posts', orderBy('createdAt', 'desc'));
  reloadPosts();
}

async function handlePostAction(id, action) {
  if (!checkWriteAccess('게시물 처리를')) return;
  const p = POSTS.find(x => x.id === id);
  if (!p) return;

  const updates = {
    hide:   { status: 'hidden' },
    show:   { status: 'active' },
    done:   { status: 'done',   done: true  },
    undone: { status: 'active', done: false },
  };

  if (action === 'delete') {
    if (!confirm('영구 삭제하시겠습니까?')) return;
    await deleteDocById('posts', id);
    POSTS.splice(POSTS.findIndex(x => x.id === id), 1);
    pSelected = null;
  } else if (updates[action]) {
    await updateDoc('posts', id, updates[action]);
    Object.assign(p, updates[action]);
  }

  reloadPosts();
  showToast('✅ 처리 완료');
}

loadPosts();
```

---

### 6-3. 신고 관리

```js
let REPORTS = [];

async function loadReports() {
  const { orderBy } = window._fb;
  REPORTS = await fetchCol('reports', orderBy('createdAt', 'desc'));
  reloadReports();
}

async function handleRvAction(id, action) {
  if (!checkWriteAccess()) return;
  const r = REPORTS.find(x => x.id === id);
  if (!r) return;

  const actionLabel = { pass:'통과', delete:'삭제', warn:'경고', ban:'정지' };

  await updateDoc('reports', id, {
    status: 'done',
    action: actionLabel[action] || action
  });
  r.status = 'done';
  r.action = actionLabel[action] || action;

  // 연관 처리
  if (action === 'delete' && r.postId)
    await updateDoc('posts',   r.postId,   { status: 'hidden'     });
  if (action === 'ban'    && r.targetId)
    await updateDoc('users',   r.targetId, { status: 'suspended'  });

  rvSelected = null;
  reloadReview();
  showToast('✅ 처리 완료');
}

loadReports();
```

---

### 6-4. 공지사항

```js
let NOTICES = [];

async function loadNotices() {
  const { orderBy } = window._fb;
  NOTICES = await fetchCol('notices', orderBy('publishDate', 'desc'));
  reloadNotices();
}

async function submitNotice(id, status) {
  if (!checkWriteAccess('공지 작성을')) return;

  const title      = document.getElementById('nc-title')?.value?.trim();
  const body       = document.getElementById('nc-body')?.value?.trim();
  const type       = document.getElementById('nc-type')?.value;
  const publishDate= document.getElementById('nc-date')?.value;
  const region     = document.getElementById('nc-region')?.value || '전체';
  const expireDate = document.getElementById('nc-expire')?.value || '';
  const popup      = document.getElementById('nc-popup')?.checked;
  const pinned     = document.getElementById('nc-pinned')?.checked;

  if (!title || !body) { alert('제목과 내용을 입력해주세요'); return; }

  const payload = { title, body, type, publishDate, region,
                    expireDate, popup, pinned, status };

  if (id) {
    await updateDoc('notices', id, payload);
    const n = NOTICES.find(x => x.id === id);
    if (n) Object.assign(n, payload);
    ncSelected = id;
  } else {
    const newId = await addNewDoc('notices', {
      ...payload, views: 0, author: _session?.name || 'Admin'
    });
    NOTICES.unshift({ id: newId, ...payload, views: 0 });
    ncSelected = newId;
  }

  ncMode = 'list';
  showToast(status === 'published' ? '✅ 공지가 발행됐습니다' : '📝 초안이 저장됐습니다');
  reloadNotices();
}

loadNotices();
```

---

### 6-5. 실시간 리스너 — 대시보드 신고 배지

```js
function setupRealtimeListeners() {
  const { collection, query, where, orderBy, onSnapshot, limit } = window._fb;

  const q = query(
    collection(window._db, 'reports'),
    where('status', '==', 'pending'),
    orderBy('createdAt', 'desc'),
    limit(20)
  );

  onSnapshot(q, (snap) => {
    const count = snap.size;
    // 사이드바 배지 업데이트
    const badge = document.querySelector('[data-page="reports"] .nav-badge');
    if (badge) badge.textContent = count || '';

    // 대시보드 KPI 업데이트
    REPORTS = snap.docs.map(d => ({ id: d.id, ...d.data() }));
    updateNotifDot();
  });
}

// 로그인 성공 후 호출
setupRealtimeListeners();
```

---

## 7. 보안 규칙 (Security Rules)

**Firebase Console → Firestore Database → 규칙** 탭에 붙여넣습니다.

```js
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {

    // ── 헬퍼 함수 ─────────────────────────────────────────────
    function isSignedIn() {
      return request.auth != null;
    }

    function isAdmin() {
      return isSignedIn() &&
        exists(/databases/$(database)/documents/admins/$(request.auth.uid));
    }

    function role() {
      return get(/databases/$(database)/documents/admins/$(request.auth.uid))
               .data.role;
    }

    function isWriteRole() {
      return isAdmin() && role() in ['super', 'manager'];
    }

    function isSuper() {
      return isAdmin() && role() == 'super';
    }

    // ── 회원 ───────────────────────────────────────────────────
    match /users/{uid} {
      allow read:   if isAdmin();
      allow update: if isWriteRole();
      allow delete: if isSuper();
      allow create: if false;           // 앱(Flutter)에서만 생성
    }

    // ── 게시물 ─────────────────────────────────────────────────
    match /posts/{postId} {
      allow read:   if isAdmin();
      allow update: if isWriteRole();
      allow delete: if isWriteRole();
      allow create: if false;
    }

    // ── 신고 ───────────────────────────────────────────────────
    match /reports/{reportId} {
      allow read:   if isAdmin();
      allow update: if isWriteRole();
      allow delete: if isSuper();
      allow create: if false;
    }

    // ── 스폰서 ─────────────────────────────────────────────────
    match /sponsors/{sponsorId} {
      allow read:   if isAdmin();
      allow write:  if isWriteRole();
    }

    // ── 딜·혜택 ────────────────────────────────────────────────
    match /deals/{dealId} {
      allow read:   if isAdmin();
      allow write:  if isWriteRole();
    }

    // ── 광고 주문 ──────────────────────────────────────────────
    match /orders/{orderId} {
      allow read:   if isAdmin();
      allow update: if isWriteRole();
      allow delete: if isSuper();
      allow create: if false;
    }

    // ── 공지사항 ───────────────────────────────────────────────
    match /notices/{noticeId} {
      allow read:   if true;            // 앱 사용자도 읽기 가능
      allow write:  if isWriteRole();
    }

    // ── 문의 ───────────────────────────────────────────────────
    match /inquiries/{inquiryId} {
      allow read:   if isAdmin();
      allow update: if isWriteRole();
      allow create: if true;            // 앱 사용자가 문의 생성
      allow delete: if isSuper();
    }

    // ── 관리자 계정 ────────────────────────────────────────────
    match /admins/{adminId} {
      allow read:   if isSignedIn() &&
                      (request.auth.uid == adminId || isSuper());
      allow write:  if isSuper();
    }

    // ── 푸시 이력 ──────────────────────────────────────────────
    match /pushHistory/{pushId} {
      allow read:   if isAdmin();
      allow write:  if isWriteRole();
    }
  }
}
```

> **Rules Playground**에서 반드시 테스트 후 적용하세요.

---

## 8. 배포 (Firebase Hosting)

### 8-1. Firebase CLI 설치 및 로그인

```bash
npm install -g firebase-tools
firebase login
```

### 8-2. 프로젝트 초기화

```bash
cd modak-admin          # modak_admin.html 이 있는 디렉토리
firebase init hosting
```

설정:
```
? Which Firebase project?    → modak-sydney (기존 프로젝트 선택)
? Public directory?          → .
? Single-page app?           → No
? GitHub Actions deploys?    → No
```

### 8-3. `.firebaseignore` 작성

```
# .firebaseignore
*.md
*.json
node_modules/
.git/
```

### 8-4. 배포

```bash
firebase deploy --only hosting
# 완료 후:
# ✔  Hosting URL: https://modak-sydney.web.app
```

### 8-5. 커스텀 도메인 연결 (선택)

**Firebase Console → Hosting → 커스텀 도메인 추가**

```
admin.modak.com.au  →  modak-sydney.web.app
```

---

## 9. 연동 체크리스트

### Phase 1 — 런치 전 필수

- [ ] Firebase Config 수령 및 `modak_admin.html`에 적용
- [ ] `doLogin()` Firebase Auth로 교체
- [ ] `doLogout()` Firebase Auth로 교체
- [ ] `onAuthStateChanged` 세션 자동 복원 적용
- [ ] `admins` 컬렉션에 관리자 계정 문서 생성
- [ ] Firestore 보안 규칙 적용 및 테스트
- [ ] 회원 (`users`) 연동 — `loadMembers()`
- [ ] 게시물 (`posts`) 연동 — `loadPosts()`
- [ ] 신고 (`reports`) 연동 — `loadReports()`
- [ ] 스폰서 (`sponsors`) 연동 — `loadSponsors()`
- [ ] 딜·혜택 (`deals`) 연동 — `loadDeals()`
- [ ] 광고 주문 (`orders`) 연동 — `loadOrders()`
- [ ] 공지사항 (`notices`) 연동 — `loadNotices()`
- [ ] Firebase Hosting 배포
- [ ] 더미 계정 (`ADMIN_ACCOUNTS`) 코드 삭제

### Phase 2 — 런치 후 선택

- [ ] 문의 (`inquiries`) 연동
- [ ] 푸시 알림 — FCM + Cloud Functions 연동
- [ ] 통계 — Cloud Functions 일별 집계 또는 BigQuery 연동
- [ ] 실시간 대시보드 — `onSnapshot` 리스너 적용
- [ ] 다크모드 설정 — `admins/{uid}/settings` Firestore 저장
- [ ] 콘텐츠 자동 검수 — Cloud Functions 금칙어 필터 연동

### 보안 강화

- [ ] Firestore 보안 규칙 Rules Playground 전체 테스트
- [ ] Firebase Auth 이메일 인증 활성화
- [ ] Firebase App Check 적용
- [ ] 관리자 2FA (Firebase MFA) 설정
- [ ] 더미 데이터 변수 전체 제거

---

## 10. FAQ

### Q. Firebase config가 HTML 파일에 노출되면 보안 문제가 없나요?

`apiKey`는 프로젝트 식별자일 뿐 비밀 키가 아닙니다. 실제 보안은 **Firestore Security Rules**로 제어합니다. Rules가 올바르게 설정되어 있으면 인증된 관리자만 데이터에 접근할 수 있습니다.

---

### Q. Flutter 앱과 관리자 웹이 같은 Firestore를 쓰나요?

네, 동일한 Firestore 데이터베이스를 공유합니다. 관리자 웹에서 회원을 정지하면 앱에서도 즉시 반영됩니다. Security Rules로 관리자와 일반 사용자의 접근 권한을 구분합니다.

---

### Q. 기존 더미 데이터(MEMBERS, POSTS 등)는 어떻게 처리하나요?

```js
// 변경 전 (삭제)
const MEMBERS = [ { id:'u001', ... }, { id:'u002', ... }, ... ];

// 변경 후
let MEMBERS = [];
// loadMembers() 호출로 Firestore에서 채움
```

---

### Q. 관리자 계정(admins 컬렉션)은 어떻게 처음 만드나요?

Firebase Admin SDK를 사용하는 초기화 스크립트를 한 번 실행합니다.

```js
// init-admin.js (Node.js, 1회 실행)
const admin = require('firebase-admin');
const sa    = require('./serviceAccount.json');   // Firebase Console에서 다운로드

admin.initializeApp({ credential: admin.credential.cert(sa) });
const db   = admin.firestore();
const auth = admin.auth();

async function createInitialAdmin() {
  // 1. Auth 계정 생성
  const user = await auth.createUser({
    email:       'admin@modak.com',
    password:    'newSecurePassword123!',  // 실제 비밀번호로 변경
    displayName: 'Admin',
  });

  // 2. Firestore admins 문서 생성
  await db.collection('admins').doc(user.uid).set({
    name:      'Admin',
    email:     'admin@modak.com',
    role:      'super',
    status:    'active',
    avatar:    'AD',
    color:     '#E24B4A',
    twoFA:     false,
    joinDate:  '2025-04-01',
    lastLogin: admin.firestore.Timestamp.now(),
  });

  console.log('✅ 관리자 생성 완료 — UID:', user.uid);
}

createInitialAdmin().catch(console.error);
```

```bash
node init-admin.js
```

---

### Q. 푸시 알림(FCM) 연동은 어떻게 하나요?

Cloud Functions를 통한 발송을 권장합니다.

```js
// functions/index.js (Firebase Cloud Functions)
const admin     = require('firebase-admin');
const functions = require('firebase-functions');

admin.initializeApp();

exports.sendPush = functions.https.onCall(async (data, context) => {
  // 관리자 권한 확인
  if (!context.auth) throw new functions.https.HttpsError('unauthenticated', '로그인 필요');

  const adminDoc = await admin.firestore()
    .collection('admins').doc(context.auth.uid).get();
  if (!adminDoc.exists || !['super','manager'].includes(adminDoc.data().role))
    throw new functions.https.HttpsError('permission-denied', '권한 없음');

  // FCM 발송
  const message = {
    notification: { title: data.title, body: data.body },
    topic: data.target === '전체' ? 'all' : `region_${data.target}`,
  };

  const response = await admin.messaging().send(message);

  // 이력 저장
  await admin.firestore().collection('pushHistory').add({
    title:    data.title,
    body:     data.body,
    target:   data.target,
    status:   'sent',
    sentAt:   admin.firestore.Timestamp.now(),
  });

  return { success: true, messageId: response };
});
```

관리자 웹에서 호출:
```js
// modak_admin.html 내부
import { getFunctions, httpsCallable } from
  'https://www.gstatic.com/firebasejs/10.7.0/firebase-functions.js';

const functions   = getFunctions(app, 'us-central1');
const callSendPush = httpsCallable(functions, 'sendPush');

async function sendPush(mode) {
  if (!checkWriteAccess('푸시 발송을')) return;
  const title = document.getElementById('push-title')?.value?.trim();
  const body  = document.getElementById('push-body')?.value?.trim();
  if (!title || !body) { alert('제목과 내용을 입력해주세요'); return; }

  try {
    await callSendPush({ title, body, target: pushRegion || '전체' });
    showToast('📨 푸시 알림이 발송됐습니다');
    setPushTab('history');
  } catch (e) {
    alert('발송 오류: ' + e.message);
  }
}
```

---

### Q. 통계 데이터는 어떻게 집계하나요?

**Cloud Functions 자동 집계 (권장)**:

```js
// 매일 자정에 실행
exports.dailyStats = functions.pubsub
  .schedule('0 0 * * *')
  .timeZone('Australia/Sydney')
  .onRun(async () => {
    const db    = admin.firestore();
    const today = new Date().toISOString().split('T')[0];

    // 오늘 신규 가입자 수 집계
    const snap = await db.collection('users')
      .where('joinDate', '>=', admin.firestore.Timestamp.fromDate(
        new Date(today)
      )).get();

    await db.collection('stats').doc(today).set({
      newUsers: snap.size,
      updatedAt: admin.firestore.Timestamp.now(),
    });

    console.log(`✅ 통계 집계 완료: ${today} — 신규 ${snap.size}명`);
  });
```

---

*MODAK Admin Firebase 연동 가이드 v1.0*  
*Firebase SDK 10.x 기준 | 작성일: 2025년 4월*
