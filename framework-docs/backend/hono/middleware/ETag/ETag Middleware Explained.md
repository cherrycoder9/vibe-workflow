# ETag Middleware Explained
이 미들웨어를 쓰면 ETag 헤더를 아주 쉽게 붙일 수 있다.

**가져오기 (Import)**

```typescript
import { Hono } from 'hono'
import { etag } from 'hono/etag'
```

**사용법 (Usage)**

```typescript
const app = new Hono()

// '/etag/'로 시작하는 모든 경로에 etag 미들웨어 적용
app.use('/etag/*', etag())

app.get('/etag/abc', (c) => {
  return c.text('Hono is cool') // 이 응답에 ETag 헤더가 자동으로 붙는다
})
```

**유지되는 헤더들 (The retained headers)**
304 응답(Not Modified)을 보낼 때는, 원래 200 OK 응답에 포함됐을 헤더들도 같이 보내줘야 한다. 기본적으로 `Cache-Control`, `Content-Location`, `Date`, `ETag`, `Expires`, `Vary` 헤더가 여기에 해당된다.

만약 304 응답에 추가로 보내고 싶은 헤더가 있다면, `retainedHeaders` 옵션이랑 기본 헤더 목록이 담긴 `RETAINED_304_HEADERS` 문자열 배열 변수를 써서 지정할 수 있다:

```typescript
import { etag, RETAINED_304_HEADERS } from 'hono/etag'

// ...

app.use(
  '/etag/*',
  etag({
    retainedHeaders: ['x-message', ...RETAINED_304_HEADERS], // x-message 헤더도 304 응답 시 유지
  })
)
```

**옵션 (Options)**
*   `weak` (선택 사항): `boolean`
    약한(weak) ETag를 쓸지 말지 정한다. `true`로 설정하면 ETag 값 앞에 `W/`가 붙는다. 기본값은 `false` (강한 ETag).
*   `retainedHeaders` (선택 사항): `string[]`
    304 응답에 남겨둘 헤더 목록.
*   `generateDigest` (선택 사항): `(body: Uint8Array) => ArrayBuffer | Promise<ArrayBuffer>`
    ETag 값을 만들 때 쓸 커스텀 다이제스트(해시) 생성 함수. 기본은 SHA-1을 쓴다. 이 함수는 응답 본문(`Uint8Array`)을 인자로 받고, 해시값(`ArrayBuffer` 또는 `Promise<ArrayBuffer>`)을 반환해야 한다.

---

**얘 뭐 하는 애냐?**
ETag 헤더를 자동으로 만들어서 HTTP 응답에 달아주는 녀석이다. ETag는 특정 버전의 리소스를 식별하는 HTTP 헤더인데, 쉽게 말해 콘텐츠의 "지문" 같은 거다. 이걸로 브라우저 캐싱을 더 효율적으로 관리할 수 있게 된다. "니 콘텐츠, 버전 몇이냐? 딱 알려주는 거지."

**왜 쓰는데? (왜 이런 기능이 필요하고, 왜 알려주는데?)**
1.  **대역폭 절약 및 속도 향상**: 브라우저가 서버에 "야, 나 이거 전에 받은 건데, 내용 그대로냐?" (`If-None-Match` 헤더에 ETag 값 넣어서) 물어봤을 때, 내용이 안 바뀌었으면 서버는 "ㅇㅇ 그대로임. 그거 써." (`304 Not Modified` 응답) 하고 실제 데이터는 안 보낸다. 이러면 데이터 전송량 줄고, 사용자는 캐시된 걸 바로 보니 빠르다고 느낀다. "데이터 아껴서 뭐하냐, 서버 트래픽 비용이나 줄이자."
2.  **서버 부하 감소**: 똑같은 데이터를 또 생성하고 전송하는 뻘짓을 안 하니 서버도 덜 힘들다.

**언제 불려 나오냐? (언제 이 기능이 동작하냐?)**
1.  **서버가 응답할 때**: 클라이언트에게 HTTP 응답을 보낼 때, 이 미들웨어가 응답 본문 내용을 바탕으로 ETag 값을 계산해서 `ETag` 헤더에 쏙 넣어준다.
2.  **클라이언트가 조건부 요청할 때**: 클라이언트가 이전에 받은 `ETag` 값을 `If-None-Match` 요청 헤더에 담아서 보내면, 서버 쪽에서 이 미들웨어가 현재 리소스의 ETag 값과 비교한다. 같으면 "바뀐 거 없음" (304 응답), 다르면 "새 거 받아라" (200 응답 + 새 ETag) 하는 식.

**쓸 때 꿀팁 및 주의사항:**
*   **`weak` 옵션 (약한 ETag vs 강한 ETag)**: `weak: true`로 설정하면 ETag 앞에 `W/`가 붙는다. 이건 콘텐츠가 의미상으로는 같지만, 바이트 단위로 아주 미세하게 다를 수 있을 때 (예: HTML 주석 변경, 동적으로 찍히는 시간 정보 등) 쓴다. 강한 ETag(기본값)는 바이트 수준까지 완벽히 일치해야 같은 ETag로 취급한다. "상황 봐서 써라. 애매하면 약하게, 확실하면 강하게!"
*   **`retainedHeaders` 옵션**: 304 응답을 보낼 때도 어떤 헤더들은 반드시 포함되어야 한다 (예: `Cache-Control`, `Expires`). `RETAINED_304_HEADERS`는 이런 기본 헤더 목록을 미리 담고 있는 편리한 배열이니, 여기에 필요한 커스텀 헤더만 추가하면 된다. "304 보낸다고 중요한 정보까지 날리면 캐시 정책 꼬인다."
*   **`generateDigest` 옵션**: ETag 값 생성 시 기본은 SHA-1 해시를 쓰는데, 만약 다른 해시 알고리즘(더 빠르거나 보안성이 강한)을 쓰고 싶다면 이 함수를 직접 만들어서 교체할 수 있다. "특별한 이유 없으면 기본빵이 최고. 굳이 건드려서 문제 만들지 말자."
*   **적용 범위**: 예제처럼 특정 경로(`/etag/*`)에만 적용할 수도 있고, 앱 전체에 걸 수도 있다. 주로 내용 변경이 잦지 않은 정적 파일이나 API 응답에 사용하는 게 효율적이다. 매번 내용이 바뀌는 동적인 콘텐츠에는 ETag가 별 의미 없을 수 있다. "총알 아껴 쓰듯, 필요한 곳에만 적용해라."
*   **다른 캐시 헤더와 연동**: ETag는 `Cache-Control`, `Expires` 같은 다른 캐시 제어 헤더와 함께 사용될 때 진가를 발휘한다. ETag는 콘텐츠 변경 여부를 확인하는 역할이고, 실제 캐시 기간이나 조건 등은 다른 헤더들이 담당한다. "ETag 혼자 다 하는 거 아니다. 팀플이 중요!"