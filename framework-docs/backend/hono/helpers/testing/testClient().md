`testClient()`
`testClient()` 함수는 Hono 인스턴스를 첫 번째 인자로 받아서, Hono 애플리케이션의 라우트에 따라 타입이 지정된 객체를 반환해. 마치 Hono 클라이언트처럼 말이지. 이걸 쓰면 테스트 코드 안에서 타입 안전하게, 편집기 자동완성까지 받으면서 정의된 라우트를 호출할 수 있어. "야, 내가 만든 API 잘 돌아가는지 똑똑하게 테스트 좀 해보자!" 할 때 쓰는 녀석이야.

**타입 추론에 대한 중요 참고 사항:**

`testClient`가 라우트 타입을 제대로 추론하고 자동완성을 제공하게 하려면, Hono 인스턴스에 직접 메서드를 체인 형태로 연결해서 라우트를 정의해야 해.

타입 추론은 `.get()`, `.post()` 같은 메서드 호출이 줄줄이 이어지는 흐름에 의존하거든. 만약 Hono 인스턴스를 만들고 나서 라우트를 따로 정의하면("Hello World" 예제에서 흔히 보이는 `const app = new Hono(); app.get(...)` 같은 패턴), `testClient`는 특정 라우트에 필요한 타입 정보를 얻지 못해서 타입 안전한 클라이언트 기능을 못 써먹게 돼. "체인 안 걸면? 타입? 그게 뭔데? 먹는 건가?" 상태가 되는 거지.

**예시:**

아래 예시는 `.get()` 메서드가 `new Hono()` 호출에 바로 체인으로 연결되어 있어서 잘 돌아가.

```typescript
// index.ts - 우리 앱 로직
const app = new Hono().get('/search', (c) => {
  const query = c.req.query('q') // 'q'라는 쿼리 파라미터 값을 가져오고
  return c.json({ query: query, results: ['result1', 'result2'] }) // JSON으로 응답!
})

export default app // 앱을 바깥으로 내보내서 테스트 파일에서 쓸 수 있게

// index.test.ts - 우리 앱 테스트하는 파일
import { Hono } from 'hono' // Hono는 기본
import { testClient } from 'hono/testing' // 테스트용 클라이언트 소환!
import { describe, it, expect } from 'vitest' // Vitest 같은 테스트 러너에서 쓰는 함수들
import app from './app' // 위에서 만든 우리 앱 가져오기

describe('Search Endpoint 테스트 그룹', () => {
  // 앱 인스턴스로 테스트 클라이언트 생성
  const client = testClient(app)

  it('검색 결과를 반환해야 함', async () => {
    // 타입이 적용된 클라이언트를 사용해서 엔드포인트 호출
    // 라우트에 정의된 쿼리 파라미터에 대한 타입 안전성,
    // 그리고 .$get()으로 직접 접근하는 걸 주목해!
    const res = await client.search.$get({ // client.라우트경로.$HTTP메서드() 형태로 호출
      query: { q: 'hono' }, // 쿼리 파라미터도 타입 체크 가능 (라우트 정의에 따라)
    })

    // 결과 검증
    expect(res.status).toBe(200) // 상태 코드는 200이어야 하고
    expect(await res.json()).toEqual({ // JSON 응답 내용은 예상과 같아야지
      query: 'hono',
      results: ['result1', 'result2'],
    })
  })
})
```

테스트에 헤더를 포함하려면, 호출 시 두 번째 파라미터로 넘겨주면 돼.

```typescript
// index.test.ts
import { Hono } from 'hono'
import { testClient } from 'hono/testing'
import { describe, it, expect } from 'vitest'
import app from './app'

describe('Search Endpoint 테스트 그룹', () => {
  const client = testClient(app)

  it('검색 결과를 반환해야 함 (헤더 포함)', async () => {
    // 헤더에 토큰 넣고, Content-Type도 설정
    const token = 'this-is-a-very-clean-token' // 아주 깨끗한 토큰이지! (농담)
    const res = await client.search.$get(
      { // 첫 번째 인자는 요청 바디나 쿼리 파라미터
        query: { q: 'hono' },
      },
      { // 두 번째 인자로 헤더 객체를 전달
        headers: {
          Authorization: `Bearer ${token}`,
          'Content-Type': `application/json`,
        },
      }
    )

    expect(res.status).toBe(200)
    expect(await res.json()).toEqual({
      query: 'hono',
      results: ['result1', 'result2'],
    })
  })
})
```

---

**얘 뭐 하는 애냐?**
`testClient()`는 Hono로 만든 웹 애플리케이션의 API 엔드포인트들을 마치 실제 사용자가 요청 보내듯이 테스트해볼 수 있는 "가상 클라이언트"야. 코드 안에서 내 앱의 `/users`나 `/products` 같은 주소로 GET, POST 요청을 날려보고 응답이 제대로 오는지, 데이터는 정확한지 등을 검증할 수 있게 해줘. "내 가게 음식 맛 어떤지 내가 직접 먹어보는 셀프 시식 코너" 같은 거지.

**왜 쓰는데?**
1.  **타입 안전성 확보**: Hono 앱 정의할 때 라우트 경로, 요청/응답 데이터 타입을 잘 명시해뒀다면, `testClient` 쓸 때도 그 타입 정보가 그대로 적용돼. 예를 들어 `/search` 엔드포인트가 `q`라는 문자열 쿼리 파라미터를 받는다고 정의했으면, 테스트 코드에서 `client.search.$get({ query: { q: 123 } })` 이렇게 숫자 넣으려고 하면 타입 에러가 뙇! "야, 거기 문자열 넣으랬지 숫자 넣으랬냐!" 하고 컴파일러가 미리 알려주니 버그 잡기 수월해.
2.  **자동완성 개꿀**: 편집기(VSCode 등)에서 `client.`까지만 쳐도 내 앱에 정의된 라우트 목록(`search`, `users` 등)이 주르륵 뜨고, `client.search.$get()`처럼 메서드까지 자동완성되니 개발 속도도 빨라지고 오타도 줄어. "개발자 어깨춤 추게 하는 자동완성!"
3.  **실제 HTTP 요청/응답 모방**: 단순히 함수 호출해서 로직만 테스트하는 게 아니라, 실제 HTTP 요청처럼 헤더도 보내고, 쿼리 파라미터도 넘기고, 응답으로 받은 상태 코드나 JSON 내용도 확인할 수 있어. 통합 테스트(Integration Test)에 아주 유용해.
4.  **테스트 코드 가독성 UP**: `fetch('/search?q=hono')` 이렇게 직접 URL 문자열 때려 박는 것보다 `client.search.$get({ query: { q: 'hono' } })`처럼 쓰는 게 훨씬 깔끔하고 의미도 명확하게 드러나.

**언제 불려 나오냐?**
주로 Vitest, Jest, Bun Test 같은 자바스크립트 테스트 러너 환경에서 Hono 애플리케이션의 API 엔드포인트별 기능을 검증하는 테스트 코드를 작성할 때 쓰여. `describe('내 API 테스트', () => { it('이 기능은 이렇게 동작해야 해', async () => { ... }) })` 이런 구조 안에서 `testClient(app)`으로 클라이언트 만들고, 그걸로 API 쏴보는 거지.

**쓸 때 꿀팁 및 주의사항:**
*   **라우트 정의는 무조건 체인으로!**: 이게 제일 중요해. `new Hono().get(...).post(...).put(...)` 이런 식으로 줄줄이 사탕 엮듯이 라우트를 정의해야 `testClient`가 타입을 제대로 빨아들여서 자동완성이랑 타입 체크 해준다. `const app = new Hono(); app.get(...); app.post(...);` 이렇게 따로따로 하면 타입 정보 날아가서 "이게 뭔 클라이언트여..." 소리 나온다.
*   **`.$HTTP메서드()` 형태로 호출**: `client.search.$get()`, `client.users.$post()`처럼 라우트 이름 뒤에 `$` 붙이고 HTTP 메서드(get, post, put, delete 등)를 소문자로 써서 호출해야 해. 그냥 `client.search()` 이렇게 하면 안 돼.
*   **요청 바디/쿼리/파라미터 전달**:
    *   GET 요청 시 쿼리 파라미터: `client.path.$get({ query: { key: 'value' } })`
    *   POST/PUT 등 요청 바디: `client.path.$post({ json: { data: 'payload' } })` (JSON 바디일 경우), 또는 `client.path.$post({ form: { field: 'value' } })` (폼 데이터일 경우)
    *   경로 파라미터: `client.users[':id'].$get({ param: { id: '123' } })` (라우트 정의가 `app.get('/users/:id', ...)` 일 때)
*   **헤더는 두 번째 인자**: `client.path.$get({}, { headers: { Authorization: 'Bearer token' } })`처럼 두 번째 인자로 헤더 객체를 넘겨주면 돼.
*   **응답은 `Response` 객체**: `await client.path.$get()`의 결과는 표준 `Response` 객체랑 비슷해서, `res.status`, `await res.json()`, `await res.text()` 같은 익숙한 메서드들로 응답 내용을 확인할 수 있어.
*   **미들웨어도 테스트됨**: `testClient`는 실제 요청처럼 Hono 앱의 미들웨어 스택을 전부 통과하기 때문에, 미들웨어 로직까지 포함된 통합적인 테스트가 가능해. "경비원, 요리사, 서빙 직원 다 거쳐서 나오는 음식 맛보는 격!"
*   **RPC 모드와의 연관성**: `testClient`는 Hono의 RPC(Remote Procedure Call) 모드와 비슷한 방식으로 타입을 추론하고 클라이언트를 생성해. 그래서 Hono RPC에 익숙하다면 `testClient` 사용법도 금방 감 잡을 수 있을 거야.