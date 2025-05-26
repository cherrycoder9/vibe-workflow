# Method Override Middleware Explained
이 미들웨어는 폼, 헤더, 또는 쿼리 값에 따라 실제 요청의 메서드와 다른, 지정된 메서드의 핸들러를 실행하고 그 응답을 반환합니다. 한마디로 "메서드 바꿔치기" 전문가죠.

**가져오기 (Import)**

```typescript
import { Hono } from 'hono'
import { methodOverride } from 'hono/method-override'
```

**사용법 (Usage)**

```typescript
const app = new Hono()

// 옵션을 지정하지 않으면 폼의 `_method` 값 (예: DELETE)을 메서드로 사용합니다.
app.use('/posts', methodOverride({ app }))

app.delete('/posts', (c) => {
  // ....
})
```

**예시 (For example)**
HTML 폼은 DELETE 메서드를 보낼 수 없으므로, `_method`라는 이름의 속성에 `DELETE` 값을 넣어 보내면 됩니다. 그러면 `app.delete()` 핸들러가 실행될 겁니다. "HTML 폼의 한계를 극복하는 꼼수랄까?"

HTML 폼:

```html
<form action="/posts" method="POST">
  <input type="hidden" name="_method" value="DELETE" />
  <input type="text" name="id" />
</form>
```

애플리케이션:

```typescript
import { methodOverride } from 'hono/method-override'

const app = new Hono()
app.use('/posts', methodOverride({ app }))

app.delete('/posts', () => {
  // ...
})
```

기본값을 변경하거나 헤더 값 및 쿼리 값을 사용할 수도 있습니다:

```typescript
// form 옵션으로 폼 필드 이름 변경
app.use('/posts', methodOverride({ app, form: '_custom_name' }))

// header 옵션으로 커스텀 헤더 이름 지정
app.use(
  '/posts',
  methodOverride({ app, header: 'X-METHOD-OVERRIDE' })
)

// query 옵션으로 쿼리 파라미터 이름 지정
app.use('/posts', methodOverride({ app, query: '_method' }))
```

**옵션 (Options)**
*   `app`: Hono (필수)
    애플리케이션에서 사용되는 Hono 인스턴스입니다. "얘 없으면 미들웨어가 누굴 위해 일해야 할지 몰라요."
*   `form`: string (선택 사항)
    메서드 이름을 값으로 포함하는 폼 필드의 키(key)입니다. 기본값은 `_method`입니다.
*   `header`: string (선택 사항)
    메서드 이름을 값으로 포함하는 헤더의 이름입니다. (예: `X-METHOD-OVERRIDE`)
*   `query`: string (선택 사항)
    메서드 이름을 값으로 포함하는 쿼리 파라미터의 키(key)입니다. (예: `_method`)

---

**얘 뭐 하는 애냐?**
HTML 폼처럼 GET이나 POST 메서드밖에 못 보내는 환경에서, 실제로는 PUT이나 DELETE 같은 다른 HTTP 메서드 요청인 것처럼 서버에 알려주는 역할을 해. "겉은 POST, 속은 DELETE? 문제없어!" 요청의 숨겨진 의도를 파악해서 Hono 앱이 올바른 핸들러로 연결하도록 돕는 거지.

**왜 쓰는데? (어떤 목적으로 사용되나?)**
1.  **HTML 폼의 한계 극복:** 이게 핵심이야. HTML `<form>` 태그는 `method` 속성에 `GET`이랑 `POST`만 쓸 수 있어. RESTful API를 제대로 구현하려면 `PUT`, `DELETE`, `PATCH` 같은 다양한 메서드가 필요한데, 이 미들웨어가 그 갭을 메꿔주는 거지. "HTML아, 넌 POST만 보내. 나머진 내가 알아서 할게."
2.  **클라이언트/프록시 제약 우회:** 일부 구형 브라우저나 특정 프록시 환경에서 특정 HTTP 메서드 전송을 막는 경우가 있는데, 이럴 때도 유용해.

**언제 불려 나오냐? (언제 호출되나?)**
클라이언트가 서버로 요청을 보낼 때, 이 미DL웨어가 설정된 라우트(예: `/posts`)로 요청이 들어오면 가장 먼저 실행돼. 요청 안에 `_method` (또는 `form`, `header`, `query` 옵션으로 지정한 다른 이름) 필드에 `PUT`, `DELETE` 같은 값이 있는지 확인하고, 있으면 요청 메서드를 그 값으로 바꿔치기한 다음 Hono 앱 내부의 라우터로 넘겨.

**쓸 때 꿀팁 및 주의사항:**
*   **Hono 앱 인스턴스 전달 필수:** `app` 옵션에 Hono 애플리케이션 인스턴스를 꼭 넘겨줘야 해. 안 그러면 어느 앱의 요청을 바꿔치기할지 모르니까. "주인님을 지정해줘야 제대로 일한다."
*   **키 이름 일관성:** `_method`가 기본이지만, `form`, `header`, `query` 옵션으로 사용자 정의 이름을 쓸 수 있어. 프로젝트 내에서 어떤 키를 쓸 건지 정했으면 일관되게 사용해야 혼란이 없어. "여기선 `_method`, 저기선 `_delete_please`? 정신 사나워."
*   **보안 생각은 하고 쓰자:** `header`나 `query` 옵션을 사용하면 URL이나 헤더에 메서드 정보가 노출될 수 있어. 민감한 작업이라면 폼 필드를 쓰는 게 그나마 좀 더 낫지. "아무나 요청 메서드를 주무르게 두면 곤란하다."
*   **최신 기술과 비교:** 요즘엔 프론트엔드에서 JavaScript(fetch, axios 등)로 직접 다양한 HTTP 메서드를 보내는 게 일반적이라, 이 미들웨어의 필요성이 예전만큼 크진 않아. 그래도 레거시 시스템이나 특정 상황에선 여전히 유용할 수 있어. "옛날 방식이지만, 아직 쓸모는 있다."
*   **HTTP 메서드 오버라이드는 신중하게:** 꼭 필요한 경우가 아니라면 남용하지 않는 게 좋아. 요청의 실제 의도와 표현이 달라지면 디버깅이 복잡해질 수도 있거든. "마법은 꼭 필요할 때만!"