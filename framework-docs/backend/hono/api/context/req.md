`req`는 `HonoRequest`의 인스턴스입니다. 자세한 건 `HonoRequest` 문서를 참고하세요.

```javascript
app.get('/hello', (c) => {
  const userAgent = c.req.header('User-Agent')
  // ... 뭔가 하겠죠 ...
})
```

---

**얘 뭐 하는 애냐?**
`c.req`는 Hono의 핸들러 함수 안에서 현재 들어온 HTTP 요청에 대한 모든 정보를 담고 있는 객체입니다. 클라이언트가 보낸 헤더, URL 경로, 쿼리 파라미터, 요청 본문(body) 등을 꺼내 쓸 때 이놈을 통하게 되죠. 한마디로 "손님이 뭘 들고 왔는지, 뭘 원하는지 알려주는 서류철" 같은 겁니다. `HonoRequest`라는 특별한 클래스의 인스턴스인데, 표준 `Request` 객체를 한번 감싸서 Hono만의 편리한 기능들을 추가한 녀석이라고 보면 됩니다.

**왜 쓰는데?**
1.  **요청 정보 접근의 관문**: 클라이언트 IP 주소, 요청 메서드(GET, POST 등), 쿠키, 요청 본문 데이터(JSON, 폼 데이터 등) 같이 요청 처리에 필요한 거의 모든 정보에 `c.req`를 통해 접근할 수 있습니다. "손님, 신분증이랑 요청서 좀 보여주시죠?" 하는 역할이죠.
2.  **편의 메서드 제공**: 표준 `Request` 객체보다 더 쓰기 편한 메서드들을 제공합니다. 예를 들어 `c.req.json()`은 요청 본문을 바로 JSON으로 파싱해주고, `c.req.param('id')`는 경로 파라미터를 쉽게 가져오게 해주고, `c.req.header('User-Agent')`는 특정 헤더 값을 간단히 읽게 해줍니다. 개발자 편의성을 높여주는 "잔기술"들이죠.
3.  **데이터 유효성 검사 지원**: `c.req.valid('query')`나 `c.req.valid('json')`처럼 Hono의 유효성 검사 미들웨어(validator middleware)와 연동해서, 검증을 통과한 안전한 데이터만 딱 뽑아 쓸 수 있게 해줍니다. "수상한 물건은 반입금지!"

**언제 불려 나오냐? (언제 사용되냐?)**
Hono의 라우트 핸들러 함수나 미들웨어 함수 안에서, 클라이언트가 보낸 요청과 관련된 정보를 얻고 싶을 때마다 사용됩니다. `c` (컨텍스트 객체)의 속성으로 존재하기 때문에, 핸들러 함수의 첫 번째 인자로 `c`를 받아서 `c.req.어쩌고()` 형태로 쓰게 되죠.

**쓸 때 꿀팁 및 주의사항:**
*   **`c.req.param()` vs `c.req.query()` vs `c.req.header()`**:
    *   `c.req.param('id')`: URL 경로에서 값 가져오기 (예: `/users/:id`)
    *   `c.req.query('search')`: URL 쿼리 스트링에서 값 가져오기 (예: `/items?search=Hono`)
    *   `c.req.header('Content-Type')`: HTTP 헤더에서 값 가져오기.
    *   각각 단일 값을 가져올 땐 `('key')`를, 전체를 객체로 가져올 땐 `()`를 씁니다. "어디 숨겨놨냐? 경로? 쿼리? 헤더?"
*   **요청 본문(Body) 읽기**:
    *   `await c.req.json()`: JSON 데이터 읽기.
    *   `await c.req.text()`: 텍스트 데이터 읽기.
    *   `await c.req.arrayBuffer()`: ArrayBuffer 형태로 읽기.
    *   `await c.req.formData()`: `multipart/form-data` 또는 `application/x-www-form-urlencoded` 데이터 읽기.
    *   **주의!** `c.req.json()` 같은 본문 읽기 메서드는 한 번만 호출할 수 있습니다. 내부적으로 스트림을 소모하기 때문이죠. 두 번 부르면 에러 나거나 빈 값 나옵니다. "한 번 맛보면 끝! 리필 안 돼요!"
*   **`c.req.valid('type')`**: Hono의 유효성 검사 미들웨어(`zValidator` 등)를 썼을 때, 검증을 통과한 "깨끗한" 데이터만 가져옵니다. `type`에는 `'query'`, `'json'`, `'form'`, `'param'`, `'header'` 등이 들어갈 수 있습니다. 안심하고 쓸 수 있는 데이터죠. "깐깐한 문지기 통과한 놈들만 드루와."
*   **원본 `Request` 객체 접근**: 만약 HonoRequest가 제공하는 편의 기능 말고 진짜 표준 `Request` 객체의 무언가를 써야 한다면, `c.req.raw`를 통해 접근할 수 있습니다. "가공 전 원본 그대로!"
*   **URL 정보**: `c.req.url` (전체 URL), `c.req.path` (경로명), `c.req.method` (HTTP 메서드) 등도 자주 쓰입니다.
*   **타입스크립트 사용자라면**: `HonoRequest`의 제네릭을 활용하면 `c.req.valid()` 등으로 가져온 데이터의 타입을 명확하게 지정할 수 있어서 개발 경험이 훨씬 좋아집니다. "타입 모르면 너만 손해!"