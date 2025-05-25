# Server Timing Middleware
서버 타이밍 미들웨어는 응답 헤더에 성능 측정 지표를 제공합니다. "우리 서버 일 얼마나 빨리 하는지 궁금해? 헤더만 까봐!"

**정보**

참고: Cloudflare Workers 환경에서는 타이머가 마지막 I/O(입출력) 시간만 보여주기 때문에 타이머로 찍히는 측정치가 정확하지 않을 수 있습니다. "엣지에서는 시간 개념이 좀 다를 수 있다, 이 말이야."

**가져오기 (Import)**
npm

```typescript
import { Hono } from 'hono'
import { timing, setMetric, startTime, endTime } from 'hono/timing'
import type { TimingVariables } from 'hono/timing'
```

**사용법 (Usage)**

`c.get('metric')` 타입을 제대로 추론하려면 아래처럼 변수 타입을 지정해주는 게 좋습니다:
```typescript
type Variables = TimingVariables // 타이밍 관련 변수 타입을 가져오고

const app = new Hono<{ Variables: Variables }>() // Hono 앱 초기화할 때 지정

// 라우터에 미들웨어를 추가합니다
app.use(timing());

app.get('/', async (c) => {

  // 커스텀 측정 지표 추가 (이름, 값)
  setMetric(c, 'region', 'europe-west3')

  // 시간 정보를 포함한 커스텀 측정 지표 추가 (이름, 소요 시간(ms), 설명)
  setMetric(c, 'custom', 23.8, 'My custom Metric') // 단위는 밀리초!

  // 새 타이머 시작 (이름)
  startTime(c, 'db');
  const data = await db.findMany(...); // DB 작업 같은 거 하고

  // 타이머 종료 (이름)
  endTime(c, 'db');

  return c.json({ response: data });
});
```

**조건부 활성화 (Conditionally enabled)**

특정 조건에서만 이 미들웨어를 쓰고 싶을 수도 있잖아? 그럴 땐 이렇게.
```typescript
const app = new Hono()

app.use(
  '*', // 모든 경로에 대해
  timing({
    // c: 요청 컨텍스트 (Context of the request)
    // POST 요청일 때만 활성화
    enabled: (c) => c.req.method === 'POST',
  })
)
```

**결과 (Result)**
이 미들웨어를 사용하면 HTTP 응답 헤더에 `Server-Timing` 항목이 추가됩니다. 예를 들면 이런 식이지:
`Server-Timing: db;dur=53, custom;dur=23.8;desc="My custom Metric", total;dur=123.4`
브라우저 개발자 도구의 네트워크 탭에서 이 정보를 활용해 성능을 분석할 수 있습니다. "숫자로 보여주니 빼도 박도 못 하지."

**옵션 (Options)**

*   `total` (선택 사항): `boolean`
    전체 응답 시간 표시 여부를 결정합니다. 기본값은 `true` (표시함).
*   `enabled` (선택 사항): `boolean | (c: Context) => boolean`
    헤더에 타이밍 정보를 추가할지 말지를 결정합니다. `true`면 항상 추가, `false`면 추가 안 함. 함수를 넘기면 요청 컨텍스트(`c`)를 받아 동적으로 결정할 수도 있습니다. 기본값은 `true`.
*   `totalDescription` (선택 사항): `string`
    전체 응답 시간에 대한 설명을 지정합니다. 기본값은 "Total Response Time".
*   `autoEnd` (선택 사항): `boolean`
    `startTime()`으로 시작된 타이머를 요청이 끝날 때 자동으로 종료할지 여부를 결정합니다. 이걸 `false`로 해놓고 `endTime()` 호출을 까먹으면 해당 타이머는 결과에 안 나옵니다. "시작했으면 끝을 봐야지, 안 그래?" 기본값은 `true`.
*   `crossOrigin` (선택 사항): `boolean | string | (c: Context) => boolean | string`
    이 타이밍 헤더를 어떤 출처(origin)에서 읽을 수 있게 할지 지정합니다.
    *   `false` (기본값): 현재 출처에서만 읽을 수 있습니다. "내 정보는 나만 본다."
    *   `true`: 모든 출처에서 읽을 수 있습니다. "월드 와이드 공개!"
    *   문자열: 지정된 도메인(들)에서만 읽을 수 있습니다. 여러 도메인은 쉼표(`,`)로 구분합니다. 예: `'example.com,another.com'`
    *   함수: 요청 컨텍스트를 받아 동적으로 결정할 수도 있습니다.
    자세한 내용은 관련 문서를 참고하세요.

---

**얘 뭐 하는 애냐?**
Hono 프레임워크에서 서버가 요청 하나 처리하는 데 얼마나 빌빌댔는지, 아니면 얼마나 날아다녔는지 그 시간을 재서 HTTP 응답 헤더(`Server-Timing`)에 딱! 찍어주는 미들웨어다. "내 서버, 일 좀 하나?" 하고 감시하는 CCTV 같은 거지.

**왜 쓰는데? (왜 이런 기능이 있고, 왜 알려주는데?)**
1.  **성능 병목 찾기**: 개발자가 브라우저 개발자 도구에서 "어디서 시간을 다 잡아먹는 거야?" 하고 원인 분석할 때 아주 유용하다. DB가 문제인지, 외부 API 호출이 느린 건지, 아니면 그냥 내 코드가 발적화인지 금방 알 수 있다. "범인은 이 안에 있어!"
2.  **최적화 길잡이**: 어디가 느린지 알아야 최적화를 하든 말든 할 거 아니냐. 이놈이 그 길을 밝혀주는 등대 같은 역할을 한다.
3.  **실시간 피드백**: 개발 중에 바로바로 성능 변화를 체크할 수 있어서 좋다. "코드 한 줄 바꿨는데 빨라졌네? 개꿀!"

**언제 불려 나오냐? (언제 이 정보가 생성/추가되냐?)**
`timing()` 미들웨어를 앱에 등록(`app.use(timing())`)하면, 그 앱으로 들어오는 모든 HTTP 요청에 대해 응답을 보내기 직전에 작동한다. `startTime()`, `endTime()` 함수를 코드 중간중간에 심어두면 특정 구간의 실행 시간도 콕 집어 측정해서 헤더에 같이 넣어준다. `enabled` 옵션으로 특정 조건의 요청에만 작동하게 만들 수도 있다.

**쓸 때 꿀팁 및 주의사항:**
*   **Cloudflare Workers에선 반만 믿어라**: 본문에도 있지만, Cloudflare Workers 같은 특정 환경에선 타이머가 I/O 기준으로만 찍혀서 실제 CPU 작업 시간 같은 건 정확하지 않을 수 있다. "환경 따라 시계가 다르게 간다니, 이게 무슨 평행 우주냐."
*   **민감 정보는 금지**: `setMetric`으로 커스텀 지표 만들 때 이름이나 설명에 DB 이름, 내부 경로, 비밀 키 같은 거 절대 넣지 마라. "개발자 도구에 다 뜨는데, 해커한테 정보 떠먹여 줄 일 있냐?"
*   **`crossOrigin` 똑바로 알자**: 기본은 `false`라 같은 도메인에서만 보인다. 만약 다른 도메인의 모니터링 툴에서 이 정보를 봐야 한다면 `true`로 하거나 특정 도메인을 지정해줘야 한다. 보안 생각하면 꼭 필요한 놈들한테만 열어주는 게 맞다.
*   **`autoEnd`는 편리하지만**: `startTime()`만 찍고 `endTime()`을 까먹어도 `autoEnd: true`(기본값)면 알아서 요청 끝날 때 정리해주니 편하긴 하다. 근데 진짜 중요한 측정 구간은 명시적으로 `endTime()`을 박아주는 게 정신 건강에 이롭다. "자동도 좋지만, 수동으로 확인 사살!"
*   **프로덕션 환경에선 신중히**: 너무 많은 타이머를 걸거나, 너무 자세한 정보를 계속 보내면 아주 약간이라도 서버에 부담이 될 수 있다. 운영 환경에선 꼭 필요한 핵심 지표 위주로, 샘플링해서 쓰거나 하는 게 좋다. "성능 측정하다 성능 조지지 말자."