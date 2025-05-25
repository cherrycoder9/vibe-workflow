`hono/cors`에서 `cors`를 가져옵니다.

```typescript
import { Hono } from 'hono'
import { cors } from 'hono/cors'
```

**CORS 미들웨어**
웹 API로 Cloudflare Workers를 사용하는 경우가 많고, 외부 프론트엔드 애플리케이션에서 이를 호출하는 경우도 많습니다. 이런 상황에서는 CORS(Cross-Origin Resource Sharing)를 구현해야 하는데, 이것도 미들웨어로 처리해 봅시다.

**사용법**

```typescript
const app = new Hono()

// CORS는 라우트보다 먼저 호출되어야 합니다.
// '/api/*' 경로에 대해 모든 출처(*)를 허용하는 기본 CORS 설정을 적용합니다.
app.use('/api/*', cors())

// '/api2/*' 경로에 대해 좀 더 상세한 CORS 설정을 적용합니다.
app.use(
  '/api2/*',
  cors({
    origin: 'http://example.com', // 'http://example.com' 출처만 허용
    allowHeaders: ['X-Custom-Header', 'Upgrade-Insecure-Requests'], // 허용할 요청 헤더 목록
    allowMethods: ['POST', 'GET', 'OPTIONS'], // 허용할 HTTP 메서드 목록
    exposeHeaders: ['Content-Length', 'X-Kuma-Revision'], // 브라우저에서 접근 가능하도록 노출할 응답 헤더 목록
    maxAge: 600, // Preflight 요청 결과를 캐시할 시간 (초 단위)
    credentials: true, // 자격 증명(쿠키, 인증 헤더 등)을 포함한 요청 허용 여부
  })
)

// '/api/abc' 경로에 대한 모든 HTTP 메서드 요청 처리
app.all('/api/abc', (c) => {
  return c.json({ success: true })
})

// '/api2/abc' 경로에 대한 모든 HTTP 메서드 요청 처리
app.all('/api2/abc', (c) => {
  return c.json({ success: true })
})
```

**여러 출처 허용하기:**

```typescript
// '/api3/*' 경로에 대해 여러 특정 출처를 배열로 지정하여 허용
app.use(
  '/api3/*',
  cors({
    origin: ['https://example.com', 'https://example.org'],
  })
)

// 또는 함수를 사용하여 동적으로 출처를 결정할 수도 있습니다.
app.use(
  '/api4/*',
  cors({
    // `c`는 `Context` 객체입니다.
    origin: (origin, c) => {
      // 요청한 출처(origin)가 '.example.com'으로 끝나면 해당 출처를 허용하고,
      // 그렇지 않으면 'http://example.com'을 기본값으로 사용합니다.
      return origin.endsWith('.example.com')
        ? origin
        : 'http://example.com'
    },
  })
)
```

**옵션**

*   `origin: string | string[] | (origin: string, c: Context) => string` (선택 사항)
    *   `Access-Control-Allow-Origin` CORS 헤더의 값입니다. 문자열, 문자열 배열, 또는 콜백 함수를 전달할 수 있습니다. 예: `origin: (origin) => (origin.endsWith('.example.com') ? origin : 'http://example.com')`. 기본값은 `*` (모든 출처 허용)입니다.
*   `allowMethods: string[]` (선택 사항)
    *   `Access-Control-Allow-Methods` CORS 헤더의 값입니다. 기본값은 `['GET', 'HEAD', 'PUT', 'POST', 'DELETE', 'PATCH']`입니다.
*   `allowHeaders: string[]` (선택 사항)
    *   `Access-Control-Allow-Headers` CORS 헤더의 값입니다. 기본값은 `[]` (빈 배열)입니다.
*   `maxAge: number` (선택 사항)
    *   `Access-Control-Max-Age` CORS 헤더의 값입니다. Preflight 요청 결과를 브라우저가 캐시할 시간을 초 단위로 지정합니다.
*   `credentials: boolean` (선택 사항)
    *   `Access-Control-Allow-Credentials` CORS 헤더의 값입니다. `true`로 설정하면 자격 증명(쿠키, HTTP 인증 등)을 사용한 요청을 허용합니다.
*   `exposeHeaders: string[]` (선택 사항)
    *   `Access-Control-Expose-Headers` CORS 헤더의 값입니다. 브라우저의 자바스크립트에서 접근할 수 있도록 허용할 응답 헤더 목록입니다. 기본값은 `[]` (빈 배열)입니다.

**환경에 따른 CORS 설정**
개발 환경이나 프로덕션 환경 등 실행 환경에 따라 CORS 설정을 조정하고 싶다면, 환경 변수에서 값을 주입하는 것이 편리합니다. 이렇게 하면 애플리케이션이 자신의 실행 환경을 알 필요가 없어집니다. 아래 예시를 참고하세요.

```typescript
app.use('*', async (c, next) => {
  // c.env.CORS_ORIGIN 환경 변수에서 허용할 출처 값을 가져옵니다.
  const corsMiddlewareHandler = cors({
    origin: c.env.CORS_ORIGIN, // 환경 변수로 출처를 동적으로 설정
  })
  // 생성된 CORS 미들웨어 핸들러를 실행합니다.
  return corsMiddlewareHandler(c, next)
})
```

---

**얘 뭐 하는 애냐?**
`cors` 미들웨어는 "내 API, 아무나 막 가져다 쓰지 마!" 혹은 "특별히 너네 사이트에서는 내 API 써도 돼!" 하고 허락 도장을 찍어주는 문지기야. 웹 브라우저는 보안 때문에 기본적으로 다른 도메인(출처)의 자원을 함부로 요청하지 못하게 막는데(Same-Origin Policy), CORS는 이 제한을 안전하게 풀어서 "이 도메인에서 오는 요청은 괜찮아~" 하고 서버가 알려주는 메커니즘이지. "옆집에서 우리 집 물건 빌려 가도 되는지 허락해주는 것과 비슷해."

**왜 쓰는데?**
1.  **분리된 프론트엔드-백엔드 구조**: 요즘 대세인 프론트엔드(React, Vue 등)와 백엔드 API 서버를 다른 도메인이나 포트에서 돌릴 때, 프론트엔드가 백엔드 API를 호출하려면 반드시 CORS 설정이 필요해. 이게 없으면 브라우저가 "어? 출처가 다르네? 위험해! 차단!" 하고 막아버리거든.
2.  **외부 서비스에 API 제공**: 내 API를 다른 개발자들이나 파트너사 웹사이트에서 사용할 수 있게 열어줄 때도 CORS 설정으로 특정 도메인만 허용해줄 수 있어. "아무나 말고, 허락된 사람만 우리 집 현관문 열쇠 복사해줄게."
3.  **보안 강화 (제한적이지만 중요)**: `origin` 옵션으로 허용할 출처를 명확히 지정하면, 아무 웹사이트나 내 API를 마구잡이로 호출해서 악용하는 걸 어느 정도 막을 수 있어. "우리 동네 사람 아니면 출입 금지!"

**언제 불려 나오냐?**
Hono 앱에서 특정 경로 또는 모든 경로에 대해 `app.use('/api/*', cors({ ... }))`처럼 미들웨어로 등록해서 써. 실제 API 로직이 실행되기 "전에" 이 CORS 미DL웨어가 먼저 끼어들어서, 들어오는 요청의 `Origin` 헤더를 보고 "음, 이 손님은 우리 가게 출입 허가 목록에 있군!" 또는 "넌 안돼!" 하고 판단한 뒤, 적절한 CORS 응답 헤더(`Access-Control-Allow-Origin` 등)를 붙여줘. 특히 `OPTIONS` 메서드로 오는 "Preflight 요청"을 자동으로 처리해주는 게 핵심이야.

**쓸 때 꿀팁 및 주의사항:**
*   **`origin` 설정이 핵심 중의 핵심**:
    *   `'*'` (기본값): "아무나 다 가져다 쓰세요!" 완전 개방. 보안상 민감한 API에는 절대 쓰면 안 돼.
    *   `'https://my-frontend.com'`: 딱 저 도메인만 허용. 제일 안전빵.
    *   `['https://site1.com', 'https://site2.com']`: 여러 개 지정 가능.
    *   함수 사용: `(origin) => origin.endsWith('.trusted-domains.com') ? origin : 'https://default-allowed.com'}`처럼 복잡한 로직으로 동적 허용 가능. "단골손님 리스트는 매일 업데이트됩니다."
*   **Preflight 요청 (`OPTIONS` 메서드) 이해하기**: 브라우저는 본 요청(GET, POST 등)을 보내기 전에 `OPTIONS` 메서드로 "이런 요청 보내도 괜찮을까요?" 하고 서버에 먼저 물어봐. CORS 미들웨어가 이 `OPTIONS` 요청에 "ㅇㅇ 괜찮음" 하고 응답해줘야 본 요청이 성공적으로 넘어갈 수 있어. `allowMethods`, `allowHeaders` 같은 옵션이 이 Preflight 응답에 쓰이지. "본 게임 전에 탐색전 하는 것과 같아."
*   **`credentials: true`는 신중하게**: 쿠키나 인증 헤더를 주고받아야 하는 API라면 `true`로 설정해야 하는데, 이때 `origin`을 `*`로 설정하면 에러 나. 보안상 특정 출처만 명시해야 해. "아무한테나 내 신용카드 정보 넘겨줄 순 없잖아?"
*   **`allowHeaders`는 필요한 것만**: 프론트에서 커스텀 헤더(`X-My-Token` 같은 거)를 보낸다면, 서버에서 `allowHeaders`에 그걸 꼭 추가해줘야 해. 안 그러면 Preflight 요청에서 막혀.
*   **`exposeHeaders`는 브라우저가 읽을 헤더**: 서버 응답에 커스텀 헤더를 담아서 보냈는데 프론트 자바스크립트에서 그걸 읽어야 한다? 그럼 `exposeHeaders`에 그 헤더 이름을 등록해줘야 해. 안 그러면 브라우저가 "이 헤더는 못 읽게 막혀있음!" 하고 숨겨버려.
*   **환경 변수로 설정 관리**: 개발 환경에서는 `origin: '*'`나 `localhost`를 허용하고, 운영 환경에서는 실제 서비스 도메인만 허용하도록 환경 변수를 쓰는 게 국룰이야. 코드 수정 없이 환경별로 다르게 적용할 수 있어서 개꿀. "상황 따라 변신하는 트랜스포머 CORS 설정!"
*   **순서가 중요**: `app.use(cors())`는 다른 라우트 핸들러보다 "앞에" 와야 해. 그래야 모든 요청에 대해 CORS 헤더를 먼저 처리할 수 있어. "문지기는 항상 입구에 있어야지."