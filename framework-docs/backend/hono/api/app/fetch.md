`app.fetch`는 당신 앱의 정문 역할을 할 겁니다.

클라우드플레어 워커에서는 이렇게 쓰면 됩니다:

```javascript
export default {
  fetch(request: Request, env: Env, ctx: ExecutionContext) {
    return app.fetch(request, env, ctx)
  },
}
```

아니면 그냥 이렇게 해도 되고요:

```javascript
export default app
```

번(Bun)에서는 이렇게 씁니다:

```javascript
export default {
  port: 3000,
  fetch: app.fetch,
}
```

---

**얘 뭐 하는 애냐?**
`app.fetch`는 Hono 애플리케이션의 심장 같은 놈입니다. HTTP 요청(`Request` 객체)을 딱 받아서, Hono가 내부적으로 라우팅하고 처리할 수 있도록 전달해주는 메인 핸들러 함수죠. 한마디로 "야, 손님 왔다! 네가 맡아라!" 하고 Hono 코어 로직에 일감 넘기는 역할입니다.

**왜 쓰는데?**
1.  **만능 플러그**: 클라우드플레어 워커, Deno, Bun, Node.js 등등 별의별 자바스크립트 런타임이나 서버 환경에 Hono 앱을 똑같은 방식으로 착 붙일 수 있게 해줍니다. 각 환경마다 HTTP 요청 처리 방식이 미묘하게 다른데, `app.fetch`가 그 중간에서 "통역사" 역할을 해주니 개발자는 Hono 로직 짜는 데만 집중하면 됩니다. "어댑터 패턴"의 좋은 예시죠.
2.  **이식성 갑**: Hono로 앱 한번 잘 짜놓으면, `app.fetch` 덕분에 이 서버 저 서버, 이 플랫폼 저 플랫폼으로 옮겨 다니기가 아주 수월해집니다. "어디든 간다!" 시전 가능.

**언제 불려 나오냐?**
서버나 런타임이 클라이언트로부터 HTTP 요청을 딱 받았을 때! 바로 그때 해당 환경의 기본 코드나 개발자가 설정한 진입점에서 `app.fetch(request, ...)` 형태로 호출됩니다. 예를 들어, 클라우드플레어 워커에서는 워커가 요청을 받을 때마다 `export default { fetch: app.fetch }` 요 부분이 실행되면서 `app.fetch`가 등판하는 거죠.

**쓸 때 꿀팁 및 주의사항:**
*   **`Request`는 기본, `env`랑 `ctx`는 옵션 (근데 은근 중요!)**:
    *   `request`: HTTP 요청 정보를 담은 표준 `Request` 객체. 이게 없으면 시작도 못 합니다.
    *   `env` (주로 Cloudflare Workers): 환경 변수, KV 네임스페이스, D1 데이터베이스 같은 바인딩된 자원에 접근할 때 씁니다. 일종의 "사장님 찬스"죠. 이거 잘 써야 워커 파워 제대로 뽑아냅니다.
    *   `ctx` (주로 Cloudflare Workers의 `ExecutionContext`): `waitUntil()`, `passThroughOnException()` 같은 메서드를 제공합니다. 요청 처리 후 비동기 작업(예: 로그 남기기, 캐시 업데이트)을 이어가거나, 예외 터졌을 때 요청을 원래 서버(오리진)로 넘길 때 씁니다. "뒷정리 부탁해!" 또는 "에잇, 난 모르겠다! 본진으로 넘겨!" 할 때 쓰는 카드.
*   **리턴은 `Promise<Response>` 또는 `Response`**: `app.fetch`는 반드시 `Response` 객체 아니면 `Response` 객체로 귀결되는 프로미스(Promise)를 반환해야 합니다. 안 그러면 "응답 어디 갔냐?" 소리 듣고 에러 파티 열립니다.
*   **얘는 그냥 우체부**: `app.fetch` 자체가 엄청 복잡한 로직을 처리하는 게 아닙니다. 요청을 받아서 Hono 내부 라우터로 안전하게 배달해주는 역할이죠. 실제 일은 라우터랑 그 뒤에 줄줄이 사탕으로 엮인 핸들러들이 다 합니다.
*   **타입스크립트 + `Env` = 꿀조합**: 타입스크립트 쓸 때 `Env` 타입을 제네릭으로 Hono 앱에 딱 지정해주면 (`const app = new Hono<{ Bindings: Env }>()` 요런 식으로), `c.env` 쓸 때 자동완성 뜨고 타입 체크도 돼서 개발 경험이 한층 쾌적해집니다. 안 그러면 `any` 타입 남발하다가 런타임 에러 맞고 "왜 안 되지?" 시전하기 딱 좋습니다.
*   **`export default app`의 비밀**: 클라우드플레어 워커에서 `export default app` 이렇게만 해도 돌아가는 건, Hono 인스턴스 자체가 `fetch` 메서드를 가지고 있기 때문입니다. 내부적으로는 `app.fetch`를 호출하는 거랑 거의 똑같이 작동해요. 코드는 깔끔해 보이지만, `env`나 `ctx`를 좀 더 명시적으로 다루고 싶다면 객체 형태로 익스포트하는 게 나을 수도 있습니다.