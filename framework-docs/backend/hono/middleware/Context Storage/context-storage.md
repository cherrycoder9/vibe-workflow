`hono/context-storage`에서 `contextStorage`와 `getContext`를 가져옵니다.

```typescript
import { Hono } from 'hono'
import { contextStorage, getContext } from 'hono/context-storage'
```

컨텍스트 스토리지 미들웨어는 Hono의 컨텍스트(Context)를 `AsyncLocalStorage`에 저장하여 전역적으로 접근할 수 있게 해줍니다.

**정보**
이 미들웨어는 `AsyncLocalStorage`를 사용합니다. 따라서 실행 환경(런타임)이 이를 지원해야 합니다.
*   Cloudflare Workers: `AsyncLocalStorage`를 활성화하려면 `wrangler.toml` 파일에 `nodejs_compat` 또는 `nodejs_als` 플래그를 추가해야 합니다.

**사용법**
`contextStorage()`가 미들웨어로 적용되면 `getContext()` 함수는 현재 컨텍스트 객체를 반환합니다.

```typescript
// 환경 변수 및 컨텍스트 변수 타입을 정의 (선택 사항이지만 권장)
type Env = {
  Variables: { // c.set/c.get으로 사용하는 변수들
    message: string
  }
}

const app = new Hono<Env>()

// contextStorage 미들웨어를 앱 전체에 적용!
app.use(contextStorage())

// 이 미들웨어에서 컨텍스트에 'message' 값을 설정
app.use(async (c, next) => {
  c.set('message', '안녕! Hono Context Storage!') // c.set으로 값 저장
  await next() // 다음 미들웨어나 핸들러로 제어 넘기기
})

// 핸들러 외부에서도 컨텍스트에 접근할 수 있게 해주는 마법의 함수
const getMessage = () => {
  // getContext<Env>()로 현재 컨텍스트를 가져오고, .var.message로 값에 접근
  return getContext<Env>().var.message
}

app.get('/', (c) => {
  // getMessage 함수를 호출해서 핸들러 외부에서 설정된 메시지를 가져와 응답
  return c.text(getMessage()) // 결과: "안녕! Hono Context Storage!"
})
```

Cloudflare Workers에서는 핸들러 외부에서 바인딩(Bindings)에도 접근할 수 있습니다.

```typescript
// Cloudflare Workers 환경에서의 바인딩 타입 정의
type Env = {
  Bindings: { // wrangler.toml에 정의된 KV 네임스페이스 같은 바인딩
    KV: KVNamespace
  }
}

const app = new Hono<Env>()

app.use(contextStorage()) // 역시나 미들웨어 등록은 필수

// 핸들러 외부에서 KV에 값을 쓰는 함수
const setKV = (value: string) => {
  // getContext<Env>()로 컨텍스트를 가져오고, .env.KV로 KV 바인딩에 접근
  return getContext<Env>().env.KV.put('key', value)
}
```

---

**얘 뭐 하는 애냐?**
`contextStorage` 미들웨어는 Hono의 `c` (컨텍스트 객체)를 `AsyncLocalStorage`라는 특별한 저장 공간에 넣어둬서, Hono의 일반적인 핸들러 함수 바깥에서도 `getContext()` 함수를 통해 `c`에 접근할 수 있게 해주는 마법사야. 마치 "어디서든 꺼내 쓸 수 있는 비밀 주머니에 `c`를 넣어두는 격"이지.

**왜 쓰는데?**
1.  **코드 모듈화 및 분리**: 로깅, 데이터베이스 헬퍼 함수, 유틸리티 함수 등 Hono 핸들러와 직접적인 관련은 없지만 컨텍스트 정보(예: 요청 ID, 사용자 정보, 환경 변수 바인딩)가 필요한 함수들을 별도의 모듈로 깔끔하게 분리할 수 있어. `c`를 함수의 인자로 계속 넘겨주지 않아도 되니까 코드가 훨씬 간결해지지. "맨날 `c` 들고 다니기 귀찮았지? 이제 필요할 때 소환만 해!"
2.  **비동기 컨텍스트 전파**: `AsyncLocalStorage`는 비동기 작업 흐름(async/await) 전반에 걸쳐 컨텍스트를 안전하게 유지해줘. 즉, `await` 때문에 실행 흐름이 잠시 다른 곳으로 넘어갔다 돌아와도, `getContext()`는 여전히 올바른 현재 요청의 컨텍스트를 가리키지. "정신없는 비동기 파티에서도 내 잔은 내가 챙긴다!"
3.  **Cloudflare Workers 바인딩 접근 용이**: 특히 Cloudflare Workers 환경에서 KV 네임스페이스나 D1 데이터베이스 같은 바인딩에 접근해야 하는 헬퍼 함수를 만들 때 유용해. `getContext().env.MY_KV`처럼 핸들러 밖에서도 쉽게 바인딩을 쓸 수 있으니까.

**언제 불려 나오냐?**
Hono 앱의 최상단 또는 특정 라우트 그룹의 시작 지점에 `app.use(contextStorage())` 형태로 미들웨어로 등록해. 이렇게 해둬야 그 이후의 모든 요청 처리 과정에서 `getContext()`가 제대로 작동할 수 있어. "마법을 쓰려면 일단 마법진부터 깔아야지."

**쓸 때 꿀팁 및 주의사항:**
*   **`AsyncLocalStorage` 지원 환경 필수**: 이 기능은 `AsyncLocalStorage`라는 Node.js 기능을 기반으로 해. 따라서 네 Hono 앱이 돌아가는 환경(Cloudflare Workers, Node.js, Deno 등)이 이걸 지원해야 쓸 수 있어. Cloudflare Workers는 `wrangler.toml`에 `nodejs_compat`이나 `nodejs_als` 플래그를 켜줘야 하고. "호환성 체크는 기본 중의 기본!"
*   **`getContext()`는 `contextStorage()` 등록 후에만 유효**: `contextStorage()` 미들웨어가 실행되기 전이나, 적용되지 않은 경로에서 `getContext()`를 호출하면 에러 나거나 `undefined`를 반환할 수 있어. "주문 전에 음식 내놓으라고 하면 안 되지."
*   **타입스크립트와 함께라면 금상첨화**: 예제 코드처럼 `Hono<Env>()`나 `getContext<Env>()` 같이 타입을 명시해주면, `c.var.message`나 `c.env.KV` 같은 속성에 접근할 때 자동완성도 되고 타입 체크도 돼서 훨씬 안전하고 편하게 코딩할 수 있어. "타입 정의는 개발자의 안전모다."
*   **전역 접근의 유혹, 남용은 금물**: 어디서든 컨텍스트에 접근할 수 있다는 건 편리하지만, 너무 남용하면 코드 흐름을 파악하기 어려워지고 의존성이 복잡하게 꼬일 수 있어. 꼭 필요한 경우에만, 그리고 명확한 목적을 가지고 사용하는 게 좋아. "편리함 뒤에는 책임이 따른다."
*   **테스트 시 주의**: 유닛 테스트 같은 걸 할 때 `getContext()`를 사용하는 함수를 테스트하려면, 해당 테스트 환경에서도 `AsyncLocalStorage` 컨텍스트를 적절히 설정하거나 모킹(mocking)해줘야 할 수 있어.
*   **성능 영향은 미미하지만 인지는 하고 있자**: `AsyncLocalStorage` 자체가 큰 성능 저하를 일으키진 않지만, 아주 민감한 고성능 환경이라면 미세한 오버헤드라도 인지하고 있는 게 좋아. 대부분의 경우엔 걱정할 수준은 아니야. "티끌 모아 태산...은 아니지만, 먼지 한 톨 정도는 될 수도?"