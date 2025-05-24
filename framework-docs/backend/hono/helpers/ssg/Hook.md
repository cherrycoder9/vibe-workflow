`toSSG` (Static Site Generation, 정적 사이트 생성) 기능을 쓸 때, 옵션으로 특정 "훅(Hook)" 함수들을 지정해서 그 처리 과정을 내 입맛대로 바꿀 수 있어. 일종의 중간 가로채기 기능이라고 보면 돼. "야, 잠깐만! 그거 그냥 보내지 말고 내가 한번 손 좀 보고 보낼게!" 하는 거지.

제공되는 훅 종류는 다음과 같아:

*   `BeforeRequestHook`: `(req: Request) => Request | false`
    *   `toSSG`가 각 라우트에 요청을 보내기 "직전"에 호출돼.
    *   들어온 `Request` 객체를 받아서, 그대로 반환하거나 수정해서 반환할 수 있어.
    *   만약 `false`를 반환하면, 해당 요청은 건너뛰고 다음으로 넘어가. (이 라우트는 SSG 안 할래!)
*   `AfterResponseHook`: `(res: Response) => Response | false`
    *   `toSSG`가 각 라우트로부터 응답을 받은 "직후"에 호출돼.
    *   받은 `Response` 객체를 살펴보고, 그대로 반환하거나 수정해서 반환할 수 있어.
    *   마찬가지로 `false`를 반환하면, 이 응답은 파일로 저장하지 않고 버려. (이 결과물은 별로군!)
*   `AfterGenerateHook`: `(result: ToSSGResult) => void | Promise<void>`
    *   모든 SSG 작업이 "끝난 후"에 최종 결과(`ToSSGResult`)를 가지고 호출돼.
    *   생성된 파일 목록이나 에러 정보 등을 확인할 수 있고, 이걸로 추가 작업을 할 수도 있어. (예: 사이트맵 생성, 배포 스크립트 실행 등)

**`BeforeRequestHook` / `AfterResponseHook` 활용 예시**

`toSSG`는 기본적으로 앱에 등록된 모든 라우트를 대상으로 정적 파일을 만들려고 해. 근데 특정 라우트는 빼고 싶거나, 특정 조건일 때만 파일을 만들고 싶을 수 있잖아? 그럴 때 이 훅들이 유용해.

예를 들어, GET 요청에 대해서만 파일을 만들고 싶다면 `beforeRequestHook`에서 `req.method`를 확인하면 돼:

```typescript
toSSG(app, fs, {
  beforeRequestHook: (req) => {
    if (req.method === 'GET') {
      return req // GET 요청은 통과!
    }
    return false // 나머지는 빠꾸!
  },
})
```

또는, 응답 상태 코드가 200 (성공) 또는 500 (서버 에러, 근데 이것도 일단 남겨두고 싶을 때)일 때만 파일을 저장하고 싶다면 `afterResponseHook`에서 `res.status`를 확인하면 되고:

```typescript
toSSG(app, fs, {
  afterResponseHook: (res) => {
    if (res.status === 200 || res.status === 500) {
      return res // 200이나 500 응답은 저장!
    }
    return false // 나머지는 버려!
  },
})
```

**`AfterGenerateHook` 활용 예시**

`toSSG` 작업이 다 끝나고 나서 그 결과물을 가지고 뭔가 하고 싶을 때 `afterGenerateHook`을 써.

```typescript
toSSG(app, fs, {
  afterGenerateHook: (result) => {
    // result 객체 안에는 성공 여부, 생성된 파일 목록, 에러 정보 등이 들어있어.
    if (result.files) { // 파일이 성공적으로 생성되었다면
      result.files.forEach((file) => console.log(file)) // 각 파일 경로를 콘솔에 찍어보자!
    }
    // 여기서 result.error 가 있으면 에러 처리 로직을 넣을 수도 있겠지.
  }
})
```

---

**얘 뭐 하는 애냐?**
`toSSG`의 훅들은 Hono가 웹사이트를 통째로 구워서 정적 파일(HTML, CSS, JS 등)로 만드는 과정에 끼어들어서 "잠깐! 이 부분은 이렇게 바꿔줘!" 또는 "이건 굽지 마!" 하고 입김을 불어넣는 기능이야. 건설 현장으로 치면, 설계도대로 건물 짓다가 "아, 이 창문은 좀 더 크게 해주세요!" 하거나 "이 방은 빼주세요!" 하고 중간에 요구사항을 전달하는 감리자 역할이지.

**왜 쓰는데?**
1.  **선택적 SSG**: 앱에 등록된 모든 라우트를 SSG 대상으로 삼는 게 기본이지만, 관리자 페이지나 API 엔드포인트처럼 정적 파일로 만들 필요가 없거나 만들면 안 되는 애들이 있어. `beforeRequestHook`으로 이런 애들을 쏙쏙 골라내서 "넌 빠져!" 할 수 있지.
2.  **조건부 파일 생성**: 응답 상태 코드가 200(성공)인 페이지만 파일로 만들고, 404(찾을 수 없음)나 500(서버 오류) 같은 건 제외하고 싶을 때 `afterResponseHook`이 유용해. "잘 구워진 빵만 팔겠다!"
3.  **후처리 자동화**: SSG 작업이 끝난 후 생성된 파일 목록을 가지고 사이트맵(`sitemap.xml`)을 자동으로 만들거나, 검색 엔진에 "새 글 올렸어요!" 하고 핑을 보내거나, 빌드 결과를 특정 폴더로 복사하는 등의 후속 작업을 `afterGenerateHook`으로 자동화할 수 있어. "빵 다 구웠으면 포장하고 진열까지 자동으로!"
4.  **디버깅 및 로깅**: 각 단계에서 요청, 응답, 최종 결과 등을 로깅해서 SSG 과정에 문제가 없는지, 의도대로 파일이 생성되는지 확인하는 데도 써먹을 수 있어.

**언제 불려 나오냐?**
`toSSG(app, fs, { /* 여기에 훅들 정의 */ })`처럼 `toSSG` 함수를 호출할 때 옵션 객체의 속성으로 이 훅 함수들을 전달하면 돼.
*   `beforeRequestHook`: `toSSG`가 내부적으로 각 라우트에 대해 가상 요청을 날리기 "바로 전".
*   `afterResponseHook`: 가상 요청에 대한 응답을 받고, 그 응답을 파일로 저장하기 "바로 전".
*   `afterGenerateHook`: 모든 라우트에 대한 처리(요청, 응답, 파일 저장)가 "완전히 끝난 후".

**쓸 때 꿀팁 및 주의사항:**
*   **`false` 반환의 의미를 정확히 알자**: `beforeRequestHook`이나 `afterResponseHook`에서 `false`를 반환하면 해당 요청이나 응답은 "무시"돼. 즉, 그 라우트에 대한 정적 파일이 안 만들어진다는 뜻이야. 실수로 중요한 페이지를 빼먹지 않도록 조건문을 잘 짜야 해. "필터 너무 빡세게 걸면 건질 게 없다."
*   **훅 체이닝은 안 돼 (기본적으로)**: 하나의 훅 타입에 여러 함수를 등록하는 기능은 없어. 한 훅 안에서 여러 조건을 다 처리해야 해.
*   **비동기 처리**: `afterGenerateHook`은 `Promise`를 반환할 수 있어서, 파일 생성 후 비동기적인 작업(예: 외부 API 호출)을 수행할 수 있어. `beforeRequestHook`이나 `afterResponseHook`은 동기적으로 `Request`, `Response` 또는 `false`를 반환해야 해.
*   **`Request` / `Response` 객체 조작 시 주의**: 훅 안에서 `Request`나 `Response` 객체를 수정할 수 있지만, 너무 과하게 바꾸면 `toSSG`의 정상적인 동작을 방해할 수도 있어. 꼭 필요한 최소한의 수정만 하는 게 좋아. "튜닝의 끝은 순정이다...는 아니지만 적당히 하자."
*   **에러 핸들링**: `afterGenerateHook`의 `result` 객체에는 `error` 속성이 있어서 SSG 과정 중 발생한 에러 정보를 담고 있을 수 있어. 이걸로 에러 로깅이나 알림 처리를 할 수 있지.
*   **상상력을 발휘해봐**: 단순 필터링 외에도, `beforeRequestHook`에서 요청 헤더를 살짝 바꿔서 특정 조건의 페이지를 생성하게 하거나, `afterResponseHook`에서 응답 내용을 살짝 가공해서 저장하는 등 창의적인 활용도 가능해. (물론 너무 복잡해지면 그냥 Hono 라우트 로직에서 처리하는 게 나을 수도.) "훅으로 어디까지 해봤니?"