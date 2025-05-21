`parseBody()`
`multipart/form-data` 또는 `application/x-www-form-urlencoded` 타입의 요청 본문을 파싱합니다.

```javascript
app.post('/entry', async (c) => {
  const body = await c.req.parseBody()
  // ... 뭔가 하겠죠 ...
})
```

`parseBody()`는 다음과 같은 동작을 지원합니다.

**단일 파일**

```javascript
const body = await c.req.parseBody()
const data = body['foo']
```
`body['foo']`는 `(string | File)` 타입입니다.

만약 여러 파일이 같은 이름으로 업로드되면, 마지막 파일만 사용됩니다.

**다중 파일 (필드 이름에 `[]` 사용)**

```javascript
const body = await c.req.parseBody()
body['foo[]'] // 또는 body.foo
```
`body['foo[]']` (또는 `body.foo`로 접근 시)는 항상 `(string | File)[]` 배열 타입입니다.

필드 이름 끝에 `[]`를 붙여야 합니다.

**같은 이름의 여러 파일 또는 필드 (`all: true` 옵션)**
`<input type="file" multiple />`처럼 여러 파일을 한 번에 올리거나, `<input type="checkbox" name="favorites" value="Hono"/>`처럼 같은 이름의 체크박스를 여러 개 사용할 때가 있죠.

```javascript
const body = await c.req.parseBody({ all: true })
body['foo']
```
`all` 옵션은 기본적으로 꺼져있습니다 (`false`).

만약 `body['foo']`가 여러 파일이면 `(string | File)[]` 배열로 파싱되고,
단일 파일이면 `(string | File)`로 파싱됩니다.

**점 표기법 (Dot notation) (`dot: true` 옵션)**
`dot` 옵션을 `true`로 설정하면, 반환값을 점 표기법 기준으로 구조화합니다.

다음과 같은 데이터를 받았다고 상상해보세요:

```javascript
const data = new FormData()
data.append('obj.key1', 'value1')
data.append('obj.key2', 'value2')
```

`dot` 옵션을 `true`로 설정해서 구조화된 값을 얻을 수 있습니다:

```javascript
const body = await c.req.parseBody({ dot: true })
// body는 이제 `{ obj: { key1: 'value1', key2: 'value2' } }` 요렇게 됩니다.
```

---

**얘 뭐 하는 애냐?**
`c.req.parseBody()`는 클라이언트가 폼(form)으로 낑낑대며 보낸 데이터 덩어리(`multipart/form-data`나 `application/x-www-form-urlencoded`)를 개발자가 써먹기 좋게 자바스크립트 객체로 착착 정리해주는 해결사입니다. 파일이든 일반 텍스트 값이든 "일단 맡겨만 주십쇼!" 하고 다 받아주죠.

**왜 쓰는데?**
1.  **폼 데이터 해독**: HTML 폼으로 날아오는 데이터는 그냥 문자열 덩어리거나 복잡한 형식이라 바로 쓰기 어렵습니다. 이걸 `key-value` 형태의 객체로 바꿔주니 "아, 'username' 필드에 '홍길동'이라고 왔구나!" 하고 바로 알 수 있죠.
2.  **파일 업로드 간편 처리**: `<input type="file">`로 보낸 파일도 `File` 객체로 뿅 만들어줘서 파일 이름, 타입, 내용 등을 쉽게 다룰 수 있게 됩니다. 직접 바이너리 파싱하는 지옥에서 해방시켜주는 거죠.
3.  **표준 방식 준수**: 웹 표준 폼 전송 방식을 따르기 때문에, 어떤 클라이언트에서 폼을 보내든 일관되게 처리할 수 있습니다. "근본 있는 처리 방식"입니다.

**언제 불려 나오냐?**
주로 `POST`나 `PUT` 요청처럼 클라이언트가 서버로 데이터를 "보낼" 때, 그 요청을 처리하는 핸들러 함수 안에서 `await c.req.parseBody()` 형태로 호출됩니다. 요청 본문(body)을 읽어야 할 때 등판하는 거죠. 비동기 함수니까 `await` 안 붙이면 프로미스 객체만 덩그러니 받게 되니 주의! "기다림의 미학, `await`!"

**쓸 때 꿀팁 및 주의사항:**
*   **`await`는 필수!**: 비동기 함수입니다. `await` 빼먹으면 "데이터 어디 갔냐?" 하고 울부짖게 됩니다.
*   **`Content-Type` 확인 사살**: 요청 헤더의 `Content-Type`이 `multipart/form-data` (파일 있을 때) 또는 `application/x-www-form-urlencoded` (파일 없고 텍스트 필드만 있을 때)여야 제대로 작동합니다. 엉뚱한 `Content-Type`이면 에러 나거나 빈 객체만 덩그러니. "문패 보고 들어가야지, 아무 데나 들어가면 안 돼!"
*   **파일 객체 반환**: 파일은 `File` 객체로 넘어옵니다. 이 객체 안에는 파일 이름(`name`), 크기(`size`), 타입(`type`), 그리고 실제 내용을 읽을 수 있는 `arrayBuffer()`나 `text()` 같은 메서드가 들어있죠. Hono는 파싱만 해주고 파일 저장까지는 안 해주니, 저장 로직은 알아서 짜야 합니다. "밥상 차려줬으니, 숟가락은 네가 들어라."
*   **옵션 활용은 센스**:
    *   **`foo[]` vs `all: true`**: 필드 이름에 `[]`를 붙이는 건 "나 배열이요!" 하고 명시적으로 선언하는 느낌이고, `all: true` 옵션은 "같은 이름으로 여러 개 오면 알아서 배열로 묶어줘" 하는 느낌입니다. 여러 값을 받는 체크박스(`name="hobby"`)나 다중 파일 업로드(`multiple`) 쓸 때 유용하죠. `all: true` 안 쓰면 같은 이름의 필드가 여러 개 와도 마지막 놈만 덩그러니 남습니다. "하나만 기억하는 더러운 세상!" (이 아니라 옵션 안 쓴 내 탓)
    *   **`dot: true`**: `user.name=hono&user.email=hono@dev.com` 같은 데이터를 `{ user: { name: 'hono', email: 'hono@dev.com' } }` 이렇게 예쁜 중첩 객체로 만들어줍니다. 데이터 구조가 깔끔해지지만, 너무 깊게 중첩된 이름은 오히려 "이게 어디 붙은 키냐?" 하고 헷갈릴 수 있으니 적당히 쓰는 게 좋습니다.
*   **보안! 보안! 보안!**: 파일 업로드 기능은 언제나 보안 취약점의 단골손님입니다. 파일 이름, 확장자, 크기, 실제 내용(MIME 타입 검증 등)을 꼼꼼히 검사해서 악성 파일이 서버에 발붙이지 못하게 해야 합니다. "아무거나 막 받으면 서버 터진다!"
*   **대용량 파일은 조심**: `parseBody`는 기본적으로 요청 전체를 메모리에 올려서 처리합니다. 아주 큰 파일(몇백MB, GB 단위)을 이렇게 받으면 서버 메모리가 터져나갈 수 있습니다. 이럴 땐 스트리밍 방식으로 처리하는 걸 고려해야 하는데, 이건 Hono의 `parseBody` 기본 기능보다는 좀 더 로우레벨 처리나 외부 라이브러리, 또는 Cloudflare Workers 같은 환경 자체의 스트리밍 지원을 활용해야 할 수 있습니다. "뷔페 음식도 한 번에 다 담으면 접시 깨진다."