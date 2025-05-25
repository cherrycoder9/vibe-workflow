**사용법**

**주의사항**

네 토큰은 반드시 `/[A-Za-z0-9._~+/-]+=*/` 정규식 패턴에 맞아야 해. 안 그러면 400 에러가 반환될 거야. 특히 이 정규식은 URL-safe Base64 방식과 표준 Base64 방식으로 인코딩된 JWT(JSON Web Token)를 둘 다 처리할 수 있도록 만들어졌어. 이 미들웨어는 베어러 토큰이 꼭 JWT여야 한다고 강제하진 않지만, 위 정규식에는 맞아야 한다는 점! (쉽게 말해, "아무거나 막 쓰면 안 되고, 정해진 글자들로만 만들어야 통과시켜준다!" 이거지.)

```typescript
const app = new Hono()

// 이게 바로 그 비밀의 토큰!
const token = 'honoiscool'

// '/api/'로 시작하는 모든 요청에 대해 토큰 검사를 시행한다!
app.use('/api/*', bearerAuth({ token }))

app.get('/api/page', (c) => {
  return c.json({ message: '인증되셨습니다, 손님.' })
})
```

**특정 라우트 + 특정 HTTP 메서드에만 제한 걸기:**

```typescript
const app = new Hono()

const token = 'honoiscool' // 여전히 이 토큰을 쓴다.

// GET /api/page 요청은 토큰 검사 없이 누구나 접근 가능!
app.get('/api/page', (c) => {
  return c.json({ message: '게시글 읽기 완료.' })
})

// POST /api/page 요청은? 어림없지! bearerAuth 통과해야만 실행된다.
app.post('/api/page', bearerAuth({ token }), (c) => {
  return c.json({ message: '게시글 작성 완료!' }, 201)
})
```

**여러 토큰 사용하기 (예: 읽기용 토큰은 여러 개, 특정 작업(생성/수정/삭제)은 권한 있는 토큰으로만 제한):**

```typescript
const app = new Hono()

const readToken = 'read' // 읽기 전용 토큰
const privilegedToken = 'read+write' // 특별 권한 토큰 (읽고 쓰기 다 됨)
const privilegedMethods = ['POST', 'PUT', 'PATCH', 'DELETE'] // 특별 권한이 필요한 HTTP 메서드 목록

// GET /api/page/* 요청은 'read' 토큰 또는 'read+write' 토큰 둘 중 하나만 있으면 통과!
app.on('GET', '/api/page/*', async (c, next) => {
  const bearer = bearerAuth({ token: [readToken, privilegedToken] }) // 유효한 토큰 목록을 배열로 전달
  return bearer(c, next)
})

// POST, PUT, PATCH, DELETE /api/page/* 요청은 반드시 'read+write' 토큰이 있어야 통과!
app.on(privilegedMethods, '/api/page/*', async (c, next) => {
  const bearer = bearerAuth({ token: privilegedToken }) // 단일 특별 권한 토큰만 허용
  return bearer(c, next)
})

// 여기에 GET, POST 등의 실제 핸들러 함수들을 정의하면 된다.
// 예: app.get('/api/page/:id', (c) => { /* ... */ })
//     app.post('/api/page', (c) => { /* ... */ })
```

**토큰 값을 직접 검증하고 싶다면 `verifyToken` 옵션을 사용해. `true`를 반환하면 통과야.**

```typescript
const app = new Hono()

app.use(
  '/auth-verify-token/*', // 이 경로의 요청들은 아래 로직으로 토큰을 검증한다.
  bearerAuth({
    // verifyToken 함수: 여기서 토큰 유효성 검사를 직접 커스텀할 수 있다.
    // 비동기 함수(async)도 가능!
    verifyToken: async (token, c) => {
      // 예를 들어, 토큰이 'dynamic-token' 문자열과 일치하는지 확인
      return token === 'dynamic-token'
    },
  })
)
```

---

**얘 뭐 하는 애냐?**
`bearerAuth`는 Hono 앱으로 들어오는 요청의 `Authorization` 헤더에서 "Bearer " 다음에 오는 토큰 값을 확인해서, 미리 설정해둔 비밀 토큰이랑 일치하는지 검사하는 문지기 역할이야. "암구호 대! (토큰 내놔!)" 해서 맞으면 통과, 틀리면 "넌 못 지나간다!" 하고 401 (Unauthorized) 에러를 뱉어내지. API 보안의 기본 중의 기본이야.

**왜 쓰는데?**
1.  **API 접근 제어**: 아무나 중요한 API에 접근해서 데이터를 마구 가져가거나 망가뜨리는 걸 막기 위해 써. 허가된 사용자(또는 서비스)만 API를 이용하도록 하는 거지. "회원 전용 공간입니다, 비회원은 나가주세요!"
2.  **간단한 인증 구현**: 복잡한 로그인 시스템 없이, 미리 발급된 정적 토큰만으로 특정 API를 보호하고 싶을 때 간편하게 쓸 수 있어. 내부 서비스 간 통신이나 간단한 웹훅 인증 등에 유용해.
3.  **유연한 권한 관리**: 위 예시처럼 여러 토큰을 사용하거나, `verifyToken` 옵션으로 직접 검증 로직을 짜면, 요청 경로/메서드별로, 또는 토큰의 종류별로 다른 접근 권한을 부여하는 것도 가능해. "VIP 손님은 프리패스, 일반 손님은 여기까지!"

**언제 불려 나오냐?**
Hono 미들웨어로 등록해서 특정 경로 또는 모든 경로의 요청이 실제 라우트 핸들러에 도달하기 "전에" 호출돼. `app.use('/admin/*', bearerAuth({ token: 'secret' }))`처럼 쓰면 `/admin/`으로 시작하는 모든 요청은 먼저 `bearerAuth`의 검문을 받아야 해.

**쓸 때 꿀팁 및 주의사항:**
*   **토큰 형식은 정규식 준수**: 앞서 말했듯이 토큰 문자열은 `/[A-Za-z0-9._~+/-]+=*/` 패턴을 따라야 해. 안 그러면 "너 토큰 이상해!" 하고 400 에러 맞는다. Base64 인코딩된 문자열이나 JWT 토큰 형식이 보통 이 패턴에 잘 맞아.
*   **HTTPS는 필수**: Bearer 토큰은 HTTP 헤더에 실려서 날아가는데, HTTP로 통신하면 중간에 누가 훔쳐볼 수 있어. "비밀 암호를 길바닥에 써놓고 다니는 격!" 반드시 HTTPS를 사용해서 통신을 암호화해야 토큰 탈취 위험을 줄일 수 있어.
*   **토큰 보관은 안전하게**: 서버 사이드에 저장하는 `token` 값이나 `verifyToken` 로직에 사용되는 비밀키 등은 코드에 직접 박아두기보다는 환경 변수나 비밀 관리 시스템(Vault, AWS Secrets Manager 등)을 통해 안전하게 관리해야 해. "금고 열쇠를 책상 위에 올려두진 않잖아?"
*   **`verifyToken`으로 커스텀 로직**: 단순 문자열 비교 이상의 복잡한 토큰 검증(예: DB 조회, 외부 인증 서비스 연동, JWT 서명 검증 및 만료 시간 체크)이 필요하면 `verifyToken` 함수를 적극 활용해. 이 함수 안에서 `c` (컨텍스트 객체)도 쓸 수 있어서 요청 정보를 추가로 활용하는 것도 가능해.
*   **에러 응답 커스터마이징**: 기본적으로 인증 실패 시 401 에러와 함께 `{"message":"Unauthorized"}` JSON 응답을 보내는데, 필요하면 `onError` 옵션을 사용해서 에러 응답 형식을 바꿀 수도 있어. (Hono 공식 문서에는 `onError`가 명시적으로 안 보일 수 있는데, Hono의 일반적인 오류 처리 방식을 따르거나, `bearerAuth` 미들웨어 다음에 별도의 오류 처리 미들웨어를 두는 방식으로 해결 가능)
*   **토큰 유출 시 대처 방안 마련**: 아무리 잘 관리해도 토큰이 유출될 가능성은 항상 존재해. 유출 시 해당 토큰을 빠르게 무효화하고 재발급할 수 있는 절차를 미리 생각해두는 게 좋아. "소 잃고 외양간 고치지 말자."