# Trailing Slash Middleware
이 미들웨어는 GET 요청 시 URL 끝에 붙는 슬래시(Trailing Slash, 꼬리 슬래시)를 알아서 처리해주는 녀석입니다. "URL 끝에 슬래시, 붙일까 말까? 고민 끝!"

`appendTrailingSlash`는 해당 경로로 요청이 왔는데 아무것도 못 찾았을 때 (404 Not Found), URL 끝에 슬래시를 붙여서 다시 한번 찾아보라고 리다이렉트 시킵니다. 반대로 `trimTrailingSlash`는 URL 끝에 슬래시가 붙어있으면 그걸 떼어버리고 리다이렉트합니다.

**가져오기 (Import)**

```typescript
import { Hono } from 'hono'
import {
  appendTrailingSlash,
  trimTrailingSlash,
} from 'hono/trailing-slash'
```

**사용법 (Usage)**

**예제 1: `/about/me` 같은 GET 요청을 `/about/me/`로 리다이렉트 (슬래시 붙이기)**

```typescript
import { Hono } from 'hono'
import { appendTrailingSlash } from 'hono/trailing-slash'

// Hono 앱을 strict: true 모드로 초기화하면 /path와 /path/를 다르게 취급합니다.
// 이 미들웨어는 그럴 때 유용하죠.
const app = new Hono({ strict: true })

app.use(appendTrailingSlash()) // 슬래시 붙이기 미들웨어 적용
app.get('/about/me/', (c) => c.text('With Trailing Slash')) // 슬래시가 붙은 경로로 핸들러 정의
```
만약 사용자가 `/about/me`로 접속하면, `appendTrailingSlash`가 `/about/me/`로 리다이렉트시켜서 위의 핸들러가 응답하게 됩니다.

**예제 2: `/about/me/` 같은 GET 요청을 `/about/me`로 리다이렉트 (슬래시 떼기)**

```typescript
import { Hono } from 'hono'
import { trimTrailingSlash } from 'hono/trailing-slash'

const app = new Hono({ strict: true })

app.use(trimTrailingSlash()) // 슬래시 떼기 미들웨어 적용
app.get('/about/me', (c) => c.text('Without Trailing Slash')) // 슬래시가 없는 경로로 핸들러 정의
```
만약 사용자가 `/about/me/`로 접속하면, `trimTrailingSlash`가 `/about/me`로 리다이렉트시켜서 위의 핸들러가 응답하게 됩니다.

**참고 (Note)**
이 미들웨어는 다음 두 가지 조건이 모두 충족될 때만 작동합니다:
1.  요청 메서드가 `GET`일 때
2.  원래 요청한 URL에 대한 응답 상태 코드가 `404 Not Found`일 때 (즉, "일단 찾아봤는데 없더라? 혹시 슬래시 때문인가?" 하는 상황)

---

**얘 뭐 하는 애냐?**
URL 끝에 슬래시(`/`)가 있든 없든, 한 가지 형태로 통일시켜주는 미들웨어다. 예를 들어 `example.com/products`랑 `example.com/products/`를 같은 페이지로 취급하고 싶을 때, 또는 둘 중 하나로 강제로 통일시키고 싶을 때 쓴다. "슬래시 하나로 갈팡질팡하는 URL, 내가 정리해드림."

**왜 쓰는데? (왜 이런 문제가 생기고, 왜 알려주는데?)**
1.  **URL 정규화 (Canonicalization)**: 검색 엔진은 `/path`와 `/path/`를 서로 다른 페이지로 인식할 수 있다. 이러면 페이지 랭크가 분산되거나 중복 콘텐츠로 찍혀서 SEO에 안 좋을 수 있다. 이 미들웨어로 한쪽으로 통일하면 "이게 진짜 주소다!"라고 알려주는 셈. "구글 봇아, 헷갈리지 말고 하나만 기억해!"
2.  **일관된 사용자 경험 및 라우팅**: 사용자가 실수로 슬래시를 빼먹거나 더 붙여도 알아서 올바른 페이지로 보내준다. 개발자도 슬래시 유무에 따라 라우트를 두 개씩 만들 필요가 없어진다.
3.  **서버 설정 단순화**: 웹 서버(Nginx, Apache 등)에서 복잡하게 리다이렉트 규칙을 설정할 필요 없이 애플리케이션 레벨에서 간단히 처리 가능하다.

**언제 불려 나오냐? (언제 이 문제가 발생하냐?)**
*   HTTP `GET` 요청일 때만 작동한다. (POST, PUT 같은 요청은 건드리지 않음)
*   Hono 라우터가 요청된 URL에 해당하는 핸들러를 찾지 못해서 "님, 그거 없음요 (404 Not Found)"라고 하려는 바로 그 순간! 이 미들웨어가 "잠깐! 혹시 슬래시 때문인가?" 하면서 끼어들어 슬래시를 붙이거나 떼고 다시 한번 시도(리다이렉트)한다.

**쓸 때 꿀팁 및 주의사항:**
*   **`strict: true`랑 찰떡궁합**: Hono 앱을 `const app = new Hono({ strict: true })`처럼 `strict` 모드로 만들면 `/path`와 `/path/`를 엄격하게 다른 경로로 취급한다. 이럴 때 이 미들웨어가 제 역할을 톡톡히 한다. `strict: false`(기본값)면 Hono가 알아서 둘 다 같은 걸로 봐주기 때문에 이 미들웨어의 존재감이 좀 줄어든다.
*   **붙일 거냐, 뗄 거냐?**: `appendTrailingSlash` (슬래시 붙이기)랑 `trimTrailingSlash` (슬래시 떼기) 중에 뭘 쓸지는 프로젝트의 URL 정책에 따라 결정하면 된다. 보통은 디렉터리를 의미하는 것처럼 슬래시를 붙이는(`append`) 쪽을 선호하는 경향이 있지만, 정답은 없다. 중요한 건 "한 놈만 패는" 일관성이다.
*   **SEO 친구, `rel="canonical"`**: URL을 한쪽으로 정규화했다면, HTML 헤더에 `<link rel="canonical" href="정규화된_URL">` 태그도 같이 써주면 검색 엔진 최적화에 더 좋다. "이게 찐 주소라고 도장 꽝!"
*   **리다이렉트는 공짜가 아니다**: 301 (영구 이동) 또는 308 (영구 이동, 메서드 유지) 리다이렉트가 발생한다. 아주 미미하지만 네트워크 왕복이 한 번 더 생기는 거라 성능에 민감하다면 고려해야 한다. 물론 대부분의 경우엔 "이 정도는 껌이지" 수준.
*   **미들웨어 순서도 중요**: 다른 라우팅 관련 미들웨어나 실제 경로 핸들러보다 먼저 실행되도록 `app.use()`로 등록하는 것이 일반적이다. 그래야 404가 나기 전에 개입할 수 있다.