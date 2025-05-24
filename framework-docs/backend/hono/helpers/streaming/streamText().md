`streamText()`
이 함수는 `Content-Type: text/plain`, `Transfer-Encoding: chunked`, 그리고 `X-Content-Type-Options: nosniff` 헤더를 포함하는 스트리밍 응답을 반환합니다. (한마디로, "텍스트 데이터를 실시간으로 찔끔찔끔 보내줄게!" 하는 거죠.)

```typescript
app.get('/streamText', (c) => {
  return streamText(c, async (stream) => {
    // 'Hello' 쓰고 줄바꿈 ('\n') 한번 쳐주고.
    await stream.writeln('Hello')
    // 1초 동안 잠깐 숨 좀 고르고. (잠깐! 멈춰!)
    await stream.sleep(1000)
    // 'Hono!' 쓰고 줄바꿈은 안 함.
    await stream.write(`Hono!`)
  })
})
```

**경고 (WARNING)**

Cloudflare Workers용 애플리케이션을 개발 중이라면, Wrangler(로컬 개발 도구)에서 스트리밍이 제대로 작동하지 않을 수 있습니다. 만약 그렇다면, `Content-Encoding` 헤더에 `Identity`를 추가하세요. (Wrangler 이놈 가끔 말썽이란 말이지...)

```typescript
app.get('/streamText', (c) => {
  // "야, Wrangler! 인코딩 장난치지 말고 그냥 보내!"
  c.header('Content-Encoding', 'Identity')
  return streamText(c, async (stream) => {
    // ... (스트리밍 로직은 위와 동일)
  })
})
```

---

**얘 뭐 하는 애냐?**
`streamText()`는 서버에서 클라이언트로 텍스트 데이터를 한 번에 다 보내는 게 아니라, 마치 수도꼭지에서 물 흘려보내듯 조금씩 나눠서 실시간으로 전송할 때 쓰는 녀석입니다. `Content-Type`은 "이거 그냥 일반 텍스트 파일임"이라고 알려주고, `Transfer-Encoding: chunked`는 "데이터 덩어리(chunk)로 쪼개서 보낼 거니까 다 받을 때까지 기다려!"라는 신호죠. `X-Content-Type-Options: nosniff`는 브라우저가 "어? 이거 HTML인가?" 하고 멋대로 내용물 추측해서 렌더링하는 걸 막는 보안 헤더입니다.

**왜 쓰는데?**
1.  **긴 텍스트 데이터 실시간 표시**: 로그 파일 내용이나, AI가 생성하는 긴 텍스트처럼 한 번에 만들기 오래 걸리거나 용량이 큰 텍스트를 사용자에게 바로바로 보여주고 싶을 때 씁니다. 사용자는 전체 데이터가 다 올 때까지 멍하니 기다릴 필요 없이, 도착하는 대로 내용을 볼 수 있죠. "로딩 중... 대신 실시간 업데이트!"
2.  **서버 부하 감소 (약간)**: 아주 큰 데이터를 한 번에 메모리에 다 올려놓고 보내는 것보다, 생성되는 대로 조금씩 보내면 서버 메모리를 좀 더 효율적으로 쓸 수 있습니다. (물론, 스트리밍 자체의 오버헤드도 있으니 만병통치약은 아님)
3.  **실시간 이벤트 알림 (SSE 유사 효과)**: 간단한 텍스트 기반의 실시간 알림 같은 걸 구현할 때도 써먹을 수 있습니다. 물론 본격적인 실시간 양방향 통신은 WebSocket이 낫지만, 서버에서 클라이언트로 단방향 텍스트 푸시만 필요하다면 가볍게 쓸 수 있죠. "새 글 알림! 댓글 알림! 텍스트로 쏴줄게!"

**언제 불려 나오냐?**
Hono 라우트 핸들러 안에서 `return streamText(c, async (stream) => { ... })` 형태로 호출됩니다. 두 번째 인자로 전달되는 비동기 함수 안에서 `stream` 객체의 메서드들(`writeln`, `write`, `sleep` 등)을 사용해서 데이터를 조금씩 쓰고, 중간중간 딜레이를 주거나 하는 식으로 스트리밍 로직을 구현합니다.

**쓸 때 꿀팁 및 주의사항:**
*   **`stream.writeln()` vs `stream.write()`**: `writeln()`은 텍스트 쓰고 자동으로 줄바꿈 문자(`\n`)를 추가해줍니다. `write()`는 딱 전달한 텍스트만 쓰고요. 로그처럼 줄 단위로 구분되는 데이터를 보낼 땐 `writeln()`이 편하겠죠.
*   **`await stream.sleep()`**: 데이터 사이에 의도적으로 시간 간격을 두고 싶을 때 씁니다. 너무 빨리 보내면 클라이언트가 정신 못 차릴 수도 있고, AI 챗봇처럼 생각하는 시간을 흉내 낼 때도 유용합니다. "잠깐만, 나도 생각할 시간이 필요해."
*   **에러 처리**: 스트리밍 도중에 에러가 발생하면 어떻게 처리할지 고민해야 합니다. `try...catch` 블록으로 감싸서 에러 발생 시 스트림을 안전하게 닫거나, 에러 메시지를 스트림에 써서 클라이언트에게 알릴 수도 있습니다.
*   **클라이언트 측 구현**: 서버에서 `streamText`로 데이터를 보내면, 클라이언트에서는 `fetch` API의 `response.body.getReader()` 같은 걸 써서 스트림 데이터를 읽어와야 합니다. 그냥 `response.text()` 해버리면 스트리밍의 의미가 없어지고 전체 데이터 다 받을 때까지 기다리게 됩니다. "받는 쪽도 스트리밍으로 받아야 짝이 맞지!"
*   **Cloudflare Workers + Wrangler `Content-Encoding: Identity`**: 이건 진짜 개발할 때 빡치는 포인트 중 하나인데, Cloudflare Workers 로컬 개발 환경인 Wrangler에서 가끔 스트리밍이 압축 관련 문제로 꼬일 때가 있습니다. 이때 응답 헤더에 `c.header('Content-Encoding', 'Identity')`를 명시적으로 넣어주면 "압축 같은 거 하지 말고 원본 그대로 보내!"라는 뜻이 돼서 문제가 해결될 때가 많습니다. 배포 환경에서는 보통 필요 없지만, 로컬에서 안 되면 이거부터 확인!
*   **`Transfer-Encoding: chunked`의 의미**: 이 헤더가 있으면 클라이언트는 "아, 데이터가 여러 조각으로 나눠서 오는구나. `Content-Length` 헤더는 없겠네. 각 조각 앞에 크기 정보가 붙어서 오고, 마지막엔 크기 0짜리 조각으로 끝을 알려주겠지." 하고 예상합니다. `streamText`가 알아서 다 해주니 개발자가 직접 신경 쓸 건 별로 없습니다.
*   **`X-Content-Type-Options: nosniff`의 중요성**: 만약 스트리밍하는 텍스트 내용에 `<script>` 같은 HTML 태그가 우연히 포함되어 있고, 브라우저가 이걸 HTML로 착각해서 실행시켜버리면 XSS 공격에 취약해질 수 있습니다. `nosniff`는 "닥치고 `Content-Type`에 명시된 `text/plain`으로만 해석해!" 하고 브라우저의 오지랖을 막아주는 중요한 보안 장치입니다.