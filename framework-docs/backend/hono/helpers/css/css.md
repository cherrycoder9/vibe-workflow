`css` (실험적 기능)
`css` 템플릿 리터럴 안에 CSS를 바로 때려 박을 수 있어. 이 경우, `headerClass`가 `class` 속성값으로 들어가게 돼. CSS 내용을 실제로 찍어주는 `<Style />` 컴포넌트 까먹지 말고 꼭 넣어주고!

```typescript
app.get('/', (c) => {
  // css 템플릿 리터럴로 스타일 정의! 오렌지 배경에 흰 글씨, 패딩 1rem.
  const headerClass = css`
    background-color: orange;
    color: white;
    padding: 1rem;
  `
  return c.html(
    <html>
      <head>
        {/* 스타일 시트 찍어주는 <Style /> 컴포넌트 필수! */}
        <Style />
      </head>
      <body>
        {/* 생성된 headerClass를 h1 태그에 적용! */}
        <h1 class={headerClass}>Hello!</h1>
      </body>
    </html>
  )
})
```

`:hover` 같은 가상 클래스(pseudo-classes)는 중첩 선택자 `&`를 사용해서 스타일링할 수 있어.

```typescript
const buttonClass = css`
  background-color: #fff; // 평소엔 흰색 배경
  &:hover { // 마우스 올리면?
    background-color: red; // 빨간색 배경으로 변신!
  }
`
```

**확장하기 (Extending)**
클래스 이름을 직접 삽입해서 CSS 정의를 확장(상속)할 수 있어.

```typescript
// 기본 스타일 정의 (흰 글씨, 파란 배경)
const baseClass = css`
  color: white;
  background-color: blue;
`

// baseClass 스타일을 가져오고, 폰트 크기만 3rem으로 키움
const header1Class = css`
  ${baseClass} 
  font-size: 3rem;
`

// baseClass 스타일을 가져오고, 폰트 크기는 2rem으로
const header2Class = css`
  ${baseClass}
  font-size: 2rem;
`
```

게다가 `${baseClass} {}` 구문을 쓰면 클래스를 중첩시킬 수도 있어. (Sass/SCSS 좀 써봤으면 익숙한 문법이지?)

```typescript
const headerClass = css`
  color: white;
  background-color: blue;
`
// containerClass 안에 headerClass를 가진 요소가 있다면, 그 안의 h1 태그에 스타일 적용
const containerClass = css`
  ${headerClass} { 
    h1 {
      font-size: 3rem;
    }
  }
`
// 이렇게 렌더링하면...
return c.render(
  <div class={containerClass}>
    <header class={headerClass}> {/* 이 header 태그 안의 h1에 위 스타일이 먹힘 */}
      <h1>Hello!</h1>
    </header>
  </div>
)
```

**전역 스타일 (Global styles)**
`:-hono-global`이라는 가상 선택자를 쓰면 전역 스타일을 정의할 수 있어. 특정 클래스에 묶이지 않고 문서 전체에 영향을 주는 스타일이지.

```typescript
const globalClass = css`
  :-hono-global { // 이 안에 쓴 건 다 전역 스타일!
    html { // html 태그 전체에 폰트 적용
      font-family: Arial, Helvetica, sans-serif;
    }
  }
`
// globalClass를 아무데나 적용해도 전역 스타일은 잘 먹힘.
// (사실 globalClass 자체를 적용할 필요 없이 <Style />만 있으면 됨)
return c.render(
  <div class={globalClass}> {/* 이 div는 그냥 예시일 뿐, globalClass 적용 안해도 됨 */}
    <h1>Hello!</h1>
    <p>Today is a good day.</p>
  </div>
)
```

아니면 `<Style />` 컴포넌트 안에 `css` 리터럴을 직접 넣어서 전역 스타일을 작성할 수도 있어. 이게 더 직관적일 수도?

```typescript
export const renderer = jsxRenderer(({ children, title }) => {
  return (
    <html>
      <head>
        {/* <Style> 태그 안에 css 리터럴로 바로 전역 스타일 박아버리기! */}
        <Style>{css`
          html {
            font-family: Arial, Helvetica, sans-serif;
          }
        `}</Style>
        <title>{title}</title>
      </head>
      <body>
        <div>{children}</div>
      </body>
    </html>
  )
})
```

---

**얘 뭐 하는 애냐?**
`hono/css`의 `css` 템플릿 리터럴은 자바스크립트 코드 안에서 CSS를 문자열처럼 쉽게 작성하고, 그걸 동적으로 HTML 요소에 클래스로 적용하거나, 전역 스타일로 문서 전체에 영향을 줄 수 있게 해주는 도구야. "CSS 파일 왔다 갔다 하기 귀찮은데, 그냥 여기서 다 해결하자!"를 실현시켜주는 거지.

**왜 쓰는데?**
1.  **간편한 스타일 정의 및 적용**: CSS 코드를 변수에 담고, 그 변수를 HTML 태그의 `class` 속성에 쏙 넣어주면 끝. 직관적이고 쉽지.
2.  **스타일 재사용 및 확장 용이**: 만들어둔 스타일(클래스)을 다른 스타일 정의에 `${baseClass}`처럼 끼워 넣어 재사용하거나, 특정 조건 하에서만 스타일을 덧붙이는 확장이 편해. "레고 블록처럼 스타일 조립하기!"
3.  **가상 클래스 및 중첩 지원**: `:hover` 같은 가상 클래스나, Sass/SCSS처럼 스타일 규칙을 중첩해서 작성하는 게 가능해서 복잡한 CSS 구조도 나름 깔끔하게 표현할 수 있어.
4.  **전역 스타일 관리**: 특정 컴포넌트에 국한되지 않고 웹사이트 전체의 기본 스타일(폰트, 배경색 등)을 `:-hono-global`이나 `<Style>{css`...`}</Style>` 구문으로 한 곳에서 정의하고 관리할 수 있어.

**언제 불려 나오냐?**
Hono의 JSX 렌더링 기능을 사용하면서, 동적인 스타일링이 필요하거나, 컴포넌트와 스타일을 한 파일에서 관리하고 싶을 때 주로 사용돼. `<Style />` 컴포넌트는 항상 세트로 따라다니면서 실제 CSS 코드를 HTML에 주입하는 역할을 해.

**쓸 때 꿀팁 및 주의사항:**
*   **`<Style />`은 생명줄**: 아무리 `css`로 멋진 스타일을 정의해도, `<Style />` 컴포넌트를 JSX 어딘가(주로 `<head>`)에 렌더링하지 않으면 브라우저는 아무것도 몰라. "스타일 정의는 설계도, `<Style />`은 시공사!"
*   **"실험적" 딱지 기억하기**: "Experimental"이라고 명시된 만큼, 기능이 변경되거나 예기치 않은 동작이 있을 수 있다는 점은 감안해야 해. 중요한 프로덕션 환경에선 좀 더 검증된 CSS-in-JS 라이브러리(Styled Components, Emotion 등)를 고려하는 게 안전빵일 수 있어. "새 장난감은 재밌지만, 가끔 고장 난다."
*   **확장 문법 `${baseClass}` vs `${baseClass} {}`**:
    *   `${baseClass}`: `baseClass`의 모든 스타일 규칙을 그대로 가져와서 현재 정의에 합치는 거야. (믹스인 비슷)
    *   `${baseClass} {}`: `containerClass` 안에 `headerClass`를 가진 자손 요소가 있을 때, 그 자손 요소 내부의 특정 태그(예: `h1`)에 스타일을 적용하는 중첩 규칙을 만들어. 선택자가 더 구체적으로 만들어지는 거지. 헷갈리면 직접 찍어보고 눈으로 확인하는 게 최고.
*   **전역 스타일 남용은 금물**: `:-hono-global`이나 `<Style>{css`...`}</Style>`로 전역 스타일을 너무 많이 만들면, 나중에 스타일 충돌 나거나 어디서 적용된 건지 추적하기 힘들어질 수 있어. 꼭 필요한 최소한의 전역 스타일만 정의하고, 나머지는 컴포넌트 범위로 한정하는 게 좋아. "공용 물품은 꼭 필요할 때만!"
*   **성능**: 모든 스타일을 자바스크립트로 처리하면 런타임에 약간의 부하가 생길 수 있어. 아주 크고 복잡한 애플리케이션이라면 정적 CSS 파일과 적절히 섞어 쓰는 전략도 고민해봐야 해.
*   **디버깅**: 브라우저 개발자 도구에서 클래스 이름이 `css-xxxxxx`처럼 자동 생성된 형태로 보일 거야. 어떤 `css` 호출에서 만들어진 건지 바로 알기 어려울 수 있으니, 코드 구조를 깔끔하게 유지하는 게 중요해.