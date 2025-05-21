`HTTPException` 던지기
이 예제는 미들웨어에서 `HTTPException`을 던지는 걸 보여줍니다.

```javascript
import { HTTPException } from 'hono/http-exception'

// ...

app.post('/auth', async (c, next) => {
  // 인증 과정
  if (authorized === false) {
    throw new HTTPException(401, { message: '커스텀 에러 메시지임' })
  }
  await next()
})
```

사용자에게 반환될 응답을 직접 지정할 수도 있습니다.

```javascript
import { HTTPException } from 'hono/http-exception'

const errorResponse = new Response('인증 안됨ㅋ', {
  status: 401,
  headers: {
    'WWW-Authenticate': 'error="invalid_token"', // 예시에서는 Authenticate로 되어있지만, WWW-Authenticate가 표준입니다.
  },
})

throw new HTTPException(401, { res: errorResponse })
```

---

**얘 뭐 하는 애냐?**
`HTTPException`은 Hono 앱에서 "야, 이거 HTTP 에러 상황이거든? 이 상태 코드랑 메시지로 클라이언트한테 알려줘!" 하고 명시적으로 에러를 발생시키는 Hono 전용 예외 클래스입니다. 일반 `Error` 객체가 그냥 "아파요!" 하고 우는 거라면, `HTTPException`은 "나 지금 401 상태고, 이유는 '인증 실패'야!" 하고 구체적으로 알려주는 확성기 같은 놈이죠.

**왜 쓰는데?**
1.  **명확한 에러 전달**: 클라이언트에게 "너님 요청 문제 있음(4xx)" 또는 "우리 서버 문제 생김(5xx)" 같은 HTTP 상태 코드를 정확히 전달해서, 클라이언트가 다음 행동을 결정하거나 사용자에게 적절한 피드백을 줄 수 있게 합니다. "이거 404니까 그만 찾아라" 하고 알려주는 거죠.
2.  **일관된 에러 처리**: Hono의 에러 처리 미들웨어(`app.onError`)와 찰떡궁합입니다. `HTTPException`을 던지면 `app.onError`에서 이놈을 딱 잡아서 미리 정의된 방식대로 깔끔하게 에러 응답을 만들어낼 수 있습니다. "에러도 체계적으로 관리하자!"
3.  **커스텀 응답 제어**: 단순 메시지뿐만 아니라, 두 번째 예제처럼 `WWW-Authenticate` 헤더가 포함된 `Response` 객체를 통째로 만들어서 던질 수 있습니다. "에러 메시지 봉투에 내용물까지 풀세트로 준비해서 던져주마!"

**언제 불려 나오냐? (언제 `throw` 하나요?)**
주로 미들웨어나 라우트 핸들러 안에서 특정 조건이 만족되지 않아 정상적인 요청 처리를 중단하고 클라이언트에게 에러 상태를 알려줘야 할 때 명시적으로 `throw new HTTPException(...)` 합니다.
*   인증/인가 실패 (예: `throw new HTTPException(401, { message: '로그인 하고 오세요' })`)
*   잘못된 요청 값 (예: `throw new HTTPException(400, { message: 'ID는 숫자여야 합니다' })`)
*   요청한 리소스 없음 (예: `throw new HTTPException(404, { message: '그딴 거 없다' })`)

**쓸 때 꿀팁 및 주의사항:**
*   **첫 번째 인자는 상태 코드, 두 번째는 옵션 객체**: `new HTTPException(상태코드, { message: '메시지', res: Response객체, cause: 원래에러 })` 이런 구조입니다.
    *   `message`: 클라이언트에게 보여줄 간단한 문자열 메시지.
    *   `res`: 이걸 주면 `message`는 무시되고, 여기서 제공한 `Response` 객체가 그대로 에러 응답으로 나갑니다. 헤더 등을 정교하게 제어하고 싶을 때 씁니다.
    *   `cause`: 이 `HTTPException`을 발생시킨 근본 원인 에러 객체를 연결해둘 수 있습니다. 디버깅할 때 유용하죠.
*   **`app.onError`에서 특별 대우**: `app.onError((err, c) => { ... })` 핸들러에서 `if (err instanceof HTTPException)` 이런 식으로 `HTTPException` 타입인지 확인해서 별도의 처리를 해줄 수 있습니다. "얘는 우리가 아는 그놈이네!" 하고 구분해서 다루기 좋죠.
*   **일반 `Error`와의 차이**: 그냥 `throw new Error('망했어요')` 하면 Hono는 보통 500 (Internal Server Error)으로 처리합니다. 하지만 `HTTPException`을 쓰면 400, 401, 403, 404 등 의도한 HTTP 상태 코드를 명확히 전달할 수 있다는 게 핵심입니다.
*   **남발은 금물**: 모든 자잘한 유효성 검사 실패마다 `HTTPException`을 던지는 건 좀 과할 수 있습니다. "모기 잡는데 대포 쏘는 격"이죠. 상황에 따라서는 그냥 일반적인 JSON 응답으로 에러 정보를 전달하는 게 더 깔끔할 수도 있습니다.
*   **메시지는 명확하게**: `message` 옵션에 담는 내용은 API를 사용하는 클라이언트 개발자가 이해하기 쉽도록 명확하게 작성하는 게 좋습니다. 너무 내부적인 에러 코드를 그대로 노출하는 건 "우리 집 내부 사정 다 까발리기"나 마찬가지입니다.