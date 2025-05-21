`executionCtx`
클라우드플레어 워커(Cloudflare Workers) 전용 `ExecutionContext`에 접근할 수 있습니다.

```javascript
// ExecutionContext 객체
app.get('/foo', async (c) => {
  c.executionCtx.waitUntil(c.env.KV.put(key, data)) // 응답 보내고 나서 KV에 데이터 저장
  // ...
})
```

---

**얘 뭐 하는 애냐?**
`c.executionCtx`는 Hono 핸들러 안에서 클라우드플레어 워커(Cloudflare Workers)가 제공하는 특별한 실행 컨텍스트(ExecutionContext) 객체에 접근할 수 있게 해주는 통로입니다. 쉽게 말해, 워커 환경에서만 쓸 수 있는 "히든 능력치"나 "보너스 스테이지 입장권" 같은 거죠. 이 객체를 통해 워커의 고급 기능들을 활용할 수 있습니다.

**왜 쓰는데?**
주로 두 가지 강력한 기능 때문에 씁니다:
1.  **`waitUntil(promise)`**: 클라이언트에게 응답을 슝 보낸 후에도, 백그라운드에서 특정 작업(Promise)이 완료될 때까지 워커가 좀 더 살아있도록 합니다. 예를 들어, 사용자 요청 처리 끝내고 응답은 바로 주되, 그 뒤에 몰래 로그를 남기거나, KV 스토어에 데이터를 업데이트하거나, 캐시를 정리하는 등의 "뒷정리" 작업을 시킬 수 있습니다. 고객님은 빠른 응답 받고 좋아하고, 서버는 할 일 마저 하고. 꿩 먹고 알 먹고죠.
2.  **`passThroughOnException()`**: 핸들러에서 얘기치 못한 에러가 빵 터졌을 때, 워커가 에러 페이지를 뱉는 대신 "아, 난 모르겠다! 원래 서버(오리진) 니가 한번 받아봐!" 하고 요청을 그대로 통과시켜 버립니다. 이건 주로 워커를 캐시나 간단한 요청 수정 용도로 쓸 때, 만약 워커가 맛이 가도 원래 사이트는 정상적으로 보이게 하는 "안전빵" 옵션입니다.

**언제 불려 나오냐?**
Hono의 라우트 핸들러 함수 안에서 컨텍스트 객체 `c`를 통해 `c.executionCtx` 형태로 언제든지 꺼내 쓸 수 있습니다. 단, 이건 클라우드플레어 워커 환경에서만 의미가 있고, 다른 환경(예: Node.js, Deno, Bun)에서는 이 값이 `undefined`일 가능성이 매우 높습니다. "우리 동네(워커)에서만 파는 특별 메뉴"라고 생각하세요.

**쓸 때 꿀팁 및 주의사항:**
*   **워커 전용템**: 다시 한번 강조하지만, `c.executionCtx`는 클라우드플레어 워커 환경의 전유물입니다. 다른 데서 "왜 `executionCtx`가 없냐"고 울고불고해도 소용없습니다.
*   **`waitUntil`은 마법이 아니다**: `waitUntil`로 작업을 넘기면 응답 시간은 빨라지지만, 워커의 총 실행 시간(CPU 시간 제한 등)에는 여전히 포함됩니다. 너무 무거운 작업을 `waitUntil`에 때려 넣으면 워커가 비명 지르다 뻗을 수 있습니다. "뒷정리도 적당히!"
*   **`waitUntil` 안에서는 `async/await` 잘 쓰기**: `waitUntil`에 전달하는 프로미스가 제대로 완료될 때까지 기다리게 하려면, 그 안의 비동기 작업들도 `async/await` 등으로 확실하게 처리해야 합니다. 어설프게 넘기면 "뒷정리 시켰는데 왜 안 했냐?" 상태가 될 수 있습니다.
*   **`passThroughOnException` 남용 금지**: 에러 처리는 기본적으로 애플리케이션 레벨에서 명확하게 하는 게 좋습니다. `passThroughOnException`은 최후의 보루 정도로 생각하고, "일단 오리진으로 넘기고 보자"는 식의 안일한 에러 처리는 장기적으로 디버깅을 더 어렵게 만듭니다.
*   **타입스크립트 사용자라면**: `wrangler.toml`이나 환경 설정에 따라 `ExecutionContext` 타입이 자동으로 잡힐 수도 있지만, Hono 컨텍스트에 제네릭으로 `CloudflareWorkerExecutionContext` 같은 타입을 명시해주면 `c.executionCtx`의 메서드들을 더 안전하게 쓸 수 있습니다. (예: `new Hono<{ Bindings: Env, Variables: {}, ExecutionContext: CFExecutionContext }>()`)
*   **로컬 테스트 시 모킹(Mocking)**: 로컬에서 Hono 앱을 개발하고 테스트할 때, `executionCtx`는 당연히 없습니다. Miniflare 같은 워커 시뮬레이션 환경을 쓰거나, 테스트 코드에서 `executionCtx`와 그 메서드들을 적절히 모킹(흉내 내는 가짜 객체 만들기)해야 관련 로직을 검증할 수 있습니다. "집에서는 가짜 총으로 연습!"