`hono/factory`에서 `createFactory`와 `createMiddleware`를 가져옵니다.

```typescript
import { Hono } from 'hono'
import { createFactory, createMiddleware } from 'hono/factory'
```

팩토리 헬퍼는 미들웨어나 핸들러 같은 Hono 컴포넌트를 만들 때 유용한 함수들을 제공합니다. 가끔 타입스크립트 타입을 제대로 설정하기 까다로울 때가 있는데, 이 헬퍼가 그 과정을 쉽게 만들어줍니다. "타입스크립트, 너 때문에 골치 아팠는데 이제 좀 살겠네!" 하는 느낌이죠.

---

**얘 뭐 하는 애냐?**
`hono/factory`는 Hono 앱을 구성하는 조각들, 특히 미들웨어나 라우트 핸들러를 만들 때 "타입스크립트 형식을 딱 맞춰서 깔끔하게 만들어주는 틀" 같은 역할을 합니다. 복잡한 제네릭 타입이나 인터페이스를 직접 다룰 필요 없이, 이 팩토리 함수들이 알아서 척척 처리해주니 개발자는 로직에만 집중할 수 있죠. "레고 조립 설명서처럼, 규격에 맞는 부품(함수)을 쉽게 만들게 도와준다!"

*   `createFactory()`: Hono 인스턴스(`app`)를 만드는 것부터 시작해서, 그 안에 들어갈 핸들러나 미들웨어까지 일관된 방식으로 생성할 수 있게 돕는 범용 팩토리입니다. (지금은 주로 `createMiddleware`를 만드는 데 쓰이는 듯)
*   `createMiddleware()`: Hono 미들웨어를 만들 때 타입 추론을 기가 막히게 잘 해줍니다. 미들웨어가 받는 `Context (c)` 객체나 `Next` 함수의 타입을 개발자가 일일이 명시하지 않아도, 이 함수를 쓰면 "아, 이 미들웨어는 이런 환경에서 이런 변수들을 다루겠구나!" 하고 똑똑하게 알아챕니다.

**왜 쓰는데?**
1.  **타입스크립트 지옥 탈출**: Hono의 컨텍스트(`c`) 객체는 미들웨어를 거치면서 환경 변수(`Variables`), 바인딩(`Bindings`), 쿠키(`Cookie`), 경로 파라미터(`Path`) 등 다양한 정보가 추가되거나 변경될 수 있습니다. 이걸 타입스크립트로 정확히 표현하려면 제네릭 타입이 덕지덕지 붙어서 코드가 난해해지기 십상인데, 팩토리 헬퍼는 이런 복잡성을 숨겨줍니다. "타입 에러와의 전쟁, 이제 그만!"
2.  **코드 가독성 및 유지보수성 향상**: 타입 정의가 간결해지니 코드가 훨씬 깔끔해지고 이해하기 쉬워집니다. 나중에 코드를 수정하거나 다른 사람이 볼 때도 "이 미들웨어는 뭘 하는 놈이고, 어떤 데이터를 다루는구나" 하고 파악하기 용이해지죠.
3.  **재사용 가능한 컴포넌트 제작 용이**: 잘 정의된 타입과 함께 미들웨어나 핸들러를 만들면, 다른 프로젝트나 앱의 다른 부분에서도 안심하고 가져다 쓸 수 있습니다. "한 번 잘 만들어두면 평생 써먹는 꿀템!"
4.  **Hono 내부 구조 이해도 UP (간접적)**: 팩토리 함수를 쓰면서 Hono가 어떤 식으로 타입을 다루고 컴포넌트를 구성하는지 어깨너머로 배울 수 있습니다. (물론 직접 타입을 다 파헤치는 것만큼은 아니겠지만요!)

**언제 불려 나오냐?**
주로 커스텀 미들웨어를 만들 때 `createMiddleware`가 단골손님으로 등장합니다. 예를 들어, 요청 헤더에 특정 값이 있는지 검사하는 인증 미들웨어나, 요청/응답 시간을 기록하는 로깅 미들웨어를 만들 때 유용합니다.

```typescript
const app = new Hono()
const factory = createFactory() // 팩토리 생성 (선택적, createMiddleware 직접 사용 가능)

// 사용자 정의 타입 (예: 미들웨어에서 추가할 변수)
type Variables = {
  startTime?: number
  userId?: string
}

// createMiddleware를 사용한 로깅 미들웨어
const logger = factory.createMiddleware<Variables>(async (c, next) => {
  // 요청 시작 시간 기록 (미들웨어에서 c.set으로 값 설정)
  c.set('startTime', Date.now())
  console.log(`[${c.req.method}] ${c.req.url} - Request received`)
  await next() // 다음 미들웨어 또는 핸들러 실행
  const ms = Date.now() - (c.var.startTime ?? Date.now()) // c.var로 값 접근
  console.log(`[${c.req.method}] ${c.req.url} - Responded in ${ms}ms`)
})

// createMiddleware를 사용한 인증 미들웨어 (간단 버전)
const authMiddleware = factory.createMiddleware<Variables>(async (c, next) => {
  const authHeader = c.req.header('Authorization')
  if (authHeader && authHeader.startsWith('Bearer ')) {
    const token = authHeader.substring(7)
    // 실제로는 토큰 검증 로직이 들어가야 함
    if (token === 'supersecret') {
      c.set('userId', 'user123') // 인증 성공 시 userId 설정
      await next()
      return
    }
  }
  return c.json({ error: 'Unauthorized' }, 401) // 인증 실패
})

app.use('*', logger) // 모든 요청에 로거 적용
app.use('/admin/*', authMiddleware) // /admin 경로에만 인증 미들웨어 적용

app.get('/', (c) => {
  return c.text('Hello Hono!')
})

app.get('/admin/dashboard', (c) => {
  // authMiddleware에서 설정한 userId를 안전하게 사용 가능
  const userId = c.var.userId
  return c.json({ message: `Welcome to admin dashboard, ${userId}!` })
})
```
위 코드에서 `logger`와 `authMiddleware`는 `createMiddleware` (또는 `factory.createMiddleware`)를 통해 만들어졌습니다. `<Variables>` 제네릭으로 이 미들웨어들이 다룰 컨텍스트 변수의 타입을 지정해줘서, `c.set('startTime', ...)`이나 `c.var.userId` 같은 코드를 타입 안전하게 쓸 수 있습니다.

**쓸 때 꿀팁 및 주의사항:**
*   **제네릭 `<E extends Env = Env>` 활용**: 미들웨어가 특정 환경 변수(Bindings)나 컨텍스트 변수(Variables)에 의존한다면, `createMiddleware<{ Variables: MyVars, Bindings: MyBindings }>(...)`처럼 제네릭을 명시적으로 지정해주는 게 좋습니다. 이러면 `c.env.MY_KV_NAMESPACE`나 `c.var.myCustomValue` 같은 접근이 타입 안전해집니다. "타입스크립트, 너는 다 계획이 있구나!"
*   **`c.set()`과 `c.var`는 짝꿍**: 미들웨어에서 `c.set('key', value)`로 값을 설정하면, 그 이후의 미들웨어나 최종 핸들러에서는 `c.var.key` (또는 `c.get('key')`)로 해당 값을 꺼내 쓸 수 있습니다. 팩토리 헬퍼는 이 과정의 타입 추론을 도와줍니다.
*   **`Next` 함수의 중요성**: 미들웨어 안에서 `await next()`를 호출해야 다음 미들웨어나 라우트 핸들러로 제어가 넘어갑니다. 이걸 빼먹으면 요청 처리가 거기서 멈춰버리니 주의! "바통 터치 안 하면 계주 끝까지 못 달린다."
*   **에러 처리도 잊지 말자**: 미들웨어에서 에러가 발생하면 Hono의 에러 핸들러가 처리하거나, 직접 응답을 반환해서 요청을 끝낼 수 있습니다.
*   **너무 많은 커스텀 타입은 오히려 독**: 팩토리 헬퍼가 타입을 쉽게 만들어주긴 하지만, 너무 자잘하게 많은 커스텀 타입을 정의하면 오히려 전체적인 코드 구조 파악이 어려워질 수도 있습니다. 적절한 수준에서 타입을 그룹화하고 재사용하는 게 중요합니다. "과유불급, 타입도 마찬가지."