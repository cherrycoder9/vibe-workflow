# Timeout Middleware Explained
타임아웃 미들웨어는 애플리케이션에서 요청 시간 초과를 쉽게 관리할 수 있게 해주는 녀석이야. 요청에 대한 최대 처리 시간을 설정하고, 지정된 시간을 초과하면 선택적으로 사용자 정의 오류 응답을 정의할 수 있게 해주지. "기다리다 지쳐 쓰러지기 전에 끊어버리겠다, 이 말씀이야."

**가져오기 (Import)**

```typescript
import { Hono } from 'hono'
import { timeout } from 'hono/timeout'
```

**사용법 (Usage)**
기본 설정과 사용자 정의 설정을 모두 사용하는 타임아웃 미들웨어 사용법은 다음과 같아:

**기본 설정:**

```typescript
const app = new Hono()

// '/api' 경로에 5초 타임아웃 적용
app.use('/api', timeout(5000))

// 라우트 처리
app.get('/api/data', async (c) => {
  // 여기에 라우트 핸들러 로직 작성
  return c.json({ data: '여기에 네 데이터가 들어간다' })
})
```

**사용자 정의 설정:**

```typescript
import { HTTPException } from 'hono/http-exception'

// 사용자 정의 예외 생성 함수
const customTimeoutException = (context) =>
  new HTTPException(408, {
    message: `요청이 ${context.req.headers.get(
      'Duration'
    )}초 동안 대기 후 시간 초과되었습니다. 나중에 다시 시도해주세요.`,
  })

// 정적 예외 메시지의 경우
// const customTimeoutException = new HTTPException(408, {
//   message: '작업 시간이 초과되었습니다. 나중에 다시 시도해주세요.'
// });

// '/api/long-process' 경로에 사용자 정의 예외와 함께 1분 타임아웃 적용
app.use('/api/long-process', timeout(60000, customTimeoutException))

app.get('/api/long-process', async (c) => {
  // 오래 걸리는 작업 시뮬레이션
  await new Promise((resolve) => setTimeout(resolve, 61000))
  return c.json({ data: '이건 보통 더 오래 걸리는 데이터야' })
})
```

**참고 사항 (Notes)**
타임아웃 시간은 밀리초 단위로 지정할 수 있어. 지정된 시간을 초과하면 미들웨어가 자동으로 프로미스를 거부하고 오류를 발생시킬 수 있지. "1초는 1000밀리초, 헷갈리지 말라고."

타임아웃 미들웨어는 스트리밍 방식과 함께 사용할 수 없어. 따라서 `stream.close()`와 `setTimeout`을 함께 사용해야 해. "스트리밍은 예외다, 이 말이야. 따로 놀아야 한다고."

```typescript
app.get('/sse', async (c) => {
  let id = 0
  let running = true
  let timer: number | undefined

  return streamSSE(c, async (stream) => {
    timer = setTimeout(() => {
      console.log('스트림 시간 초과, 스트림 닫는 중')
      stream.close()
    }, 3000) as unknown as number // NodeJS.Timeout 대신 number로 단언

    stream.onAbort(async () => {
      console.log('클라이언트가 연결을 닫음')
      running = false
      clearTimeout(timer)
    })

    while (running) {
      const message = `현재 시간은 ${new Date().toISOString()} 입니다`
      await stream.writeSSE({
        data: message,
        event: 'time-update',
        id: String(id++),
      })
      await stream.sleep(1000)
    }
  })
})
```

**미들웨어 충돌 (Middleware Conflicts)**
미들웨어 순서에 주의해야 해. 특히 오류 처리나 다른 타이밍 관련 미들웨어를 사용할 때 이 타임아웃 미들웨어의 동작에 영향을 줄 수 있으니 조심하라고. "줄 잘 서야 한다, 안 그러면 꼬인다."

---

**얘 뭐 하는 애냐?**
말 그대로 요청 처리 시간을 제한하는 기능이야. 서버가 특정 요청을 처리하는 데 너무 오랜 시간을 쓰지 않도록 상한선을 정해두는 거지. "무한 대기는 이제 그만! 서버도 좀 쉬자."

**왜 쓰는데? (어떤 목적으로 사용되나?)**
1.  **서버 자원 보호**: 어떤 요청이 비정상적으로 오래 걸리거나, 악의적인 공격으로 서버 자원을 계속 붙잡고 늘어지는 걸 막아줘. "내 서버는 내가 지킨다!"
2.  **사용자 경험 개선**: 사용자가 응답 없는 화면만 멍하니 쳐다보게 하는 대신, "야, 이거 시간 너무 걸리는데? 나중에 다시 해봐" 하고 빠르게 알려줄 수 있어.
3.  **시스템 안정성**: 특정 요청 하나 때문에 서버 전체가 맛이 가는 걸 방지해. "하나가 썩었다고 전체를 버릴 순 없잖아?"

**언제 불려 나오냐? (언제 호출되나?)**
클라이언트로부터 HTTP 요청이 들어와서 Hono 앱이 해당 요청을 처리하기 시작할 때, 이 미들웨어가 설정된 경로라면 가장 먼저 또는 지정된 순서에 따라 호출돼. 요청 처리 시작과 동시에 타이머를 돌리기 시작하는 거지.

**쓸 때 꿀팁 및 주의사항:**
*   **적절한 타임아웃 값 설정**: 너무 짧으면 정상적인 요청도 시간 초과로 실패할 수 있고, 너무 길면 있으나 마나 한 기능이 돼. 서비스 특징과 API 응답 시간을 고려해서 신중하게 설정해야 해. "밀당 잘해야 본전이다."
*   **커스텀 에러 메시지는 센스**: 그냥 "타임아웃!" 하고 띡 던지는 것보다, `customTimeoutException` 예시처럼 사용자에게 좀 더 친절하고 구체적인 상황을 알려주는 게 좋아. "고객님, 당황하셨어요? 잠시 후 다시 시도해주세요~" 이런 느낌.
*   **스트리밍은 특별 취급**: 위에 나온 것처럼 스트리밍 응답(SSE 등)에는 이 미들웨어가 직접 안 먹히니까, `setTimeout`이랑 `stream.close()` 조합으로 직접 구현해야 한다는 점 잊지 마. "얘는 VIP라 전용 통로가 필요하다."
*   **미들웨어 순서가 생명**: 다른 미들웨어, 특히 에러 처리 미들웨어나 로깅 미들웨어와의 순서에 따라 동작이 달라질 수 있어. 타임아웃이 먼저냐, 다른 게 먼저냐에 따라 결과가 천차만별일 수 있으니 주의. "선빵필승일 수도, 아닐 수도."
*   **테스트는 필수**: 다양한 상황(정상 응답, 느린 응답, 네트워크 오류 등)에서 타임아웃이 의도한 대로 잘 작동하는지 충분히 테스트해봐야 뒤탈이 없어. "실전은 연습처럼, 연습은 실전처럼!"