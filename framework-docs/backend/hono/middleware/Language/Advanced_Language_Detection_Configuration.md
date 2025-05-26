# Advanced_Language_Detection_Configuration

**고급 설정**

**커스텀 감지 순서**
URL 경로에서 언어 감지하는 걸 최우선으로 하고 싶을 때 (예: `/ko/about`):

```typescript
app.use(
  languageDetector({
    order: ['path', 'cookie', 'querystring', 'header'], // 감지 순서: 경로 > 쿠키 > 쿼리스트링 > 헤더
    lookupFromPathIndex: 0, // URL 경로에서 언어 코드 위치. /ko/profile 이면 0번째 인덱스가 'ko'
    supportedLanguages: ['en', 'ar'], // 지원하는 언어 목록
    fallbackLanguage: 'en', // 기본으로 사용할 언어
  })
)
```

**언어 코드 변환**
복잡한 언어 코드(예: `en-US`)를 단순하게 정규화하고 싶을 때 (`en`으로):

```typescript
app.use(
  languageDetector({
    convertDetectedLanguage: (lang) => lang.split('-')[0], // 감지된 언어 코드를 변환하는 함수. 'en-US' -> 'en'
    supportedLanguages: ['en', 'ja'],
    fallbackLanguage: 'en',
  })
)
```

**쿠키 설정**

```typescript
app.use(
  languageDetector({
    lookupCookie: 'app_lang', // 언어 설정 저장할 쿠키 이름
    caches: ['cookie'], // 감지된 언어를 쿠키에 캐싱
    cookieOptions: {
      path: '/', // 쿠키 경로
      sameSite: 'Lax', // 쿠키 SameSite 정책 (Lax, Strict, None)
      secure: true, // HTTPS 연결에서만 쿠키 전송
      maxAge: 86400 * 365, // 쿠키 만료 시간 (초 단위, 여기선 1년)
      httpOnly: true, // 자바스크립트에서 쿠키 접근 불가
      domain: '.example.com', // (선택) 쿠키 적용 도메인. 앞에 . 찍으면 서브도메인 포함
    },
  })
)
```

쿠키 캐싱을 끄고 싶을 때:

```typescript
languageDetector({
  caches: false, // 쿠키 캐싱 안 함
})
```

**디버깅**
언어 감지 과정을 로그로 보고 싶을 때:

```typescript
languageDetector({
  debug: true, // 디버그 모드 켜기. "Detected from querystring: ar" 같은 로그 출력
})
```

---

**얘 뭐 하는 애냐?**
웹사이트 방문자의 언어를 어떤 순서로, 어떤 방법으로 알아낼지, 그리고 알아낸 언어 코드를 어떻게 다듬을지, 또 그걸 쿠키에 어떻게 저장할지 등을 아주 세세하게 설정할 수 있게 해주는 고급 옵션들이다. "언어 감지, A부터 Z까지 내 맘대로 주무르기!"

**왜 쓰는데? (왜 이런 기능이 필요하고, 왜 알려주는데?)**
1.  **정교한 언어 감지**: 기본 설정만으로는 부족할 때, 예를 들어 URL 경로(`example.com/ko/`)를 최우선으로 치거나, `en-US`나 `en-GB` 같은 복잡한 언어 코드를 그냥 `en`으로 통일하고 싶을 때 쓴다. "디테일이 살아있는 다국어 지원을 위해."
2.  **쿠키 제어**: 언어 설정을 쿠키에 저장할 때, 그 쿠키의 이름, 유효기간, 보안 옵션(`Secure`, `HttpOnly`, `SameSite`) 등을 서비스 정책에 맞게 커스터마이징하려고. "쿠키 하나도 허투루 다룰 수 없다."
3.  **문제 해결**: "왜 자꾸 이상한 언어가 뜨지?" 싶을 때 디버그 모드를 켜서 언어 감지 과정을 단계별로 추적하고 원인을 찾으려고. "개발자의 영원한 친구, 디버그 로그."

**언제 불려 나오냐? (언제 이 기능이 동작하냐?)**
Hono 앱에 요청이 들어올 때마다 `languageDetector` 미들웨어가 동작하는데, 이때 여기에 설정된 고급 옵션들이 적용돼서 언어 감지 로직이 돌아간다. 사용자가 어떤 언어로 콘텐츠를 봐야 할지 결정하는 바로 그 순간!

**쓸 때 꿀팁 및 주의사항:**
*   **감지 순서(`order`)가 핵심**: 사용자가 URL에 `ko`를 박아놨는데, 쿠키에 `en` 있다고 영어 보여주면 욕 나온다. `order` 배열에서 뭐가 우선순위인지 잘 정해야 한다. 보통은 URL 경로 > 쿼리스트링 > 쿠키 > 헤더 순이 무난. "사용자 의도 파악이 관건."
*   **언어 코드 변환(`convertDetectedLanguage`)은 신중하게**: `en-US`를 `en`으로 바꾸는 건 좋지만, `zh-CN`(중국 본토)과 `zh-TW`(대만)를 무턱대고 `zh`로 합치면 정치적으로 민감할 수도. 서비스 대상 지역과 언어의 특성을 고려해야 한다. "단순화가 능사는 아니다."
*   **쿠키 옵션, 보안과 직결**: `secure: true` (HTTPS 전용), `httpOnly: true` (JS 접근 불가), `sameSite: 'Lax'` 또는 `'Strict'`는 기본으로 깔고 가자. 개인정보는 소중하니까. `maxAge`도 너무 길게 잡으면 불필요. "보안은 아무리 강조해도 지나치지 않다."
*   **`lookupFromPathIndex`**: URL이 `/api/v1/en/data` 이런 식이면 `lookupFromPathIndex: 2`가 되겠지. URL 구조에 맞춰 정확히 설정해야 엉뚱한 거 안 읽어온다.
*   **디버깅(`debug: true`)은 개발 중에만**: 실제 서비스에서는 꺼야 불필요한 로그도 안 남고 성능에도 좋다. "개발 끝나면 스위치 내리자."
*   **캐싱 비활성화(`caches: false`)**: 특별한 이유(예: 매번 서버에서 언어 설정을 새로 받아와야 하는 경우)가 아니면 굳이 끌 필요는 없다. 사용자 편의성을 위해 보통은 켜두는 게 이득.