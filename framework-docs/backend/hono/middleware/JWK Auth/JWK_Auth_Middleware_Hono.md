# JWK_Auth_Middleware_Hono

JWK Auth 미들웨어는 JWK(JSON 웹 키)를 사용해서 토큰을 검증하고 요청을 인증하는 녀석이다. `Authorization` 헤더나, 설정했다면 쿠키 같은 다른 곳에서 토큰을 찾아낸다. 구체적으로는, 제공된 키로 토큰을 검증하고, `jwks_uri`가 지정되어 있으면 거기서 키를 가져오고, `cookie` 옵션이 설정되어 있으면 쿠키에서 토큰을 뽑아낸다.

**참고 (INFO)**

클라이언트에서 보내는 `Authorization` 헤더에는 정해진 스킴(scheme)이 있어야 한다.
예시: `Bearer my.token.value` 또는 `Basic my.token.value`

**가져오기 (Import)**

```typescript
import { Hono } from 'hono'
import { jwk } from 'hono/jwk'
```

**사용법 (Usage)**

```typescript
const app = new Hono()

// '/auth/*' 경로로 들어오는 모든 요청에 JWK 인증 적용
app.use(
  '/auth/*',
  jwk({
    // JWK 세트를 제공하는 URI. 보통 인증 서버의 .well-known 엔드포인트.
    jwks_uri: `https://${backendServer}/.well-known/jwks.json`,
  })
)

app.get('/auth/page', (c) => {
  return c.text('인증되셨습니다, 고객님.') // 성공 시 메시지
})
```

**페이로드(Payload) 가져오기:**

```typescript
const app = new Hono()

app.use(
  '/auth/*',
  jwk({
    jwks_uri: `https://${backendServer}/.well-known/jwks.json`,
  })
)

app.get('/auth/page', (c) => {
  // 검증된 JWT의 페이로드를 컨텍스트(c)에서 가져옴
  const payload = c.get('jwtPayload')
  // 페이로드를 JSON 형태로 응답 (예: { "sub": "1234567890", "name": "홍길동", "iat": 1516239022 })
  return c.json(payload)
})
```

**옵션 (Options)**

*   `keys` (선택 사항): `HonoJsonWebKey[] | (() => Promise<HonoJsonWebKey[]>)`
    공개 키 값들이나, 공개 키 값들을 반환하는 함수. "열쇠 꾸러미 직접 챙겨주는 옵션."
*   `jwks_uri` (선택 사항): `string`
    이 값을 설정하면, 해당 URI에서 JWK 세트를 가져오려고 시도한다. 키들이 담긴 JSON 응답을 기대하며, 가져온 키들은 `keys` 옵션으로 제공된 키 목록에 추가된다. "열쇠 어디 있는지 알려주면 알아서 가져옴."
*   `cookie` (선택 사항): `string`
    이 값을 설정하면, 해당 값을 키로 사용해서 쿠키 헤더에서 토큰 값을 가져와 검증한다. "쿠키에 숨겨둔 비상금 찾는 느낌."

---

**얘 뭐 하는 애냐?**
JSON 웹 키(JWK)를 이용해서 JWT(JSON Web Token)의 유효성을 검사해서 API 요청을 인증하는 미들웨어다. 쉽게 말해 "API 출입증 검사원"인데, 출입증(JWT)이 진짜인지, 위조된 건 아닌지 JWK라는 열쇠 목록으로 대조해서 확인한다.

**왜 쓰는데? (왜 이런 기능이 필요하고, 왜 알려주는데?)**
1.  **상태 비저장 인증 (Stateless Authentication)**: 서버에 사용자 세션 정보를 일일이 저장할 필요 없이 토큰만으로 인증을 처리할 수 있다. 서버 확장이 용이해지고, "나 누구랑 얘기했더라?" 같은 고민을 덜어준다.
2.  **표준 기반의 유연성**: JWT와 JWK는 업계 표준이라 다양한 시스템이나 프로그래밍 언어 간에 호환성이 좋다. `jwks_uri`를 통해 동적으로 키를 받아올 수 있어서 키 관리도 유연하다. "어디 가서든 먹히는 기술."
3.  **보안**: 토큰이 서명되어 있어서 중간에 누가 내용을 바꿔치기했는지 바로 알 수 있다.

**언제 불려 나오냐? (언제 이 기능이 동작하냐?)**
로그인해야 접근할 수 있는 API 엔드포인트에 요청이 들어올 때마다 이 미들웨어가 호출된다. 요청 헤더(`Authorization: Bearer <토큰>`)나 지정된 쿠키에서 JWT를 꺼내서 `jwks_uri` 또는 `keys` 옵션으로 제공된 JWK를 사용해 서명을 검증한다.

**쓸 때 꿀팁 및 주의사항:**
*   **`jwks_uri`는 HTTPS 필수**: JWK에는 공개키 정보가 들어있으니, 중간에 누가 훔쳐보거나 바꿔치기 못하게 무조건 HTTPS를 써야 한다. "개인 정보는 소중하니까."
*   **토큰 만료 시간(exp 클레임)은 필수**: JWT에는 반드시 만료 시간을 설정해서, 혹시 토큰이 유출되더라도 피해를 최소화해야 한다. 이 미들웨어는 기본적으로 만료된 토큰은 거부한다. "유통기한 지난 우유는 폐기처분."
*   **`alg` (알고리즘) 검증**: JWT 헤더에 있는 `alg` 필드가 내가 예상한 서명 알고리즘(예: RS256)과 일치하는지 확인하는 게 좋다. 특히 "none" 알고리즘은 절대 허용하면 안 된다. 이건 서명이 없다는 뜻이라 보안상 구멍. "비밀번호 없이 문 열어주는 격."
*   **키 순환(Key Rotation) 고려**: 보안을 더 빡세게 하려면 주기적으로 서명 키를 바꿔주는 게 좋다. `jwks_uri`를 사용하면 클라이언트 코드 변경 없이 서버에서 키만 갈아 끼우면 되니 편리하다.
*   **에러 처리 확실하게**: 토큰이 없거나, 형식이 틀렸거나, 서명이 안 맞거나, 만료됐거나 등등 다양한 인증 실패 상황에 대해 클라이언트가 알아먹기 쉽게 적절한 HTTP 상태 코드(401 Unauthorized, 403 Forbidden 등)와 에러 메시지를 보내줘야 한다. "문제 생기면 똑바로 알려줘야 삽질 안 한다."