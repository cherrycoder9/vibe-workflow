# IP_Restriction_Middleware_Hono

IP 제한 미들웨어는 사용자 IP 주소를 기반으로 리소스 접근을 통제하는 녀석이다. "우리 구역엔 너 같은 놈 못 들어온다" 시전하는 문지기 역할이지.

**가져오기 (Import)**

```typescript
import { Hono } from 'hono'
import { ipRestriction } from 'hono/ip-restriction'
```

**사용법 (Usage)**
Bun 환경에서 돌아가는 앱이 로컬에서만 접근을 허용하고 싶다면, 아래처럼 짜면 된다. 거부할 규칙은 `denyList`에, 허용할 규칙은 `allowList`에 명시한다. "화이트리스트, 블랙리스트 그거 맞다."

```typescript
import { Hono } from 'hono'
import { getConnInfo } from 'hono/bun' // Bun 환경용
import { ipRestriction } from 'hono/ip-restriction'

const app = new Hono()

app.use(
  '*', // 모든 경로에 적용
  ipRestriction(getConnInfo, { // 첫 번째 인자로 환경에 맞는 getConnInfo 함수 전달
    denyList: [], // 여긴 비워두고
    allowList: ['127.0.0.1', '::1'], // 로컬호스트 IP (IPv4, IPv6)만 허용
  })
)

app.get('/', (c) => c.text('Hello Hono!'))
```

`ipRestriction` 함수의 첫 번째 인자로는 현재 실행 환경에 맞는 `ConnInfo` 헬퍼의 `getConnInfo` 함수를 넘겨줘야 한다. 예를 들어, Deno 환경이라면 아래처럼 쓴다. "환경 따라 연장 챙기는 건 기본."

```typescript
import { getConnInfo } from 'hono/deno' // Deno 환경용
import { ipRestriction } from 'hono/ip-restriction'

//...

app.use(
  '*',
  ipRestriction(getConnInfo, { // Deno용 getConnInfo 사용
    // ... 규칙은 위와 동일하게
  })
)
```

**규칙 (Rules)**
규칙 작성은 아래 지침을 따른다.

*   **IPv4**
    *   `192.168.2.0` - 고정 IP 주소
    *   `192.168.2.0/24` - CIDR 표기법 (IP 대역)
    *   `*` - 모든 주소 ("프리패스" 혹은 "전부 차단"에 쓰임)
*   **IPv6**
    *   `::1` - 고정 IP 주소
    *   `::1/10` - CIDR 표기법
    *   `*` - 모든 주소

**에러 처리 (Error handling)**
접근 거부 시 보여줄 에러 메시지를 바꾸고 싶으면, 세 번째 인자로 `Response` 객체를 반환하는 함수를 넣어주면 된다. "차단 메시지도 센스 있게."

```typescript
app.use(
  '*',
  ipRestriction(
    getConnInfo,
    {
      denyList: ['192.168.2.0/24'], // 이 대역은 차단
    },
    async (remote, c) => { // remote 객체에는 차단된 IP 정보(remote.addr)가 들어있음
      return c.text(`님 IP(${remote.addr})는 우리랑 안 맞네요. 접근 불가.`, 403) // 403 Forbidden 응답
    }
  )
)
```

---

**얘 뭐 하는 애냐?**
말 그대로 IP 주소 보고 문전박대하거나 VIP 대접해 주는 미들웨어다. 특정 IP 대역만 서비스에 접근하게 하거나, 반대로 악성 IP들을 차단하는 데 쓴다. "IP로 신분 검사하는 셈."

**왜 쓰는데? (왜 이런 기능이 필요하고, 왜 알려주는데?)**
1.  **보안 강화**: 특정 국가나 알려진 악성 IP 대역からの攻撃を未然に防ぐ。내부 관리자 페이지 같은 건 특정 IP에서만 접속하게 해서 보안 수준을 높일 수 있다. "우리 집 대문은 아무나 못 열지."
2.  **접근 제어**: 특정 지역 사용자에게만 서비스를 제공하거나, 반대로 특정 지역은 차단할 때. 또는 회사 내부망에서만 접근 가능한 서비스를 만들 때도 유용하다.

**언제 불려 나오냐? (언제 이 기능이 동작하냐?)**
서버에 요청이 들어오면, 본격적인 요청 처리 전에 이 미들웨어가 먼저 나서서 방문객의 IP 주소를 스캔한다. `allowList`나 `denyList` 규칙에 걸리면 거기서 바로 차단하거나 통과시킨다.

**쓸 때 꿀팁 및 주의사항:**
*   **`getConnInfo`는 환경 따라**: Bun, Deno, Cloudflare Workers 등 실행 환경마다 실제 사용자 IP를 가져오는 방법이 다르니, 환경에 맞는 `getConnInfo`를 써야 한다. 안 그러면 엉뚱한 IP(예: 프록시 서버 IP)로 판단해서 다 막거나 다 풀어주는 대참사가 날 수 있다. "옷은 TPO, 코드는 환경에 맞게."
*   **`allowList` vs `denyList`**: 둘 다 쓸 수 있지만, 로직 꼬이기 쉽다. 보통 `denyList`가 우선 적용되는 경우가 많지만, Hono 문서를 다시 한번 확인하거나, 그냥 하나만 명확하게 쓰는 게 속 편하다. "심플 이즈 베스트."
*   **CIDR 표기법 마스터**: `172.16.0.0/12` 같은 CIDR 표기법에 익숙해지면 IP 대역을 한 줄로 깔끔하게 관리할 수 있다. IP 계산기 사이트 같은 거 활용하면 편하다.
*   **동적 IP는 골치**: 일반 가정집 인터넷처럼 IP가 수시로 바뀌는 환경의 사용자를 `allowList`로 관리하려면 IP 바뀔 때마다 업데이트해줘야 해서 매우 귀찮다. "고정 IP 아니면 답 없다."
*   **프록시/로드밸런서 뒤에 있다면**: 서버 앞에 리버스 프록시나 로드밸런서가 있으면 실제 클라이언트 IP 대신 프록시 IP가 넘어올 수 있다. `X-Forwarded-For` 헤더 등을 확인해서 실제 IP를 가져오도록 `getConnInfo`가 잘 처리해 주는지, 또는 별도 설정이 필요한지 확인해야 한다.
*   **에러 메시지는 친절하게**: 차단됐을 때 기본 에러 페이지만 덜렁 보여주는 것보다, 왜 차단됐는지 (예: "해외 IP 접근 제한") 알려주면 사용자 경험이 좀 더 낫다. 물론, 너무 자세한 정보는 보안에 안 좋을 수도 있으니 적당히.
*   **`*` (모든 주소)는 신중하게**: `allowList: ['*']`는 사실상 아무것도 안 막는 거고, `denyList: ['*']`는 전부 다 막는 거다. 의미를 정확히 알고 써야 한다. "와일드카드는 양날의 검."