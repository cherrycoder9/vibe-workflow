`render()` / `setRenderer()`
커스텀 미들웨어 안에서 `c.setRenderer()`를 사용해 레이아웃을 설정할 수 있습니다.

```javascript
app.use(async (c, next) => {
  c.setRenderer((content) => { // content는 c.render()의 첫 번째 인자로 전달된 값
    return c.html(
      <html>
        <body>
          <p>{content}</p>
        </body>
      </html>
    )
  })
  await next()
})
```

그런 다음, 이 레이아웃 안에서 응답을 만들 때 `c.render()`를 활용할 수 있습니다.

```javascript
app.get('/', (c) => {
  return c.render('헬로!') // 이 '헬로!'가 위 setRenderer의 content로 들어감
})
```

결과물은 이렇게 나옵니다:

```html
<html>
  <body>
    <p>헬로!</p>
  </body>
</html>
```

추가로, 이 기능은 인자를 커스터마이징할 수 있는 유연성을 제공합니다. 타입 안전성을 확보하기 위해, 타입은 이렇게 정의할 수 있습니다:

```typescript
declare module 'hono' {
  interface ContextRenderer {
    (
      content: string | Promise<string>, // 첫 번째 인자: 내용물
      head: { title: string }            // 두 번째 인자: 헤드 정보 (예: 제목)
    ): Response | Promise<Response>      // 반환 타입: Response 또는 Promise<Response>
  }
}
```

이렇게 사용하는 예시입니다:

```javascript
// '/pages/*' 경로에만 적용되는 미들웨어
app.use('/pages/*', async (c, next) => {
  c.setRenderer((content, head) => { // content와 head 인자를 받음
    return c.html(
      <html>
        <head>
          <title>{head.title}</title>
        </head>
        <body>
          <header>{head.title}</header>
          <p>{content}</p>
        </body>
      </html>
    )
  })
  await next()
})

app.get('/pages/my-favorite', (c) => {
  return c.render(<p>라면과 스시</p>, { // 첫 번째 인자가 content, 두 번째 객체가 head로 전달
    title: '내가 제일 좋아하는 것',
  })
})

app.get('/pages/my-hobbies', (c) => {
  return c.render(<p>야구 보기</p>, {
    title: '내 취미',
  })
})
```

---

**얘네 뭐 하는 애들이냐?**
`setRenderer()`와 `render()`는 Hono에서 HTML 렌더링할 때 "공통 틀(레이아웃)"을 쉽게 적용하고 재사용할 수 있게 해주는 짝꿍 기능입니다.
*   `c.setRenderer()`: 미들웨어에서 "앞으로 이 앱(또는 이 경로)에서 HTML 만들 땐 이 틀을 기본으로 써!" 하고 레이아웃 함수를 등록하는 놈입니다. 일종의 HTML 청사진을 미리 그려두는 거죠.
*   `c.render()`: 실제 라우트 핸들러에서 "아까 그 틀에다가 이 내용물 좀 채워서 손님한테 보여줘!" 하고 요청하는 놈입니다. 내용물(content)과 필요하면 추가 데이터(예: 페이지 제목)를 레이아웃 함수에 전달해서 최종 HTML을 완성시킵니다.

**왜 쓰는데?**
1.  **코드 중복 박살**: 웹사이트 만들다 보면 헤더, 푸터, 내비게이션 바처럼 모든 페이지에 똑같이 들어가는 부분이 있죠? 이런 걸 매번 복붙하면 나중에 수정할 때 죽어납니다. `setRenderer`로 공통 레이아웃 딱 정해두면, 각 페이지에서는 내용물만 갈아 끼우면 되니 아주 편해집니다. "한번 만들고 계속 우려먹자!"
2.  **유지보수 용이성**: 공통 레이아웃 디자인 바꾸고 싶을 때, `setRenderer`에 등록된 함수 하나만 고치면 그 레이아웃 쓰는 모든 페이지에 일괄 적용됩니다. "일해라 한 놈만!"
3.  **구조화된 뷰 관리**: HTML 구조를 레이아웃과 내용물로 분리해서 생각할 수 있으니 코드가 더 깔끔해지고 이해하기 쉬워집니다. 서버 사이드 렌더링(SSR)할 때 특히 유용하죠.
4.  **타입스크립트 지원으로 안전성 UP**: `declare module 'hono'`를 통해 `ContextRenderer` 인터페이스를 확장하면, `c.render()`에 전달하는 인자들의 타입을 미리 정의해서 개발 중에 실수할 확률을 줄여줍니다. "타입 틀리면 바로 딱밤!"

**언제 불려 나오냐?**
*   `c.setRenderer(레이아웃_함수)`: 주로 앱 전체 또는 특정 경로 그룹에 적용할 미들웨어(`app.use(...)`) 안에서 호출됩니다. 앱이 시작되거나 해당 미들웨어가 처리될 때 레이아웃 함수가 컨텍스트(`c`)에 등록되는 거죠.
*   `c.render(내용물, 추가_데이터)`: 실제 HTTP 요청을 처리하는 라우트 핸들러(`app.get(...)`, `app.post(...)` 등) 안에서 최종적으로 HTML 응답을 생성할 때 호출됩니다.

**쓸 때 꿀팁 및 주의사항:**
*   **레이아웃 함수는 `(content, ...args) => Response | Promise<Response>` 형태**: `setRenderer`에 등록하는 함수는 첫 번째 인자로 `c.render`에서 넘겨준 내용물(content)을 받고, 그 뒤로 추가 인자들을 받을 수 있습니다. 그리고 반드시 `c.html(...)` 등을 사용해 `Response` 객체 (또는 `Promise<Response>`)를 반환해야 합니다.
*   **`content`는 문자열, JSX, 뭐든 가능**: `c.render()`의 첫 번째 인자로 문자열, JSX 컴포넌트, 심지어 프로미스를 넘길 수도 있습니다 (레이아웃 함수가 프로미스를 처리할 수 있다면). Hono의 JSX 지원 덕분에 `<p>안녕</p>` 같은 걸 바로 넘길 수 있어서 편하죠.
*   **타입 정의는 선택이지만 강력 추천**: 타입스크립트 쓴다면 `ContextRenderer` 인터페이스 확장해서 `render` 함수의 두 번째 이후 인자들 타입을 명시해주는 게 좋습니다. 자동완성도 되고, 실수도 줄고, "이거 뭐 넣는 거였더라?" 하고 까먹을 일도 줄어듭니다.
*   **미들웨어 적용 순서 중요**: `setRenderer`를 사용하는 미들웨어는 당연히 `render`를 사용하는 라우트 핸들러보다 먼저 실행되도록 등록해야 합니다. 안 그러면 "어? 레이아웃 등록된 거 없다는데?" 하고 에러 만납니다.
*   **여러 레이아웃 사용 가능**: 경로별로 다른 미들웨어를 사용해서 `app.use('/admin/*', adminRenderer)` , `app.use('/blog/*', blogRenderer)` 처럼 다른 레이아웃을 설정할 수도 있습니다. "손님마다 다른 옷 입히기!"
*   **비동기 레이아웃**: 레이아웃 함수가 비동기 작업(예: 데이터베이스에서 공통 데이터 가져오기)을 포함한다면 `async` 함수로 만들고 `Promise<Response>`를 반환하도록 할 수 있습니다. `c.render`도 이걸 기다려줍니다.
*   **`c.html` 말고 다른 것도 가능은 함**: 이론적으로 레이아웃 함수에서 `c.text`나 `c.json`을 반환할 수도 있지만, `render`라는 이름 자체가 HTML 렌더링을 위한 거라 보통은 `c.html`을 씁니다. "이름값은 하자!"