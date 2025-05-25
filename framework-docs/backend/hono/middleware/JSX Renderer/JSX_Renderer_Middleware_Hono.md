# JSX_Renderer_Middleware_Hono

JSX 렌더러 미들웨어는 `c.render()` 함수로 JSX를 렌더링할 때, `c.setRenderer()`를 일일이 설정하는 귀찮음 없이도 레이아웃을 바로 적용할 수 있게 해주는 놈이다. 추가로, `useRequestContext()`를 쓰면 컴포넌트 안에서도 현재 요청 컨텍스트(Context)에 접근할 수 있다. "JSX 렌더링, 이제 좀 편하게 하자."

**가져오기 (Import)**

```typescript
import { Hono } from 'hono'
import { jsxRenderer, useRequestContext } from 'hono/jsx-renderer'
```

**사용법 (Usage)**

```typescript
const app = new Hono()

// '/page/*' 경로로 들어오는 요청에 대해 JSX 렌더링 시 기본 레이아웃 적용
app.get(
  '/page/*',
  jsxRenderer(({ children }) => { // children으로 실제 페이지 내용이 들어온다
    return (
      <html>
        <body>
          <header>메뉴</header>
          <div>{children}</div> {/* 여기에 페이지별 컨텐츠가 쏙! */}
        </body>
      </html>
    )
  })
)

// '/page/about' 경로 요청 시, 위에서 설정한 레이아웃 안에 <h1>About me!</h1>가 렌더링됨
app.get('/page/about', (c) => {
  return c.render(<h1>나에 대해서!</h1>)
})
```

**옵션 (Options)**

*   `docType` (선택 사항): `boolean | string`
    HTML 문서 시작 부분에 `<!DOCTYPE html>` 같은 DOCTYPE 선언을 넣고 싶지 않으면 `false`로 설정한다. 아니면 원하는 DOCTYPE 문자열을 직접 지정할 수도 있다. "DOCTYPE, 그거 은근히 신경 쓰이지?"

    ```typescript
    // DOCTYPE 빼고 싶을 때
    app.use(
      '*',
      jsxRenderer(
        ({ children }) => {
          return (
            <html>
              <body>{children}</body>
            </html>
          )
        },
        { docType: false } // DOCTYPE 꺼버리기
      )
    )

    // 특정 DOCTYPE 지정 (예: XHTML 1.1)
    app.use(
      '*',
      jsxRenderer(
        ({ children }) => {
          return (
            <html>
              <body>{children}</body>
            </html>
          )
        },
        {
          docType:
            '<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.1//EN" "http://www.w3.org/TR/xhtml11/DTD/xhtml11.dtd">',
        }
      )
    )
    ```

*   `stream` (선택 사항): `boolean | Record<string, string>`
    이 값을 `true`나 객체(`Record`) 형태로 주면, 응답을 스트리밍 방식으로 렌더링한다. "데이터 로딩 기다리다 지쳐? 스트리밍으로 바로바로 쏴주자!"

    ```typescript
    // 비동기 컴포넌트 예시 (1초 딜레이)
    const AsyncComponent = async () => {
      await new Promise((r) => setTimeout(r, 1000)) // 1초 대기
      return <div>반갑다!</div>
    }

    // 스트리밍 활성화
    app.get(
      '*',
      jsxRenderer(
        ({ children }) => {
          return (
            <html>
              <body>
                <h1>SSR 스트리밍</h1>
                {children}
              </body>
            </html>
          )
        },
        { stream: true } // 스트리밍 ON!
      )
    )

    // Suspense와 함께 비동기 컴포넌트 사용
    app.get('/', (c) => {
      return c.render(
        <Suspense fallback={<div>로딩 중...</div>}> {/* 로딩 중일 때 보여줄 내용 */}
          <AsyncComponent />
        </Suspense>
      )
    })
    ```

    `stream: true`로 설정하면 기본적으로 다음 HTTP 헤더들이 추가된다:

    ```json
    {
      "Transfer-Encoding": "chunked",
      "Content-Type": "text/html; charset=UTF-8",
      "Content-Encoding": "Identity"
    }
    ```

    이 헤더 값들은 `stream` 옵션에 객체를 전달해서 커스터마이징할 수 있다.

---

**얘 뭐 하는 애냐?**
Hono 프레임워크에서 JSX(리액트 문법 비슷한 거)로 웹페이지 찍어낼 때, 모든 페이지에 공통으로 들어가는 머리글, 바닥글 같은 레이아웃을 쉽게 씌워주고, 각 컴포넌트에서 현재 요청 정보(URL, 헤더 등)를 편하게 쓸 수 있도록 도와주는 도구다. "웹페이지 공장 돌릴 때 쓰는 만능 틀 + 작업 도구 세트랄까."

**왜 쓰는데? (왜 이런 기능이 필요하고, 왜 알려주는데?)**
1.  **DRY (Don't Repeat Yourself) 원칙 실현**: 페이지마다 `<html><head><body>...</body></head></html>` 복붙하는 노가다를 줄여준다. 레이아웃은 한 군데서 관리하고, 내용만 갈아 끼우는 방식. "복붙은 이제 그만! 개발자는 소중하니까."
2.  **개발 능률 향상**: `c.setRenderer()` 같은 복잡한 설정 없이 바로 `c.render()`로 JSX를 HTML로 변환 가능. `useRequestContext()` 쓰면 컴포넌트 안에서도 "지금 URL이 뭐지?" 같은 정보를 쉽게 알 수 있다.
3.  **사용자 경험 개선 (스트리밍)**: `stream: true` 옵션 켜면, 서버에서 HTML 만들어지는 대로 족족 브라우저에 보내준다. 특히 데이터 불러오는 데 시간 걸리는 컴포넌트가 있어도, 일단 화면 일부라도 빨리 보여줘서 사용자가 덜 지루하게 만든다. "로딩 바만 쳐다보는 시대는 갔다."

**언제 불려 나오냐? (언제 이 기능이 동작하냐?)**
특정 URL 경로로 요청이 들어와서 서버가 JSX를 HTML로 바꿔서 응답해야 할 때, 이 미들웨어가 먼저 나서서 설정된 레이아웃을 준비한다. 그 후 `c.render()`가 호출되면, 페이지별 JSX 컨텐츠가 이 레이아웃에 쏙 들어가서 최종 HTML이 완성된다. 스트리밍 옵션이 켜져 있으면, 비동기 컴포넌트들이 순차적으로 렌더링되면서 데이터가 조각조각 전송된다.

**쓸 때 꿀팁 및 주의사항:**
*   **레이아웃은 가볍게**: 모든 페이지에 적용되는 공용 레이아웃(`jsxRenderer`의 첫 번째 인자로 전달하는 함수)은 최대한 단순하고 빠르게 실행되도록 짜는 게 좋다. 여기가 무거우면 모든 페이지가 느려진다. "공용 시설은 심플 이즈 베스트."
*   **`useRequestContext`는 현명하게**: 컴포넌트에서 현재 요청(request) 객체의 정보가 꼭 필요할 때만 `useRequestContext`를 쓰자. 아무 데서나 막 쓰면 컴포넌트가 특정 상황에 너무 꽉 물려서 재사용성이 떨어질 수 있다. "만능 열쇠도 때와 장소를 가려서 써야."
*   **스트리밍, 과유불급**: 스트리밍이 초기 응답 속도를 개선해주긴 하지만, 모든 페이지에 무조건 좋은 건 아니다. 페이지가 아주 작거나, 모든 데이터가 한 번에 다 있어야 의미 있는 화면이라면 굳이 스트리밍할 필요 없다. 괜히 복잡도만 늘어날 수도. "좋은 약도 오남용하면 독."
*   **`docType` 확인**: HTML 표준을 잘 따르려면 `docType`이 제대로 설정됐는지 확인하자. 기본은 `<!DOCTYPE html>`이지만, 빼거나 다른 걸로 바꿔야 한다면 `docType` 옵션을 잊지 말 것. "기본 중의 기본, DOCTYPE!"
*   **에러 처리와 `Suspense`**: 스트리밍 환경, 특히 비동기 컴포넌트를 쓸 때는 로딩 상태나 에러 발생 시 대처가 중요하다. 리액트의 `Suspense` 같은 걸로 감싸서 로딩 UI나 에러 메시지를 적절히 보여줘야 사용자가 당황하지 않는다. "비동기는 언제 터질지 모른다. 대비만이 살길."