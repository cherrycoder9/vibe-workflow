# Language_Middleware_Hono

언어 탐지기(Language Detector) 미들웨어는 사용자가 선호하는 언어(로케일)를 다양한 출처에서 자동으로 파악해서 `c.get('language')`로 쓸 수 있게 해준다. 탐지 전략에는 쿼리 파라미터, 쿠키, 헤더, URL 경로 세그먼트가 포함된다. 국제화(i18n) 및 로케일별 콘텐츠 표시에 딱이다. "글로벌 서비스 만들 때 필수템."

**가져오기 (Import)**

```typescript
import { Hono } from 'hono'
import { languageDetector } from 'hono/language'
```

**기본 사용법 (Basic Usage)**
쿼리 문자열, 쿠키, 헤더 순서(기본값)로 언어를 감지하고, 없으면 영어로 대체한다:

```typescript
const app = new Hono()

app.use(
  languageDetector({
    supportedLanguages: ['en', 'ar', 'ja'], // 지원 언어 목록 (대체 언어 포함 필수)
    fallbackLanguage: 'en', // 대체 언어 (필수)
  })
)

app.get('/', (c) => {
  const lang = c.get('language')
  return c.text(`Hello! Your language is ${lang}`)
})
```

**클라이언트 예시 (Client Examples)**

```bash
# 경로(path)로 지정
curl http://localhost:8787/ar/home

# 쿼리 파라미터로 지정
curl http://localhost:8787/?lang=ar

# 쿠키로 지정
curl -H 'Cookie: language=ja' http://localhost:8787/

# 헤더로 지정
curl -H 'Accept-Language: ar,en;q=0.9' http://localhost:8787/
```

**기본 설정값 (Default Configuration)**

```typescript
export const DEFAULT_OPTIONS: DetectorOptions = {
  order: ['querystring', 'cookie', 'header'], // 탐지 순서: 쿼리 > 쿠키 > 헤더
  lookupQueryString: 'lang', // 쿼리 파라미터 이름
  lookupCookie: 'language', // 쿠키 이름
  lookupFromHeaderKey: 'accept-language', // 헤더 키 이름
  lookupFromPathIndex: 0, // URL 경로에서 언어 코드 위치 (예: /en/home 이면 0)
  caches: ['cookie'], // 감지된 언어 저장 위치 (기본은 쿠키)
  ignoreCase: true, // 대소문자 무시 여부
  fallbackLanguage: 'en', // 대체 언어
  supportedLanguages: ['en'], // 지원 언어 목록
  cookieOptions: { // 쿠키 설정
    sameSite: 'Strict', // 같은 사이트에서만 쿠키 전송
    secure: true, // HTTPS에서만 쿠키 전송
    maxAge: 365 * 24 * 60 * 60, // 쿠키 유효 기간 (1년)
    httpOnly: true, // 자바스크립트에서 쿠키 접근 불가
  },
  debug: false, // 디버그 모드
}
```

**주요 동작 방식 (Key Behaviors)**
*   **탐지 흐름 (Detection Workflow)**
    *   **순서 (Order)**: 기본적으로 다음 순서로 출처를 확인한다:
        1.  쿼리 파라미터 (`?lang=ar`)
        2.  쿠키 (`language=ar`)
        3.  `Accept-Language` 헤더
*   **캐싱 (Caching)**: 감지된 언어를 쿠키에 저장한다 (기본 1년).
*   **대체 (Fallback)**: 유효한 언어를 감지하지 못하면 `fallbackLanguage`를 사용한다 (`supportedLanguages` 목록에 반드시 포함되어야 함).

---

**얘 뭐 하는 애냐?**
웹사이트 방문자가 어떤 언어를 선호하는지 알아내서 개발자가 써먹기 좋게 `c.get('language')` 같은 곳에 쏙 넣어주는 똑똑한 미들웨어다. 사용자가 직접 URL에 `?lang=ko`처럼 쓰거나, 브라우저 설정에 맞춰서 "나 한국어 쓰는 사람인데?" 하고 알려주면 그걸 캐치해서 처리해준다. "다국어 서비스의 시작은 언어 감지부터."

**왜 쓰는데? (왜 이런 기능이 필요하고, 왜 알려주는데?)**
1.  **맞춤형 콘텐츠 제공**: 사용자 언어에 맞춰서 웹사이트 내용을 보여주려고. 한국인이 오면 한국어로, 미국인이 오면 영어로. "손님 맞춤 서비스, 기본 중의 기본."
2.  **개발 편의성**: 언어 감지 로직을 직접 짜려면 귀찮은데, 이걸 쓰면 설정 몇 줄로 끝. 개발자는 감지된 언어 가져다 쓰기만 하면 된다. "개발자는 핵심 로직에만 집중하라."
3.  **다양한 감지 방법 지원**: URL, 쿼리 파라미터, 쿠키, 브라우저 헤더 등 여러 경로로 언어 정보를 받을 수 있어서 유연하다. "어떤 방식으로든 알려만 주면 알아서 찾아낸다."

**언제 불려 나오냐? (언제 이 기능이 동작하냐?)**
웹 서버에 요청이 들어올 때마다, 본격적인 페이지 처리 전에 이 미들웨어가 먼저 나서서 "이 손님 언어 뭐 쓰시나?" 하고 스캔한다.

**쓸 때 꿀팁 및 주의사항:**
*   **`supportedLanguages`랑 `fallbackLanguage`는 짝꿍**: `fallbackLanguage`로 지정한 언어는 반드시 `supportedLanguages` 목록 안에 있어야 한다. 안 그러면 "지원도 안 하는 언어를 왜 기본값으로 쓰냐?" 하고 에러 난다. "기본 세팅은 신중하게."
*   **탐지 순서 커스터마이징**: `order` 옵션으로 언어 탐지 우선순위를 바꿀 수 있다. 예를 들어 URL 경로(`path`)에서 언어 코드를 먼저 확인하고 싶으면 `['path', 'querystring', 'cookie', 'header']` 이런 식으로 조정하면 된다. "우리 서비스는 이게 더 중요해!" 싶으면 순서를 바꿔라.
*   **쿠키 설정은 꼼꼼하게**: `cookieOptions`에서 쿠키 유효 기간(`maxAge`), 보안 설정(`secure`, `httpOnly`, `sameSite`) 등을 서비스 정책에 맞게 잘 설정해야 한다. 특히 `secure: true`는 HTTPS 환경에서만 쿠키를 보내므로, 개발 환경이 HTTP면 `false`로 하거나 HTTPS 환경을 구축해야 테스트할 수 있다. "쿠키 하나도 소홀히 다루지 마라."
*   **대소문자 무시 (`ignoreCase`)**: 기본값이 `true`라서 `?lang=KO`나 `?lang=ko`나 똑같이 한국어로 인식한다. 특별한 이유가 없다면 그냥 `true`로 두는 게 사용자 편의성 면에서 좋다. "깐깐하게 굴지 말고 웬만하면 다 받아줘라."
*   **URL 경로 탐지 인덱스 (`lookupFromPathIndex`)**: `/ko/products/awesome-item`처럼 URL 경로에 언어 코드를 넣을 때, 몇 번째 경로 세그먼트가 언어 코드인지 알려주는 설정이다. 위 예시처럼 맨 앞에 있으면 `0`. `/api/v1/ko/data`처럼 중간에 있으면 `2`가 된다. "경로에서 언어 찾을 땐 주소 잘 찍어줘."