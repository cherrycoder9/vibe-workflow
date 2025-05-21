`HTTPException` 처리하기
`app.onError`를 사용해서 던져진 `HTTPException`을 처리할 수 있습니다.

```javascript
import { HTTPException } from 'hono/http-exception'

// ... (중략) ...

app.onError((err, c) => {
  if (err instanceof HTTPException) {
    // 커스텀 응답 가져오기
    return err.getResponse()
  }
  // ... (다른 에러 처리) ...
})
```

---

**얘 뭐 하는 애냐?**
`HTTPException`은 Hono에서 HTTP 에러를 좀 더 똑똑하게 다루려고 만든 특수 에러 클래스입니다. 그냥 `Error` 객체 던지는 것보다 상태 코드(400, 404, 503 등)랑 에러 메시지, 심지어 미리 만들어둔 `Response` 객체까지 담아서 던질 수 있죠. 그리고 `app.onError`는 앱 전역에서 발생하는 에러들을 한 군데서 잡아 처리하는 종합병원 응급실 같은 놈이고요.

결국 이 둘을 같이 쓰면, 코드 중간에서 `throw new HTTPException(404, { message: '페이지 없음ㅋ' })` 이렇게 던졌을 때, `app.onError`가 "어, 너는 HTTPException이네? 너한테 이미 준비된 응답이 있다고? 그럼 그거 그대로 손님한테 전달할게!" 하고 `err.getResponse()`를 호출해서 미리 정의된 응답을 클라이언트에게 보내주는 겁니다. 한마디로, "내가 만든 에러 응답, 그대로 써먹기" 기능이죠.

**왜 쓰는데?**
1.  **에러 응답 표준화**: API 만들다 보면 400 (잘못된 요청), 401 (인증 실패), 403 (권한 없음), 404 (못 찾음) 등등 다양한 HTTP 에러를 뱉어야 하는데, `HTTPException`을 쓰면 이런 에러 응답의 구조(예: JSON 형식, 특정 헤더 포함)를 일관되게 가져갈 수 있습니다. "우리 가게 에러는 다 이 포맷으로 나갑니다!" 도장 찍는 거죠.
2.  **에러 처리 로직 간소화**: `app.onError`에서 `err`가 `HTTPException`인지 아닌지만 구분하면 되니까 코드가 깔끔해집니다. `HTTPException`이면 "네가 알아서 응답 만들어놨지? 그거 써!" 하면 되고, 아니면 "이건 예상 못한 에러네, 500번 에러 처리하자!" 이렇게 분기하기 쉽죠.
3.  **가독성 & 유지보수 UP**: `throw new Error('뭐가 문제임?')` 보다는 `throw new HTTPException(400, { message: '입력값이 이상한데요?' })` 이렇게 쓰는 게 훨씬 명확하잖아요? 나중에 코드를 봐도 "아, 이건 400 에러 상황이구나" 하고 바로 이해할 수 있습니다.

**언제 불려 나오냐?**
앱의 라우트 핸들러나 미들웨어 안에서 `throw new HTTPException(...)` 코드가 실행됐는데, 그 에러를 중간에 `try...catch`로 아무도 안 잡아줬을 때, 최종적으로 `app.onError` 핸들러가 호출됩니다. 그럼 `app.onError` 안에서 `if (err instanceof HTTPException)` 조건문이 "너 혹시 HTTPException 타입이니?" 하고 검문하는 거죠.

**쓸 때 꿀팁 및 주의사항:**
*   **`HTTPException` 만들기**: `new HTTPException(상태코드, 옵션?)` 이렇게 만듭니다.
    *   `상태코드`: 400, 401, 404 같은 HTTP 상태 코드 숫자.
    *   `옵션` (객체):
        *   `message`: 에러 메시지 문자열. 안 주면 Hono가 상태 코드에 맞는 기본 메시지 넣어줍니다.
        *   `res`: 아예 내가 직접 만든 `Response` 객체를 통째로 넣어줄 수도 있습니다. 이걸 쓰면 `err.getResponse()`는 그냥 이 `res`를 반환합니다. "에러 응답, 완전 내 맘대로!"
        *   `cause`: 이 `HTTPException`을 일으킨 원인 에러 객체 (다른 에러를 잡아서 `HTTPException`으로 다시 던질 때 유용).
*   **`err.getResponse()`의 마법**: 이 메서드가 핵심입니다. `HTTPException` 객체 안에 담긴 정보(상태 코드, 메시지, 또는 직접 제공한 `res` 객체)를 바탕으로 최종 `Response` 객체를 만들어서 돌려줍니다. 만약 `options.res`를 안 썼다면, 보통 JSON 형태로 `{"success":false, "message":"에러 메시지"}` 같은 응답을 만들어줍니다.
*   **일반 `Error`와 구분은 필수**: `app.onError`에서는 `if (err instanceof HTTPException)`으로 `HTTPException`을 특별 대우해주고, `else` 구문에서는 나머지 일반 `Error`들을 처리하는 로직(보통 500 에러로 응답)을 반드시 넣어줘야 합니다. 안 그러면 예상 못한 에러 터졌을 때 앱이 그냥 죽거나 이상하게 동작할 수 있어요. "특별 손님은 VVIP룸, 일반 손님은 일반석 안내!"
*   **상태 코드는 숫자로 명확하게**: 괜히 문자열로 넣거나 하지 말고, 400, 401, 403, 404, 500 등 표준 HTTP 상태 코드를 숫자로 정확히 사용하세요. 이게 국룰입니다.
*   **커스텀 응답의 끝판왕, `options.res`**: `new HTTPException(404, { res: c.html('<h1>페이지 없어요! 돌아가세요!</h1>', 404) })` 이런 식으로 `res` 옵션에 Hono의 `c.html()`, `c.text()`, `c.json()` 등으로 만든 `Response` 객체를 넣어버리면, 에러 응답을 HTML이든, 특정 헤더가 포함된 JSON이든, 뭐든 원하는 대로 완벽하게 제어할 수 있습니다. "이 구역의 에러 응답 디자인은 내가 한다!"