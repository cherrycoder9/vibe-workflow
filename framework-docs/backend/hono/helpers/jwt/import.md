이봐, JWT 인증 헬퍼는 JSON 웹 토큰(JWT)을 인코딩, 디코딩, 서명, 검증하는 함수들을 제공해. JWT는 웹 애플리케이션에서 인증이나 권한 부여 목적으로 흔하게 쓰이지. 이 헬퍼는 다양한 암호화 알고리즘을 지원하는 강력한 JWT 기능을 제공한다구. "출입증 발급부터 위조 검사까지 한 큐에 해결!"

**가져오기 (Import)**
이 헬퍼를 쓰려면 아래처럼 가져오면 돼:

```typescript
import { decode, sign, verify } from 'hono/jwt'
```

**참고 (INFO)**
JWT 미들웨어도 `hono/jwt`에서 `jwt` 함수를 가져와서 사용해. (그러니까 얘네는 사실상 한솥밥 먹는 식구들이라는 거지.)

---

**얘 뭐 하는 애냐?**
`hono/jwt`는 웹 서비스에서 "얘가 누군지", "뭘 할 수 있는지"를 안전하게 확인하고 증명하는 데 쓰이는 JWT(JSON Web Token, 일명 '제이더블유티' 또는 '조트')를 만들고(sign), 까보고(decode), 진짜인지 가짜인지 검증(verify)하는 도구 세트야. 온라인 세상의 신분증 겸 출입증 관리소라고 보면 돼.

*   `sign`: 사용자 정보(페이로드)랑 비밀키를 섞어서 암호화된 JWT 문자열을 뚝딱 만들어내. "신분증 발급 완료!"
*   `decode`: JWT 문자열을 받아서 그 안에 어떤 정보가 들어있는지 (서명 검증 없이) 일단 까서 보여줘. "일단 신분증 내용물만 좀 봅시다." (주의: 이걸로 진짜 신분증인지 확인하는 건 아님!)
*   `verify`: JWT 문자열이랑 비밀키를 대조해서 "이거 위조된 거 아니야? 유효기간은 안 지났고?" 하고 꼼꼼하게 검증해. 이게 통과돼야 진짜 믿을 수 있는 정보지. "이 신분증, 진짜 맞습니다, 손님!"

**왜 쓰는데?**
1.  **상태 없는(Stateless) 인증**: 서버가 사용자 세션 정보를 일일이 기억하고 있을 필요가 없어. 클라이언트가 요청할 때마다 JWT를 보내주면, 서버는 그 JWT만 검증해서 사용자를 인증하거든. 서버 확장이 훨씬 쉬워지지. "서버는 기억상실증 걸려도 괜찮아, JWT만 있으면 돼!"
2.  **마이크로서비스 환경에 적합**: 여러 서비스들이 각자 돌아가는 환경에서, JWT 하나로 여러 서비스에 대한 인증/인가 정보를 전달할 수 있어. "만능 프리패스 하나로 여기저기 다 뚫고 다니기!"
3.  **정보 전달의 유연성**: JWT 안(페이로드)에 사용자 ID뿐만 아니라 권한 정보(예: 'admin', 'editor'), 만료 시간 같은 추가 정보를 담아서 보낼 수 있어. 서버가 DB를 뒤지지 않고도 필요한 정보를 얻을 수 있지.
4.  **보안**: 비밀키로 서명되어 있어서 중간에 누가 내용을 바꿔치기해도 `verify` 과정에서 바로 들통나. "봉인된 편지처럼 안전하게!" (물론 비밀키 관리는 철저히 해야 함)

**언제 불려 나오냐?**
*   `sign`: 사용자가 로그인 성공했을 때, 서버에서 해당 사용자의 정보를 담은 JWT를 생성해서 클라이언트에게 전달해 줄 때 쓰여.
*   `decode`: JWT 내용을 간단히 확인하고 싶을 때 (예: 디버깅 목적). 하지만 이걸로 인증 처리를 하면 절대 안 돼!
*   `verify`: 클라이언트가 API 요청 시 `Authorization: Bearer <JWT>` 헤더에 JWT를 담아 보내면, 서버 쪽 미들웨어나 라우트 핸들러에서 이 JWT가 유효한지 검증할 때 쓰여. 이게 통과되어야 "아, 이 사용자는 인증된 사용자구나" 하고 다음 작업을 진행하지.

```typescript
// 대충 이런 느낌으로 쓴다~ (실제 사용 시엔 에러 처리, 비동기 처리 등 더 필요)
const app = new Hono()
const secret = '나만아는비밀키' // 실제로는 이렇게 코드에 박으면 안 됨! 환경변수로!

// 로그인 성공 시 JWT 발급 (sign)
app.post('/login', async (c) => {
  const { username, password } = await c.req.json()
  // 여기서 실제 DB에서 사용자 검증 로직이 들어가야 함
  if (username === 'hono_user' && password === 'password123') {
    const payload = {
      sub: username, // 보통 사용자 식별자
      role: 'user',
      exp: Math.floor(Date.now() / 1000) + (60 * 60) // 1시간 뒤 만료
    }
    const token = await sign(payload, secret)
    return c.json({ token })
  }
  return c.text('로그인 실패!', 401)
})

// 보호된 API (verify)
app.get('/protected', async (c) => {
  const authHeader = c.req.header('Authorization')
  if (!authHeader || !authHeader.startsWith('Bearer ')) {
    return c.text('인증 토큰이 없거나 형식이 틀렸습니다.', 401)
  }
  const token = authHeader.split(' ')[1]

  try {
    const decodedPayload = await verify(token, secret) // 여기서 검증!
    // decodedPayload 안에 sign 할 때 넣었던 정보가 들어있음
    return c.json({ message: `환영합니다, ${decodedPayload.sub}님! 당신의 역할은 ${decodedPayload.role}입니다.` })
  } catch (error) {
    // 토큰이 만료됐거나, 서명이 안 맞거나, 그냥 가짜 토큰일 경우 에러 발생
    return c.text('유효하지 않은 토큰입니다.', 401)
  }
})

// JWT 내용 까보기 (decode) - 서명 검증 안 함!
app.get('/decode-token', async (c) => {
  const token = c.req.query('token') // 쿼리 파라미터로 토큰 받기
  if (!token) return c.text('토큰을 주세요.', 400)
  try {
    const decoded = decode(token)
    return c.json(decoded)
  } catch (error) {
    return c.text('토큰 해석 실패', 400)
  }
})
```

**쓸 때 꿀팁 및 주의사항:**
*   **비밀키(Secret)는 생명**: `sign`하고 `verify`할 때 쓰는 비밀키는 절대 코드에 하드코딩하거나 깃허브 같은 데 올리면 안 돼. "비밀번호를 대문짝만하게 써 붙여놓는 격!" 환경 변수로 빼서 안전하게 관리해야 해. 유출되면 그냥 다 뚫리는 거야.
*   **알고리즘 선택은 신중하게**: `sign` 함수는 `alg` 옵션으로 다양한 암호화 알고리즘(HS256, RS256 등)을 지원해. HS256은 대칭키(하나의 비밀키 사용), RS256은 비대칭키(공개키/개인키 쌍 사용) 방식이야. 서비스 규모나 보안 요구사항에 맞춰 골라야 해. 잘 모르겠으면 일단 HS256이 무난하지만, 보안이 더 중요하다면 RS256 같은 비대칭키 방식도 고려해봐.
*   **페이로드(Payload)에는 민감 정보 넣지 마**: JWT 페이로드는 Base64로 인코딩된 거라, `decode` 함수나 온라인 디코더로 누구나 까볼 수 있어. 비밀번호, 주민번호 같은 민감 정보는 절대 넣으면 안 돼. "길바닥에 개인정보 뿌리고 다니는 꼴." 사용자 ID, 역할, 만료 시간 정도만 넣는 게 일반적이야.
*   **만료 시간(exp)은 필수**: JWT에는 반드시 만료 시간을 설정해야 해 (`exp` 클레임). 만료 시간 없는 토큰은 "무기한 프리패스"나 다름없어서, 한번 탈취되면 영원히 악용될 수 있어. 적절히 짧은 만료 시간을 설정하고, 필요하면 리프레시 토큰(Refresh Token) 같은 걸로 보완하는 게 좋아.
*   **HTTPS는 기본 중의 기본**: JWT를 HTTP 헤더나 바디로 주고받을 때는 반드시 HTTPS를 써야 해. 안 그러면 중간에 토큰이 홀랑 가로채여서 "내 신분증 누가 훔쳐갔어!" 하는 상황이 발생해.
*   **`decode`는 검증이 아니다**: `decode(token)`은 그냥 토큰 내용물을 보여줄 뿐, 그 토큰이 진짜인지, 위변조되지 않았는지, 만료되지 않았는지는 전혀 신경 안 써. 실제 사용자 인증/인가 로직에서는 반드시 `verify(token, secret)`를 써야 해. "겉모습만 보고 사람 판단하면 큰코다친다."
*   **토큰 저장 위치**: 클라이언트에서 JWT를 어디에 저장할지도 고민해야 해. 웹 브라우저라면 보통 localStorage나 sessionStorage, 혹은 HttpOnly 쿠키에 저장하는데, 각각 장단점과 보안 고려사항이 있어. XSS, CSRF 공격에 대한 대비도 필요하고. "보관도 잘해야 도둑 안 맞는다."