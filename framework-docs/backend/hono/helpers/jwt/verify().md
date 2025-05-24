`verify()`
이 함수는 JWT(JSON Web Token) 토큰이 진짜인지, 그리고 아직 유효한지를 확인합니다. 토큰이 중간에 누가 건드리지 않았는지, 그리고 만약 페이로드 유효성 검사(Payload Validation) 규칙을 추가했다면 그 규칙에도 맞는지까지 따져봅니다. "이 신분증 위조된 거 아니죠? 유효기간은 안 지났구요?" 하고 검문하는 역할이죠.

```typescript
verify(
  token: string,          // 확인할 JWT 토큰 (문자열)
  secret: string,         // 서명할 때 썼던 그 비밀키 (문자열)
  alg?: 'HS256'          // 사용한 알고리즘 (선택 사항, 기본값은 'HS256')
): Promise<any>;         // 검증 성공 시 토큰에 담겨있던 내용(페이로드)을 Promise로 반환
```

**예시**

```typescript
// hono/jwt에서 verify 함수를 소환!
import { verify } from 'hono/jwt'

// 검증할 토큰 (실제로는 클라이언트가 보내준 토큰이겠지?)
const tokenToVerify = '여기에.클라이언트가.보내준.토큰이.들어감'
// 토큰 만들 때 썼던 바로 그 비밀키! 이게 다르면 검증 실패다.
const secretKey = '나만아는비밀키123!'

// 비밀키로 토큰을 검증하고, 이상 없으면 내용물을 꺼내줘! (비동기 처리 필수)
const decodedPayload = await verify(tokenToVerify, secretKey)

// 내용물 확인! (예: 사용자 ID, 권한 정보 등)
console.log(decodedPayload)
```

**옵션 상세 설명**

*   `token: string` (필수)
    *   검증할 JWT 토큰 문자열입니다. 보통 클라이언트가 HTTP 요청 헤더(`Authorization: Bearer <token>`)에 담아 보냅니다.
*   `secret: string` (필수)
    *   JWT를 서명하거나 검증할 때 사용하는 비밀키입니다. 토큰을 발급할 때 사용했던 비밀키와 **반드시** 같아야 합니다. 이게 다르면 "너 누구냐?" 하면서 검증 실패 처리됩니다.
*   `alg?: AlgorithmTypes` (선택 사항)
    *   JWT 서명/검증에 사용된 알고리즘입니다. 기본값은 `HS256` (HMAC SHA256)입니다. 만약 다른 알고리즘(예: `HS384`, `HS512`, 또는 RSA/ECDSA 계열)으로 토큰을 만들었다면, 여기서 그 알고리즘을 명시해줘야 합니다. 지금 `hono/jwt`의 `verify`는 `HS256`만 명시적으로 보여주고 있네요. (다른 알고리즘 지원 여부는 Hono 문서를 더 자세히 파봐야 알겠지만, 일단 예시로는 HS256만 나왔으니 참고!)

---

**얘 뭐 하는 애냐?**
`verify()`는 클라이언트가 들고 온 JWT 토큰이 "우리가 발급한 게 맞는지(진위 여부)" 그리고 "아직 써도 되는 건지(유효 기간 등)"를 꼼꼼하게 따져보는 문지기 같은 녀석입니다. 이 검문을 통과해야만 토큰 안에 담긴 정보(페이로드)를 믿고 쓸 수 있는 거죠. "출입증 확인하고, 유효기간 체크하고, 위조 안 됐는지까지 보는 삼중 보안!"

**왜 쓰는데?**
1.  **인증 (Authentication)**: 사용자가 로그인할 때 JWT 토큰을 발급해주고, 그 이후 사용자가 API 요청할 때마다 이 토큰을 `verify()`로 검증해서 "아, 이 사람은 아까 로그인했던 그 사람이 맞구나!" 하고 인증 처리를 합니다. 서버가 매번 사용자 정보를 기억할 필요 없이 토큰만으로 인증이 가능해지죠 (Stateless).
2.  **인가 (Authorization)**: 토큰의 페이로드 안에 사용자 역할(role)이나 권한(permission) 정보를 넣어두고, `verify()`로 토큰을 까본 뒤 "이 사용자는 관리자 권한이 있으니 이 API를 쓰게 해주고, 저 사용자는 일반 사용자니 저 API는 막아야겠다!" 하는 식으로 접근 제어를 할 수 있습니다.
3.  **데이터 무결성 보장**: JWT는 서명(signature)을 통해 토큰 내용이 중간에 변경되지 않았음을 보장합니다. `verify()`는 이 서명을 비밀키로 대조해서 토큰이 위변조되지 않았는지 확인합니다. "봉인씰 뜯어졌는지 확인하는 격!"
4.  **유효 기간 관리**: 토큰에는 만료 시간(expiration time, `exp`)을 설정할 수 있는데, `verify()`는 이 만료 시간을 자동으로 체크해서 이미 만료된 토큰이면 에러를 발생시킵니다. "유통기한 지난 음식은 폐기처분!"

**언제 불려 나오냐?**
주로 사용자의 인증이 필요한 API 엔드포인트의 미들웨어나 핸들러 시작 부분에서 호출됩니다. 클라이언트가 요청 헤더에 JWT를 담아 보내면, 서버는 이 토큰을 꺼내서 `verify()` 함수에 넣고 돌려봅니다.

```typescript
// Hono 앱 예시
import { Hono } from 'hono'
import { verify } from 'hono/jwt' // verify 함수 임포트

const app = new Hono()
const SECRET = '나만의-비밀-시크릿-키' // 실제로는 환경변수에서 가져와야 함!

// 인증이 필요한 라우트에 미들웨어로 적용하는 예시
app.use('/api/secure/*', async (c, next) => {
  const authHeader = c.req.header('Authorization')
  if (!authHeader || !authHeader.startsWith('Bearer ')) {
    return c.json({ error: '인증 토큰이 없거나 형식이 잘못되었습니다.' }, 401)
  }
  const token = authHeader.substring(7) // "Bearer " 부분 제거

  try {
    const payload = await verify(token, SECRET)
    c.set('user', payload) // 검증된 사용자 정보를 컨텍스트에 저장
    await next()
  } catch (error) {
    // 토큰이 유효하지 않거나 만료된 경우
    return c.json({ error: '유효하지 않은 토큰입니다.' }, 401)
  }
})

app.get('/api/secure/profile', (c) => {
  const user = c.get('user') // 미들웨어에서 저장한 사용자 정보 사용
  return c.json({ message: `환영합니다, ${user.username}님! 당신의 프로필 정보입니다.` })
})
```

**쓸 때 꿀팁 및 주의사항:**
*   **비밀키(Secret) 관리는 생명**: `secret`은 절대 코드에 하드코딩하거나 깃허브 같은 곳에 올리면 안 됩니다. 환경 변수로 관리하고, 아주 복잡하고 예측 불가능한 문자열을 사용해야 합니다. "비밀번호 아무거나 쓰면 집 비밀번호 공개하는 꼴!"
*   **에러 처리 필수**: `verify()`는 토큰이 유효하지 않거나(서명 불일치, 만료 등) 문제가 있으면 에러를 냅다 던집니다. `try...catch` 블록으로 감싸서 에러 상황에 맞게 적절한 HTTP 응답(주로 401 Unauthorized 또는 403 Forbidden)을 클라이언트에게 보내줘야 합니다. "문제 생기면 그냥 죽지 말고, 무슨 문제인지 알려는 줘야지."
*   **알고리즘 일치 확인**: 토큰을 생성(sign)할 때 사용한 알고리즘과 `verify()` 할 때 지정한 알고리즘이 같아야 합니다. 기본값은 `HS256`이지만, 다른 걸 썼다면 명시적으로 `alg` 옵션에 넣어줘야 합니다.
*   **페이로드 유효성 검사 (Payload Validation)**: `hono/jwt`의 `sign` 함수에는 `exp` (만료 시간), `nbf` (Not Before, 이 시간 전에는 유효하지 않음), `iat` (Issued At, 발급 시간) 같은 표준 클레임(claim)을 설정하는 옵션이 있습니다. `verify`는 이런 표준 클레임들을 자동으로 검사해줍니다. 만약 커스텀 클레임에 대한 유효성 검사가 필요하다면, `verify` 성공 후 반환된 페이로드 내용을 직접 확인해야 합니다.
*   **비동기 처리 (Async/Await)**: `verify` 함수는 `Promise`를 반환하므로, `await` 키워드를 사용하거나 `.then().catch()`로 처리해야 합니다. "기다릴 줄 아는 자가 페이로드를 얻는다."
*   **토큰 저장 위치**: 클라이언트에서 토큰을 어디에 저장하느냐(localStorage, sessionStorage, HttpOnly 쿠키 등)에 따라 보안 수준이 달라집니다. 일반적으로 HttpOnly 쿠키가 XSS 공격에 좀 더 안전하다고 알려져 있습니다. 서버에서 `verify` 하는 것과는 별개로, 클라이언트 측 보안도 신경 써야 합니다.