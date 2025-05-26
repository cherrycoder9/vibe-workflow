# JWT_Auth_Middleware_Hono

JWT 인증 미들웨어는 JWT(JSON Web Token)를 사용해 토큰을 검증함으로써 인증 기능을 제공한다. 쿠키 옵션이 설정되어 있지 않으면, 이 미들웨어는 `Authorization` 헤더에서 토큰을 찾는다.

**참고**
클라이언트에서 보내는 `Authorization` 헤더에는 반드시 스킴(scheme)이 명시되어야 한다.
예시: `Bearer 내.토큰.값` 또는 `Basic 내.토큰.값` (JWT는 보통 `Bearer` 스킴을 쓴다.)

**가져오기 (Import)**

```typescript
import { Hono } from 'hono'
import { jwt } from 'hono/jwt'
import type { JwtVariables } from 'hono/jwt' // 타입스크립트 쓸 때, 페이로드 타입 추론용
```

**사용법 (Usage)**

`c.get('jwtPayload')`의 타입을 제대로 추론하려면 변수 타입을 지정해주는 게 좋다.

```typescript
// 변수 타입을 지정해서 c.get('jwtPayload') 타입 추론 지원
type Variables = JwtVariables

const app = new Hono<{ Variables: Variables }>()

app.use(
  '/auth/*', // '/auth/'로 시작하는 모든 경로에 적용
  jwt({
    secret: 'it-is-very-secret', // 실제론 이렇게 쓰면 안 되고, 환경 변수 등으로 안전하게 관리해야 함
  })
)

app.get('/auth/page', (c) => {
  // 이 핸들러는 위 미들웨어를 통과해야만 실행됨
  return c.text('인증된 사용자입니다!')
})
```

**페이로드 가져오기:**

미들웨어를 통과하면 요청 컨텍스트(`c`)에서 `jwtPayload`라는 이름으로 토큰의 내용(페이로드)을 꺼내 쓸 수 있다.

```typescript
const app = new Hono() // 타입 지정 안 해도 사용은 가능

app.use(
  '/auth/*',
  jwt({
    secret: 'it-is-very-secret',
  })
)

app.get('/auth/page', (c) => {
  const payload = c.get('jwtPayload') // 여기서 토큰 내용물 획득
  // 예시: { "sub": "1234567890", "name": "John Doe", "iat": 1516239022 }
  return c.json(payload)
})
```

**꿀팁**
`jwt()`는 그냥 미들웨어 함수다. 만약 환경 변수(예: `c.env.JWT_SECRET`)를 비밀키로 쓰고 싶으면, 다음과 같이 함수형 미들웨어 안에서 `jwt` 미들웨어를 호출하면 된다. "이게 바로 실전용 코드."

```typescript
app.use('/auth/*', async (c, next) => {
  // c.env 같은 환경 변수 접근은 Hono의 컨텍스트(c)를 통해 가능
  const jwtMiddleware = jwt({
    secret: c.env.JWT_SECRET, // 환경 변수에서 비밀키 가져오기
  })
  return jwtMiddleware(c, next) // 생성된 jwt 미들웨어 실행
})
```

**옵션 (Options)**

*   **`secret` (필수)**: `string`
    비밀 키 값. 이 키로 토큰의 서명을 검증한다. "이게 털리면 다 털리는 거다."
*   **`cookie` (선택)**: `string`
    이 값을 설정하면, `Authorization` 헤더 대신 쿠키 헤더에서 해당 이름(키)으로 토큰 값을 찾아 검증한다. 예를 들어 `cookie: 'auth_token'`으로 설정하면 `auth_token`이라는 이름의 쿠키에서 JWT를 찾는다.
*   **`alg` (선택)**: `string`
    토큰 검증에 사용할 알고리즘 타입. 기본값은 `HS256`.
    사용 가능한 알고리즘: `HS256` | `HS384` | `HS512` | `RS256` | `RS384` | `RS512` | `PS256` | `PS384` | `PS512` | `ES256` | `ES384` | `ES512` | `EdDSA`. "알고리즘은 많지만, 결국 쓰는 놈만 쓴다."

---

**얘 뭐 하는 애냐?**
JWT(JSON Web Token)라는 전자 신분증을 까보고 "너 진짜 우리 식구 맞냐?" 확인하는 문지기 역할이다. API 요청 같은 거 받을 때, 이놈이 먼저 나서서 토큰을 검사하고 통과시켜 줄지 말지 결정한다. "신분증 없으면 못 들어간다 이거야."

**왜 쓰는데? (왜 이런 기능이 필요하고, 왜 알려주는데?)**
1.  **상태 없음(Stateless) 인증**: 서버가 사용자 정보를 일일이 기억할 필요 없이, 토큰만 보고도 사용자를 인증할 수 있다. 서버 부담 줄고, 여러 서버로 확장하기도 편하다. "서버는 기억상실증 걸려도 괜찮아, 토큰만 있으면 돼."
2.  **보안**: 토큰이 암호화 서명되어 있어서 중간에 누가 내용을 바꿔치기하거나 위조하기 어렵다. (물론 비밀키 관리가 생명)
3.  **유연한 전달 방식**: HTTP `Authorization` 헤더에 `Bearer` 토큰으로 넣거나, 필요하면 쿠키로도 전달 가능.

**언제 불려 나오냐? (언제 이 기능이 동작하냐?)**
로그인해야만 접근할 수 있는 API 엔드포인트나 특정 페이지에 요청이 들어올 때마다 이 미들웨어가 호출된다. 보통 `/api/protected-route`나 `/user/profile`처럼 인증이 필요한 경로 앞에 딱 버티고 서서 검문한다.

**쓸 때 꿀팁 및 주의사항:**
*   **비밀키는 목숨처럼**: `secret` 값은 코드에 하드코딩하면 절대 안 된다. 환경 변수(`process.env.JWT_SECRET` 등)로 빼서 철저히 관리해야 한다. 털리면 인증 시스템 전체가 무너진다. "비밀번호를 공공장소에 써 붙이는 꼴."
*   **HTTPS는 국룰**: JWT 자체는 암호화된 게 아니라 그냥 인코딩된 거라 까보면 내용 다 보인다. HTTPS 안 쓰면 토큰이 중간에 탈취당해서 악용될 수 있다. "중요한 편지는 봉투에 넣고, 밀봉까지 해야지."
*   **토큰 만료 시간(Expiration)은 짧고 굵게**: 토큰에 `exp` 클레임(만료 시간)을 꼭 설정하고, 너무 길게 잡지 마라. 만약 토큰이 탈취돼도 피해를 최소화할 수 있다. 리프레시 토큰 전략과 함께 사용하면 좋다. "유통기한 지난 우유 마시면 배탈 나듯이, 만료된 토큰도 위험하다."
*   **알고리즘 선택 신중히**: 기본 `HS256`(대칭키)도 많이 쓰지만, 더 높은 보안이 필요하면 `RS256`(비대칭키) 같은 공개키/개인키 방식도 고려해볼 수 있다. 다만 키 관리가 더 복잡해진다. "강력한 무기일수록 다루기 어려운 법."
*   **페이로드에는 최소한의 정보만**: 토큰 페이로드(내용물)에는 사용자 비밀번호 같은 민감 정보 절대 넣지 말고, 사용자 식별자(ID), 권한 등 꼭 필요한 최소한의 정보만 담아라. "지갑에 현금 다발 들고 다니는 것보다 카드 한 장이 안전하다."
*   **쿠키로 토큰 전달 시 보안 옵션 필수**: `cookie` 옵션을 사용한다면, XSS(Cross-Site Scripting) 공격 방지를 위해 `HttpOnly` 플래그를, HTTPS 통신 강제를 위해 `Secure` 플래그를, CSRF(Cross-Site Request Forgery) 공격 완화를 위해 `SameSite` 속성을 반드시 설정해라. "보안은 옵션이 아니라 필수다."