# Hono_Advanced_JSX_Features

**중첩 레이아웃 (Nested Layouts)**
`Layout` 컴포넌트를 쓰면 레이아웃을 여러 겹으로 쌓을 수 있다. "마트료시카 인형처럼 말이지."

```typescript
// 기본 레이아웃: 모든 페이지의 기본 틀 (html, body)
app.use(
  jsxRenderer(({ children }) => {
    return (
      <html>
        <body>{children}</body>
      </html>
    )
  })
)

const blog = new Hono()
// 블로그 섹션 전용 레이아웃: 기본 레이아웃 위에 블로그 메뉴 추가
blog.use(
  jsxRenderer(({ children, Layout }) => {
    return (
      <Layout> {/* 부모 레이아웃을 여기에 쏙 */}
        <nav>블로그 메뉴</nav>
        <div>{children}</div> {/* 실제 페이지 내용 */}
      </Layout>
    )
  })
)

app.route('/blog', blog) // '/blog' 경로에 블로그 레이아웃 적용
```

---

**얘 뭐 하는 애냐?**
웹페이지 만들 때 공통적인 부분(헤더, 푸터, 내비게이션 바 같은 거)은 한 번만 만들고, 특정 페이지나 섹션에만 추가적인 레이아웃 요소를 덧씌우고 싶을 때 쓰는 기능이다. "기본 뼈대 위에 옷 겹쳐 입는다고 생각하면 쉽다."

**왜 쓰는데?**
1.  **코드 재사용**: 모든 페이지에 똑같이 들어가는 HTML 구조를 복붙할 필요 없이, `Layout` 컴포넌트로 한 방에 해결. "Ctrl+C, Ctrl+V는 이제 그만."
2.  **유지보수 용이**: 공통 레이아웃 수정하면 모든 페이지에 알아서 반영되니, 나중에 고치기도 편하다.
3.  **구조적 설계**: 페이지 구조를 논리적으로 분리해서 관리할 수 있다. 기본 틀, 섹션별 틀, 실제 내용... 이런 식으로.

**언제 불려 나오냐?**
*   사이트 전체에 적용되는 기본 레이아웃(예: `<html>`, `<body>`, 기본 CSS/JS 링크)을 설정할 때.
*   특정 경로 그룹(예: `/blog/*`, `/admin/*`)에만 다른 메뉴나 사이드바가 포함된 하위 레이아웃을 적용하고 싶을 때.

**쓸 때 꿀팁 및 주의사항:**
*   **`jsxRenderer` 필수**: 이 기능 쓰려면 `jsxRenderer` 미들웨어가 먼저 등록되어 있어야 한다. 얘가 없으면 `Layout`이고 뭐고 다 그림의 떡.
*   **`children`과 `Layout` 프롭스**: 하위 레이아웃의 `jsxRenderer` 콜백 함수는 `children` (실제 페이지 내용)과 `Layout` (상위 레이아웃 컴포넌트)을 인자로 받는다. 이걸 잘 써먹어야 중첩이 된다.
*   **과도한 중첩은 금물**: 너무 여러 겹으로 레이아웃을 쌓으면 구조 파악하기 힘들어지고 성능에도 좋을 거 없다. "양파도 아니고... 적당히 하자."
*   **데이터 전달**: 레이아웃 컴포넌트에 데이터를 넘기고 싶으면 `ContextRenderer` 확장 같은 다른 방법을 같이 써야 할 수 있다.

---

**`useRequestContext()`**
`useRequestContext()`는 `Context`의 인스턴스를 반환한다. "쉽게 말해, 현재 요청에 대한 모든 정보가 담긴 보따리 같은 거다."

```typescript
import { useRequestContext, jsxRenderer } from 'hono/jsx-renderer'

const app = new Hono()
app.use(jsxRenderer()) // jsxRenderer는 기본으로 깔고 가자

// 현재 요청 URL을 굵게 표시하는 컴포넌트
const RequestUrlBadge: FC = () => {
  const c = useRequestContext() // 여기서 Context 객체를 꺼내 쓴다
  return <b>{c.req.url}</b> // 요청 객체(c.req)에서 URL 정보 획득!
}

app.get('/page/info', (c) => {
  return c.render(
    <div>
      당신이 접속한 곳: <RequestUrlBadge />
    </div>
  )
})
```

**경고 (WARNING)**
Deno의 JSX 사전 컴파일(precompile) 옵션을 사용하면 `useRequestContext()`를 쓸 수 없다. 대신 `react-jsx`를 사용해야 한다:

```json
{
  "compilerOptions": {
    "jsx": "react-jsx",
    "jsxImportSource": "hono/jsx"
  }
}
```

---

**얘 뭐 하는 애냐?**
Hono 프레임워크의 JSX 컴포넌트 안에서 현재 HTTP 요청(request)과 관련된 정보(예: URL, 헤더, 메서드 등)나 컨텍스트(Context) 객체 자체에 접근하고 싶을 때 쓰는 훅(hook)이다. "컴포넌트 안에서 '지금 상황이 어떻게 돌아가고 있나?' 궁금할 때 쓰는 비상벨."

**왜 쓰는데?**
1.  **동적 렌더링**: 요청 URL이나 사용자 정보에 따라 컴포넌트 내용을 다르게 보여주고 싶을 때.
2.  **컨텍스트 데이터 활용**: `Context` 객체에 담긴 다른 미들웨어가 설정한 값이나, Hono의 기본 기능(쿠키 설정, 리다이렉트 등)을 JSX 컴포넌트 내에서 간접적으로 사용해야 할 때.

**언제 불려 나오냐?**
JSX 컴포넌트 함수 내부에서 호출된다. `jsxRenderer`를 통해 렌더링되는 컴포넌트 안에서만 의미가 있다.

**쓸 때 꿀팁 및 주의사항:**
*   **`jsxRenderer` 선행 필수**: `app.use(jsxRenderer())`가 먼저 설정되어 있어야 `useRequestContext()`가 제대로 작동한다. "밥부터 먹고 국 떠먹는 순서."
*   **Deno 사용자 주의**: Deno 환경에서 JSX를 미리 컴파일하는 옵션(`"jsx": "react"`, `"jsxFactory": "h"`, `"jsxFragmentFactory": "Fragment"` 등)을 쓰면 `useRequestContext`가 작동안한다. `tsconfig.json` (또는 `deno.json`)에서 `"jsx": "react-jsx"` 와 `"jsxImportSource": "hono/jsx"` 로 바꿔야 한다. "Deno 유저들, 이 부분에서 삽질 금지."
*   **반환 값은 `Context` 객체**: `const c = useRequestContext()` 이렇게 받으면 `c`가 바로 Hono의 `Context` 객체다. `c.req`, `c.res`, `c.json()` 등등 다 쓸 수 있다.

---

**`ContextRenderer` 확장하기 (Extending ContextRenderer)**
아래처럼 `ContextRenderer`를 정의하면, 렌더러에 추가적인 데이터를 넘겨줄 수 있다. 예를 들어 페이지마다 `<head>` 태그 안의 내용을 바꾸고 싶을 때 아주 유용하다. "기본 옵션에 나만의 특제 소스 추가하는 느낌."

```typescript
// hono 모듈의 ContextRenderer 인터페이스에 우리만의 약속을 추가
declare module 'hono' {
  interface ContextRenderer {
    (
      content: string | Promise<string>, // 원래 있던 렌더링할 내용
      props: { title: string } // 우리가 추가할 props! 여기선 title
    ): Response
  }
}

const app = new Hono()

// 모든 /page/* 경로에 적용될 jsxRenderer 설정
app.get(
  '/page/*',
  jsxRenderer(({ children, title }) => { // 이제 title을 props로 받을 수 있다!
    return (
      <html>
        <head>
          <title>{title}</title> {/* 받은 title을 HTML title 태그에 쏙 */}
        </head>
        <body>
          <header>메뉴</header>
          <div>{children}</div>
        </body>
      </html>
    )
  })
)

// '/page/favorites' 경로 핸들러
app.get('/page/favorites', (c) => {
  // c.render() 할 때 두 번째 인자로 { title: '내가 좋아하는 것들' } 을 넘겨준다
  return c.render(
    <div>
      <ul>
        <li>초밥 먹기</li>
        <li>야구 경기 보기</li>
      </ul>
    </div>,
    {
      title: '내가 좋아하는 것들', // 이 값이 위 jsxRenderer의 title로 전달된다
    }
  )
})
```

---

**얘 뭐 하는 애냐?**
Hono에서 JSX 페이지를 렌더링할 때, 각 페이지마다 다른 정보(예: HTML `<title>`, 메타 태그 내용 등)를 레이아웃 컴포넌트에 전달하고 싶을 때 쓰는 방법이다. 타입스크립트를 이용해서 Hono의 기본 `ContextRenderer` 인터페이스에 우리가 원하는 추가 속성을 "선언적으로" 확장하는 거다. "기본 배달 음식에 '치즈 추가요!' 외치는 격."

**왜 쓰는데?**
1.  **동적 `<head>` 관리**: 페이지마다 다른 제목, 설명, 키워드 등을 `<head>` 태그에 넣어서 SEO(검색 엔진 최적화)나 사용자 경험을 향상시키려고.
2.  **레이아웃에 데이터 주입**: `c.render()` 호출 시점에 특정 데이터를 레이아웃 컴포넌트로 넘겨서, 레이아웃이 그 데이터에 따라 다르게 보이거나 작동하도록 만들고 싶을 때.

**언제 불려 나오냐?**
1.  **타입 정의**: `declare module 'hono'` 블록은 타입스크립트 컴파일 시점에 읽혀서 `ContextRenderer`의 타입을 확장한다.
2.  **`jsxRenderer` 설정**: 확장된 props(예: `title`)를 받도록 `jsxRenderer`의 콜백 함수를 수정한다.
3.  **`c.render()` 호출**: 라우트 핸들러에서 `c.render(내용, { title: '페이지 제목' })`처럼 두 번째 인자로 확장된 props 객체를 전달할 때, 그 값이 `jsxRenderer`로 전달된다.

**쓸 때 꿀팁 및 주의사항:**
*   **타입스크립트 필수**: `declare module`은 타입스크립트 문법이라, 자바스크립트만 쓸 때는 이 방식이 아니라 다른 방식으로 데이터를 전달해야 한다. (예: `c.set()` 쓰고 `jsxRenderer`에서 `c.get()` 하기)
*   **인터페이스 일치**: `declare module`에서 정의한 props 이름과 타입, 그리고 `jsxRenderer` 콜백에서 받는 props 이름과 타입, `c.render()`에서 전달하는 객체의 키와 값이 정확히 일치해야 한다. 안 그러면 타입 에러 파티. "약속은 지키라고 있는 거다."
*   **전역적 영향**: `declare module`로 모듈 선언을 확장하면 프로젝트 전체에 영향을 준다. 특정 부분에서만 다르게 하고 싶다면 좀 더 고민이 필요할 수 있다.
*   **`Promise<string>` 지원**: `content` 인자는 `string` 뿐만 아니라 `Promise<string>`도 될 수 있어서, 비동기적으로 내용을 가져와 렌더링하는 것도 가능하다. "비동기 처리, 요즘 기본 소양."