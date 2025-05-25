# Language_Detector_Options_Reference

**기본 옵션 (Basic Options)**

| 옵션                 | 타입                                    | 기본값                               | 필수 | 설명                             |
| :------------------- | :-------------------------------------- | :----------------------------------- | :--- | :------------------------------- |
| `supportedLanguages` | `string[]`                              | `['en']`                             | 예   | 허용되는 언어 코드 배열            |
| `fallbackLanguage`   | `string`                                | `'en'`                               | 예   | 지원 언어 중 하나여야 하는 기본 언어 |
| `order`              | `감지 방식(DetectorType) 배열`            | `['querystring', 'cookie', 'header']`  | 아니오 | 언어 감지 순서                     |
| `debug`              | `boolean`                               | `false`                              | 아니오 | 디버깅 로그 활성화                 |

**감지 옵션 (Detection Options)**

| 옵션                  | 타입     | 기본값              | 설명                                     |
| :-------------------- | :------- | :------------------ | :--------------------------------------- |
| `lookupQueryString`   | `string` | `'lang'`            | 쿼리 파라미터 이름 (예: `?lang=ko`)        |
| `lookupCookie`        | `string` | `'language'`        | 쿠키 이름                                |
| `lookupFromHeaderKey` | `string` | `'accept-language'` | HTTP 요청 헤더 이름                      |
| `lookupFromPathIndex` | `number` | `0`                 | URL 경로 세그먼트 인덱스 (0부터 시작)      |

**쿠키 옵션 (Cookie Options)**

| 옵션                     | 타입                               | 기본값       | 설명                                     |
| :----------------------- | :--------------------------------- | :----------- | :--------------------------------------- |
| `caches`                 | `캐시 방식(CacheType) 배열 또는 false` | `['cookie']` | 감지된 언어를 캐시할 위치 (예: 쿠키)       |
| `cookieOptions.path`     | `string`                           | `'/'`        | 쿠키 경로                                |
| `cookieOptions.sameSite` | `'Strict' \| 'Lax' \| 'None'`      | `'Strict'`   | 쿠키의 SameSite 정책                     |
| `cookieOptions.secure`   | `boolean`                          | `true`       | HTTPS 연결에서만 쿠키 전송 여부          |
| `cookieOptions.maxAge`   | `number`                           | `31536000`   | 쿠키 만료 시간 (초 단위, 기본값 1년)     |
| `cookieOptions.httpOnly` | `boolean`                          | `true`       | 자바스크립트에서 쿠키 접근 불가 여부       |
| `cookieOptions.domain`   | `string`                           | `undefined`  | 쿠키 도메인 (설정 안 하면 현재 도메인)   |

**고급 옵션 (Advanced Options)**

| 옵션                      | 타입                          | 기본값       | 설명                                     |
| :------------------------ | :---------------------------- | :----------- | :--------------------------------------- |
| `ignoreCase`              | `boolean`                     | `true`       | 언어 코드 매칭 시 대소문자 구분 안 함      |
| `convertDetectedLanguage` | `(lang: string) => string`    | `undefined`  | 감지된 언어 코드를 변환하는 함수 (예: `en-US` -> `en`) |

**유효성 검사 및 오류 처리 (Validation & Error Handling)**
*   `fallbackLanguage`는 `supportedLanguages` 배열 안에 반드시 포함되어야 합니다 (설정 시 오류 발생). "기본 언어가 지원 목록에 없으면 그게 말이 되냐?"
*   `lookupFromPathIndex`는 0 이상의 정수여야 합니다.
*   잘못된 설정 값은 미들웨어 초기화 단계에서 오류를 발생시킵니다.
*   언어 감지에 실패하면, 오류 없이 조용히 `fallbackLanguage`를 사용합니다. "일단 기본빵으로 간다 이거지."

**자주 쓰는 레시피 (Common Recipes)**

*   **경로 기반 라우팅 (Path-Based Routing)**
    URL 경로의 일부를 사용해 언어를 감지합니다 (예: `/ko/home`, `/en/products`).
    ```javascript
    // 예시: /:lang/home 같은 라우트에서 언어 가져오기
    app.get('/:lang/home', (c) => {
      const lang = c.get('language') // 'en', 'ar' 등이 감지됨
      return c.json({ message: getLocalizedContent(lang) }) // 해당 언어의 콘텐츠 반환
    })
    ```

*   **다중 지원 언어 및 지역 코드 변환 (Multiple Supported Languages & Normalization)**
    다양한 지역별 언어 코드(예: `en-US`, `en-GB`)를 지원하고, 필요시 표준 형식으로 변환합니다.
    ```javascript
    languageDetector({
      supportedLanguages: ['en', 'en-GB', 'ar', 'ar-EG'], // 영국 영어, 이집트 아랍어 등 지원
      convertDetectedLanguage: (lang) => lang.replace('_', '-'), // '_'를 '-'로 정규화 (예: en_US -> en-US)
    })
    ```

---

**얘 뭐 하는 애냐?**
웹사이트 방문자가 어떤 언어를 쓰는지 알아채는 탐정 같은 놈. 사용자의 브라우저 설정, URL, 쿠키 등을 뒤져서 "이 사람 한국어 쓰네?" 아니면 "영어 사용자군!" 하고 알려준다. "너의 언어, 내가 다 읽어주마."

**왜 쓰는데?**
1.  **맞춤형 서비스**: 사용자 언어에 맞춰서 웹사이트 내용을 보여주려고. 한국인에겐 한국어, 미국인에겐 영어로. "고객님, 편한 언어로 모시겠습니다."
2.  **사용자 경험 (UX) 향상**: 사용자가 언어 설정 직접 안 바꿔도 되니까 편하지. 알아서 착착.

**언제 불려 나오냐?**
사용자가 웹사이트에 접속해서 페이지를 요청할 때마다, 서버가 응답 페이지를 만들기 직전에 이놈이 먼저 나서서 언어 탐색 작전을 펼친다.

**쓸 때 꿀팁 및 주의사항:**
*   **탐색 순서(`order`)가 중요**: `['querystring', 'cookie', 'header']` 순이면 URL 파라미터(`?lang=ko`)가 1순위, 없으면 쿠키, 그것도 없으면 브라우저 `Accept-Language` 헤더 순으로 뒤진다. "우선순위, 그거슨 진리."
*   **`fallbackLanguage`는 보험**: 지원 목록(`supportedLanguages`)에 `fallbackLanguage`가 없으면 설정부터 에러. "기본 언어는 확실하게 챙겨라."
*   **쿠키 설정은 꼼꼼하게**: `secure` (HTTPS 전용), `httpOnly` (JS 접근 차단), `sameSite` (CSRF 방어) 옵션 잘 만져서 보안과 프라이버시를 챙기자. "쿠키 하나도 허투루 다루지 마라."
*   **언어 코드 정규화**: `en-US`, `en_US`, `en`처럼 들쭉날쭉한 언어 코드를 `convertDetectedLanguage`로 통일하면 관리하기 편하다. `(lang) => lang.split('-')` 식으로 기본 언어만 뽑아 쓰거나. "표준이 없으면 개판."
*   **경로 기반 감지**: `/ko/home`, `/en/home`처럼 URL 경로에 언어 코드를 넣는 방식(`lookupFromPathIndex`)은 SEO에도 좋고 명확하지만, URL 구조 설계 잘해야 함. "주소만 봐도 언어가 딱!"