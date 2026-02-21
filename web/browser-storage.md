## 🍪 Cookies

가장 오래된 브라우저 저장 방식으로, HTTP 요청 시 브라우저가 자동으로 관련 쿠키를 함께 서버에 전송한다는 특징이 있습니다.

- **용량:** 쿠키 하나당 약 4KB
- **만료:** 설정한 만료 시간까지 유지, 미설정 시 세션 종료 시 삭제
- **주요 용도:**
  - 로그인 세션 관리 (세션 ID 저장)
  - 사용자 추적 및 광고 타겟팅
  - 서버와 상태를 공유해야 하는 소규모 데이터
- **보안:** `HttpOnly` 플래그를 설정하면 JavaScript에서 접근 불가능해 XSS 공격 위험이 줄고, `Secure` 플래그는 HTTPS에서만 전송되도록 보장합니다.

---

## 📦 Local Storage

동일 출처(origin) 내에서 접근 가능하며, 만료 없이 사용자가 직접 삭제하거나 코드로 지우기 전까지 영구적으로 데이터가 유지됩니다.

- **용량:** 출처(origin)당 약 5MB
- **만료:** 없음 (영구 저장)
- **주요 용도:**
  - 다크 모드 같은 UI 테마/설정 저장
  - 사용자 환경설정
  - 서버에 보낼 필요 없는 캐시 데이터
- **특징:** 동기(synchronous) 방식으로 동작하며 문자열만 저장 가능 (객체는 `JSON.stringify` 필요)

---

## 🕐 Session Storage

탭 또는 창이 닫히면 데이터가 삭제되는 현재 세션 전용 임시 저장소입니다. Local Storage와 API가 동일하지만 생명 주기가 다릅니다.

- **용량:** 출처당 약 5MB
- **만료:** 탭/창 닫을 때 자동 삭제
- **주요 용도:**
  - 페이지 이동 중 임시 폼(form) 데이터 유지
  - 단계별 마법사(wizard) UI의 진행 상태
  - 탭 간에 공유되지 않아야 하는 데이터
- **특징:** 탭마다 별도로 분리되어 있어 탭 간 데이터 충돌이 없음

---

## 🗄️ IndexedDB

브라우저가 제공하는 로컬 NoSQL 데이터베이스로, 트랜잭션을 지원하고 비동기 방식으로 동작하여 브라우저의 메인 스레드를 차단하지 않음

- **용량:** 디스크 용량의 일정 비율(보통 10~60%)로, 수십 GB까지 가능
- **만료:** 없음 (영구 저장)
- **주요 용도:**
  - 오프라인 웹 앱의 대용량 데이터 캐시
  - 파일, 이미지, Blob 데이터 저장
  - 복잡한 구조의 데이터 관리 (인덱싱/검색 가능)
  - Service Worker와 함께 사용하는 경우 (Local Storage는 Service Worker에서 접근 불가)

---

## 🚀 Cache Storage

HTML, CSS, JavaScript, 이미지 등 웹 리소스를 로컬에 캐시하여 오프라인 접근 및 네트워크 요청 감소를 위한 전용 저장소입니다. Service Worker와 함께 PWA(Progressive Web App)에서 주로 활용됩니다.

- **용량:** IndexedDB와 유사하게 대용량 가능
- **주요 용도:**
  - 오프라인에서도 앱이 동작하도록 정적 파일 캐시
  - API 응답 캐시로 성능 향상

---

## 📊 한눈에 비교

| | Cookies | Local Storage | Session Storage | IndexedDB | Cache Storage |
|---|---|---|---|---|---|
| **용량** | ~4KB | ~5MB | ~5MB | 수GB | 수GB |
| **만료** | 설정 가능 | 없음 | 탭 닫으면 삭제 | 없음 | 없음 |
| **서버 전송** | ✅ 자동 | ❌ | ❌ | ❌ | ❌ |
| **데이터 형식** | 문자열 | 문자열 | 문자열 | 모든 타입 | 리소스 파일 |
| **비동기** | ❌ | ❌ | ❌ | ✅ | ✅ |

---

**참고 링크:**
- [MDN - Storage quotas and eviction criteria](https://developer.mozilla.org/en-US/docs/Web/API/Storage_API/Storage_quotas_and_eviction_criteria)
- [Cookies vs LocalStorage (SuperTokens)](https://supertokens.com/blog/cookies-vs-localstorage-for-sessions-everything-you-need-to-know)
- [Web Storage 종합 가이드 (Ramotion)](https://www.ramotion.com/blog/what-is-web-storage/)