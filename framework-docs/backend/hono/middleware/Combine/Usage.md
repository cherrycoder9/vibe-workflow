`hono/combine`에서 `some`, `every`, `except`를 가져옵니다.

```typescript
import { Hono } from 'hono'
import { some, every, except } from 'hono/combine'
```

미들웨어 결합(Combine Middleware)은 여러 미들웨어 함수를 하나의 미들웨어로 합쳐줍니다. 세 가지 함수를 제공합니다:

*   `some` - 주어진 미들웨어 중 **하나만** 실행합니다. (조건에 맞는 놈 하나만 골라 실행!)
*   `every` - 주어진 미들웨어를 **모두** 실행합니다. (줄줄이 비엔나처럼 다 실행!)
*   `except` - 특정 조건이 충족되지 **않을 경우에만** 주어진 미들웨어를 모두 실행합니다. (특정 상황 빼고 다 실행!)

---

**얘네 뭐 하는 애들이냐?**
`hono/combine`에서 제공하는 `some`, `every`, `except`는 여러 개의 미들웨어를 마치 레고 블록처럼 조립해서 하나의 새로운 미들웨어로 만들어주는 녀석들이야. 복잡한 미들웨어 로직을 깔끔하게 정리하거나, 특정 조건에 따라 다른 미들웨어 세트를 실행하고 싶을 때 유용하지. "미들웨어 부대, 상황 따라 맞춤 편성!"

*   **`some(...handlers)`**: 인자로 전달된 미들웨어 핸들러들을 순서대로 실행하다가, 그중 **하나라도 응답(Response 객체)을 반환하면 거기서 멈추고** 그 응답을 사용해. 마치 "이것저것 시도해보다가 뭐 하나 성공하면 그걸로 끝!" 하는 느낌. 주로 여러 인증 방식 중 하나만 통과하면 되는 경우 등에 쓸 수 있어.
*   **`every(...handlers)`**: 인자로 전달된 미들웨어 핸들러들을 **무조건 순서대로 전부 다 실행**해. 중간에 누가 응답을 반환하든 말든 신경 안 쓰고 끝까지 간다. (주의: `every` 자체는 마지막 핸들러의 결과를 따르거나, 중간에 `c.res`가 명시적으로 세팅되지 않는 한, 직접적인 응답 반환보다는 요청 객체(`c.req`)를 변경하거나 컨텍스트(`c`)에 값을 추가하는 등의 연속적인 작업을 할 때 더 적합해. 만약 중간 핸들러가 응답을 반환하면, 그 이후 핸들러는 실행되지 않을 수 있어. Hono의 미들웨어 체인 기본 동작을 따르기 때문이지. "모든 관문을 통과해야 비로소 다음 단계로!")
*   **`except(conditionFn, ...handlers)`**: 첫 번째 인자로 조건 함수(`conditionFn`)를 받고, 그 뒤에 미들웨어 핸들러들을 받습니다. 이 `conditionFn`이 `false`를 반환할 때만 (즉, 조건이 충족되지 않을 때만) 뒤따라오는 핸들러들이 실행돼. "이 경우만 빼고, 나머지는 다 이 미들웨어들 거쳐가세요!" 하는 거지. 특정 경로, 특정 사용자 유형 등을 제외하고 공통 미들웨어를 적용할 때 좋아.

**왜 쓰는데?**
1.  **미들웨어 로직 재사용 및 모듈화**: 비슷한 미들웨어 묶음을 여러 라우트에서 사용해야 할 때, `every` 같은 걸로 하나로 합쳐두면 코드 중복 줄이고 관리하기 편해. "자주 쓰는 공구 세트 미리 만들어두기!"
2.  **조건부 미들웨어 실행**: `some`이나 `except`를 쓰면 특정 조건에 따라 다른 미들웨어가 실행되도록 유연하게 제어할 수 있어. 예를 들어, 요청 헤더에 특정 토큰이 있으면 A 미들웨어를, 없으면 B 미들웨어를 시도하게 하는 식이지. "상황별 맞춤 대응 메뉴판!"
3.  **가독성 향상**: 여러 `app.use()`나 라우트 핸들러에 미들웨어를 줄줄이 나열하는 것보다, 의미 단위로 묶어서 `combine` 함수를 사용하면 코드가 훨씬 깔끔해지고 이해하기 쉬워져. "복잡한 레시피를 단계별로 정리하는 느낌!"

**언제 불려 나오냐?**
Hono 앱의 미들웨어 체인 중간이나 라우트 핸들러의 일부로 사용돼. `app.use()`에 통째로 넣거나, 특정 라우트의 핸들러 자리에 `some(...)`이나 `every(...)` 등을 직접 넣어서 쓰는 거지.

```typescript
const app = new Hono()

// 가상의 인증 미들웨어들
const checkJwt = async (c, next) => {
  const authHeader = c.req.header('Authorization')
  if (authHeader && authHeader.startsWith('Bearer jwt_token_')) {
    c.set('user', { id: 'jwtUser', authType: 'jwt' }) // 인증 성공! 사용자 정보 저장
    console.log('JWT 인증 시도 및 성공 (또는 여기서 응답 반환 가능)')
    // 만약 여기서 c.json(...) 등으로 응답하면 some은 여기서 멈춤
    // await next() // some에서는 다음 핸들러로 넘어가지 않도록 주의 (하나만 성공하면 되니까)
    return // some의 경우, 응답을 직접 반환하거나 아무것도 반환 안 하면 다음 some 내 핸들러 시도
           // 하지만 보통은 인증 성공 시 c.res를 설정하거나, 아니면 그냥 통과(next 호출 안 함)
  }
  // JWT 인증 실패 시, 다음 some 내 핸들러 시도를 위해선 아무것도 안 하거나 에러를 던지지 않아야 함.
  // 또는 명시적으로 다음 핸들러를 호출하지 않고 끝내서 some이 다음 것을 시도하게 할 수 있음.
  // 여기서는 간단히 하기 위해 다음 핸들러로 넘어가지 않는다고 가정.
  // 실제로는 await next()를 호출해야 다음 핸들러로 넘어감. some의 동작 방식에 따라 조정 필요.
  // Hono의 some은 첫 번째로 '성공적인' 응답을 반환하는 핸들러를 찾음.
  // '성공적인' 응답이란, undefined가 아닌 값을 리턴하는 것을 의미.
  // 따라서, 인증 실패 시에는 undefined를 리턴하거나 다음 핸들러로 제어를 넘겨야 함.
  // 여기서는 설명을 위해 단순화. 실제 구현 시에는 Hono 문서를 참고하여 정확한 흐름 제어 필요.
  console.log('JWT 인증 시도: 실패 또는 해당 없음')
  await next() // some 내부에서는, 다음 핸들러를 시도하려면 next() 호출
}

const checkApiKey = async (c, next) => {
  const apiKey = c.req.header('X-API-KEY')
  if (apiKey === 'secret_api_key') {
    c.set('user', { id: 'apiKeyUser', authType: 'apiKey' })
    console.log('API Key 인증 시도 및 성공')
    // return c.json({ message: 'API Key 인증 성공!', user: c.get('user') }) // some에서 여기서 응답하면 끝
    await next() // API Key 인증 성공 후 다음 미들웨어/핸들러로
    return // some의 경우, 여기서 응답하거나 next()를 호출하여 체인을 이어갈 수 있음.
           // 여기서는 next()를 호출했으므로, 만약 이게 some의 마지막이라면 다음 미들웨어로 넘어감.
           // 하지만 some은 '하나의' 성공적인 응답을 찾으므로, 여기서 응답을 반환하는 것이 일반적.
           // 설명을 위해 next()를 호출하는 형태로 둠.
  }
  console.log('API Key 인증 시도: 실패 또는 해당 없음')
  await next()
}

// 로깅 미들웨어
const logger = async (c, next) => {
  console.log(`[${c.req.method}] ${c.req.url} - 요청 시작`)
  await next()
  console.log(`[${c.req.method}] ${c.req.url} - 요청 끝: ${c.res.status}`)
}

// 특정 IP 대역인지 확인하는 조건 함수 (예시)
const isInternalNetwork = (c) => {
  const clientIp = c.req.header('X-Forwarded-For') || 'unknown'
  return clientIp.startsWith('192.168.') || clientIp.startsWith('10.')
}

// 1. some: JWT 또는 API Key 둘 중 하나만 통과하면 됨
app.use('/api/data', some(checkJwt, checkApiKey, async (c) => {
  // 둘 다 실패하면 이 핸들러가 실행됨 (일종의 최종 실패 처리)
  console.log('모든 인증 방법 실패')
  return c.json({ error: '인증 실패' }, 401)
}))

// /api/data 경로는 위 some 미들웨어를 통과해야만 아래 핸들러 실행 가능
app.get('/api/data', (c) => {
  const user = c.get('user')
  if (!user) {
    // some에서 이미 401을 반환했겠지만, 방어적으로 한 번 더 체크
    return c.json({ error: '인증 정보 없음' }, 401)
  }
  return c.json({ message: `데이터 접근 성공! 인증 방식: ${user.authType}`, userId: user.id })
})


// 2. every: 로깅도 하고, 요청에 특정 헤더도 추가하고 싶을 때
const addCustomHeader = async (c, next) => {
  c.req.raw.headers.append('X-Powered-By-Hono-Combine', 'true') // 실제 요청 객체 헤더 수정
  console.log('커스텀 헤더 추가됨')
  await next()
}
app.use('/admin/*', every(logger, addCustomHeader))

app.get('/admin/info', (c) => {
  // logger와 addCustomHeader가 모두 실행된 후 이 핸들러 실행
  const customHeader = c.req.header('X-Powered-By-Hono-Combine')
  return c.text(`관리자 페이지입니다. 커스텀 헤더: ${customHeader}`)
})


// 3. except: 내부 네트워크가 아닐 때만 접근 제한 미들웨어를 실행
const restrictExternalAccess = async (c, next) => {
  console.log('외부 접근 제한 미들웨어 실행됨 (내부 네트워크가 아님)')
  // 여기서 실제 접근 제한 로직 구현 (예: 특정 사용자만 허용 등)
  // 여기서는 간단히 로그만 남기고 통과
  await next()
}
app.use('/sensitive-data', except(isInternalNetwork, logger, restrictExternalAccess)) // 내부망이면 logger, restrictExternalAccess 건너뜀

app.get('/sensitive-data', (c) => {
  if (isInternalNetwork(c)) {
    return c.text('내부망 사용자님, 민감 정보에 접근하셨습니다. (제한 로직 건너뜀)')
  }
  return c.text('민감 정보에 접근하셨습니다. (외부 접근으로 간주되어 제한 로직 거침)')
})
```

**쓸 때 꿀팁 및 주의사항:**
*   **`some`의 실행 흐름**: `some`은 핸들러 중 하나가 `undefined`가 아닌 값(주로 `Response` 객체)을 반환하면 그걸로 끝내고 다음 핸들러는 실행 안 해. 만약 모든 핸들러가 `await next()`를 호출하거나 `undefined`를 반환하면, `some`은 마치 아무 일도 없었다는 듯이 다음 미들웨어로 제어를 넘겨. "첫 번째 성공이 중요!"
*   **`every`와 응답 반환**: `every` 안의 미들웨어가 중간에 `c.json(...)` 같은 걸로 응답을 반환해버리면, 그 뒤에 있는 `every` 내 다른 미들웨어나 최종 라우트 핸들러는 실행 안 될 수 있어. Hono의 기본 미들웨어 체인 동작을 따르기 때문이지. `every`는 주로 요청/응답을 직접 수정하거나 로깅, 컨텍스트 값 설정 등 "모두 거쳐가야 하는" 작업에 적합해.
*   **`except`의 조건 함수**: `except(conditionFn, ...handlers)`에서 `conditionFn(c)`는 컨텍스트 객체 `c`를 인자로 받고, `true` 또는 `false`를 반환해야 해. `true`면 "예외 상황! 뒤에 핸들러들 실행하지 마!", `false`면 "예외 아니네. 뒤에 핸들러들 다 실행해!" 라는 뜻.
*   **`await next()`의 중요성**: `combine` 함수 내부에 사용되는 각 미들웨어 핸들러 안에서 `await next()`를 호출해야 다음 미들웨어나 최종 라우트 핸들러로 제어가 넘어가. 이걸 빼먹으면 미들웨어 체인이 거기서 멈춰버릴 수 있으니 주의해야 해. (단, `some`의 경우 의도적으로 `next()`를 호출하지 않고 응답을 반환하여 체인을 종료시킬 수 있음)
*   **디버깅**: 미들웨어가 여러 개 겹쳐있으면 문제 생겼을 때 어디가 잘못된 건지 찾기 어려울 수 있어. `console.log`를 적절히 찍어가며 흐름을 파악하는 게 중요해. "복잡할수록 정신 바짝 차려야 한다."
*   **순서가 생명**: `every`나 `some`에 전달하는 미들웨어의 순서가 중요해. 실행 순서에 따라 결과가 달라질 수 있으니 신중하게 배치해야 해. "줄 잘 서야 콩고물이라도 떨어진다."