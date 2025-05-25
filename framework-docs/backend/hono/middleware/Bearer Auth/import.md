`hono/bearer-auth`에서 `bearerAuth`를 가져옵니다.

```typescript
import { Hono } from 'hono'
import { bearerAuth } from 'hono/bearer-auth'
```

Bearer 인증 미들웨어는 요청 헤더에 담긴 API 토큰을 검증하여 인증 기능을 제공합니다. 엔드포인트에 접근하는 HTTP 클라이언트는 `Authorization` 헤더에 `Bearer {토큰}` 형식으로 값을 추가해야 합니다.

터미널에서 `curl`을 사용하면 이렇게 보일 겁니다:

```bash
curl -H 'Authorization: Bearer honoiscool' http://localhost:8787/auth/page
```

---

**얘 뭐 하는 애냐?**
`bearerAuth` 미들웨어는 API 서버의 문지기 같은 녀석입니다. "암구호!"를 외치며, 요청 헤더의 `Authorization` 필드에 `Bearer [우리가_정한_비밀토큰]` 형식으로 올바른 출입증(토큰)을 제시한 놈만 통과시켜주는 역할을 합니다. 틀린 토큰이나 토큰이 없으면 "넌 못 지나간다!" 하고 401 Unauthorized 에러를 퉤 뱉어버리죠.

**왜 쓰는데?**
1.  **간단한 API 인증**: 복잡한 OAuth 2.0 같은 거 말고, 그냥 서버랑 클라이언트끼리 미리 약속된 고정 토큰 하나로 "우리 편인지 아닌지"만 확인하고 싶을 때 딱입니다. 내부 서비스 간 통신이나 간단한 외부 API 보호용으로 쓰기 좋죠. "야, 우리끼리 아는 비밀번호 대봐!" 하는 느낌.
2.  **무상태(Stateless) 인증**: 서버에 세션 정보를 저장할 필요 없이, 요청마다 토큰만 검증하면 되니까 서버 확장이 용이합니다. 각 서버가 토큰 검증 로직만 알면 되거든요. "어느 창구로 가든 신분증만 보여주면 OK!"
3.  **다양한 클라이언트 지원**: HTTP 헤더에 토큰만 실어 보낼 수 있으면 웹 브라우저, 모바일 앱, 서버 스크립트, `curl` 명령어 등 어떤 클라이언트든 쉽게 연동할 수 있습니다.

**언제 불려 나오냐?**
Hono 앱에서 특정 라우트나 라우트 그룹을 보호하고 싶을 때 미들웨어로 등록해서 씁니다.

```typescript
const app = new Hono()

// 이 토큰을 가진 자만이 /auth 이하 경로에 접근할 수 있다!
const TOKEN = 'honoiscool' // 실제로는 이렇게 코드에 박으면 안 되고 환경변수에서 가져와야 함!

// '/auth'로 시작하는 모든 경로에 bearerAuth 미들웨어 적용
// 올바른 토큰이 없으면 여기서 요청이 차단됨.
app.use('/auth/*', bearerAuth({ token: TOKEN }))

// 이 핸들러는 bearerAuth를 통과한 요청만 받음
app.get('/auth/page', (c) => {
  return c.text('인증 성공! 비밀 페이지에 오신 것을 환영합니다.')
})

app.get('/', (c) => {
  return c.text('여기는 아무나 들어올 수 있는 공개 페이지입니다.')
})
```
위 코드에서 `/auth/*` 경로에 `bearerAuth`를 걸어놨기 때문에, `/auth/page`에 접근하려면 `Authorization: Bearer honoiscool` 헤더가 반드시 필요합니다. `/` 경로는 미들웨어가 없으니 그냥 열리고요.

**쓸 때 꿀팁 및 주의사항:**
*   **토큰은 비밀! 또 비밀!**: 예제 코드처럼 토큰을 소스 코드에 직접 박아두는 건 "우리 집 현관 비밀번호는 1234입니다" 하고 써 붙이는 거랑 똑같습니다. 반드시 환경 변수(`process.env.MY_SECRET_TOKEN`)나 보안 저장소에서 불러와서 사용해야 합니다. "개발자님, 토큰 관리 똑바로 안 하면 해커한테 탈탈 털립니다!"
*   **HTTPS는 기본 예의**: Bearer 토큰은 HTTP 헤더에 실려서 날아가는데, HTTP로 통신하면 중간에 누가 엿듣고 토큰을 홀랑 가로챌 수 있습니다. 반드시 HTTPS (SSL/TLS 암호화 통신) 위에서 사용해야 안전합니다. "비밀 쪽지인데, 봉투도 없이 길바닥에 던져놓으면 어떡해?"
*   **토큰 유출 대비책**: 만약 토큰이 유출된 것 같으면 즉시 해당 토큰을 비활성화하고 새 토큰으로 교체할 수 있는 시스템을 고려해야 합니다. 고정된 단일 토큰 방식은 이게 좀 어렵죠. 그래서 더 복잡한 시스템에서는 JWT(JSON Web Token)처럼 만료 시간이나 특정 권한을 담을 수 있는 토큰을 사용하기도 합니다. `bearerAuth`는 단순 문자열 토큰 검증에 특화되어 있습니다.
*   **토큰 전달 방식은 `Bearer` 스킴 준수**: `Authorization: Bearer <token>` 형식을 정확히 지켜야 합니다. `Bearer` 뒤에 공백 하나, 그리고 토큰 문자열입니다. 괜히 `Token <token>`이나 `MyAuth <token>` 이런 식으로 커스텀하게 보내면 `bearerAuth` 미들웨어가 못 알아먹습니다. "국제 표준은 지키라고 있는 겁니다."
*   **여러 토큰 지원 (prefix 옵션)**: 만약 여러 종류의 Bearer 토큰을 구분해서 써야 한다면, `bearerAuth({ token: TOKEN_A, prefix: 'ServiceA' })`, `bearerAuth({ token: TOKEN_B, prefix: 'ServiceB' })` 처럼 `prefix` 옵션을 활용할 수 있습니다. 이러면 클라이언트는 `Authorization: ServiceA Bearer <token_a>` 또는 `Authorization: ServiceB Bearer <token_b>` 형태로 보내야 합니다. (이 기능은 `hono/bearer-auth`의 최신 버전에 따라 지원 여부가 다를 수 있으니 공식 문서를 확인하는 게 좋습니다. 기본적으로는 단일 `Bearer` 스킴을 가정합니다.)
*   **에러 응답 커스터마이징**: 인증 실패 시 기본적으로 401 Unauthorized 응답과 함께 간단한 메시지가 갑니다. 이걸 좀 더 친절하게 바꾸고 싶다면, `bearerAuth` 미들웨어 다음에 에러 핸들러를 추가해서 `c.res.status === 401`일 때 특정 응답을 보내도록 할 수 있습니다. (Hono의 에러 처리 방식을 활용)