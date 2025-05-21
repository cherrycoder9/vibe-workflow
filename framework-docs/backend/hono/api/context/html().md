`html()`
HTML을 `Content-Type:text/html`로 렌더링합니다.

```javascript
app.get('/', (c) => {
  return c.html('<h1>Hello! Hono!</h1>')
})
```

---

**얘 뭐 하는 애냐?**
`c.html()`은 말 그대로 HTML 문자열을 브라우저한테 "이거 HTML이니까 그렇게 알아들어!" 하고 `Content-Type: text/html` 딱지 붙여서 보내주는 녀석입니다. 복잡한 거 없이 그냥 문자열로 된 HTML 코드를 바로 응답으로 쏴줄 때 쓰는 가장 간단한 방법이죠.

**왜 쓰는데?**
1.  **초간단 HTML 응답**: 간단한 웹페이지나 HTML 조각을 후딱 만들어서 보내야 할 때 최고입니다. "안녕하세요!" 한 줄짜리 HTML도 이걸로 뚝딱이죠.
2.  **템플릿 리터럴과 찰떡궁합**: 자바스크립트의 템플릿 리터럴(백틱 `` ` ``)이랑 같이 쓰면 동적인 HTML 만들기가 아주 편해집니다. ``c.html(`<h1>${user.name}님, 환영합니다!</h1>`)`` 이런 식으로요.
3.  **서버 사이드 렌더링(SSR)의 기초**: 복잡한 프론트엔드 프레임워크 없이 간단한 SSR을 구현할 때 기본 발판이 됩니다. 물론, 진짜 제대로 된 SSR 하려면 JSX나 템플릿 엔진 쓰는 게 낫지만, 맛보기용으론 충분하죠.

**언제 불려 나오냐?**
Hono 라우트 핸들러 함수 안에서 클라이언트에게 HTML 페이지를 응답으로 보내고 싶을 때, `return c.html(...)` 형태로 호출합니다. 이게 핸들러의 최종 리턴 값이 되는 거죠.

**쓸 때 꿀팁 및 주의사항:**
*   **`Content-Type`은 자동**: `c.html()` 쓰면 알아서 `Content-Type: text/html; charset=UTF-8` 헤더가 붙습니다. 따로 `c.header()` 안 해줘도 돼요. "알아서 다 해준다, 편하게 써라!"
*   **템플릿 리터럴 활용**: 위에 말했듯이, 동적인 데이터 넣을 땐 템플릿 리터럴이 국룰입니다. ``const title = '내 페이지'; return c.html(`<title>${title}</title>`);``
*   **`hono/html`의 `html` 헬퍼 (선택 사항, 근데 좋음)**: `import { html } from 'hono/html'` 해서 ``return c.html(html`<!DOCTYPE html>...`)`` 이렇게 쓰면, VS Code 같은 편집기에서 HTML 자동완성이나 문법 강조가 더 잘 돼서 개발 경험이 쾌적해집니다. 그냥 문자열로 쓰는 것보다 오타도 줄일 수 있고요. "개발자 편의성 UP!"
*   **보안! XSS 조심!**: 사용자 입력값을 HTML에 그대로 때려 박으면 XSS 공격에 탈탈 털릴 수 있습니다. 예를 들어 ``c.html(`<div>${userInput}</div>`)`` 이런 코드는 위험천만! `userInput`에 `<script>alert('해킹!')</script>` 같은 거 들어오면 바로 실행됩니다.
    *   **이스케이프 처리**: `hono/html/html-escape`에서 `htmlEscape` 함수를 가져오거나, Hono의 JSX를 사용하면 자동으로 이스케이프 처리해줘서 안전합니다. ``c.html(`<div>${htmlEscape(userInput)}</div>`)`` 이런 식으로요. "사용자 입력은 일단 의심하고 보자!"
*   **JSX/TSX 쓰고 싶으면?**: Hono는 JSX도 지원합니다! `hono/jsx` 미들웨어를 쓰고 파일 확장자를 `.jsx`나 `.tsx`로 하면 React처럼 HTML을 직관적으로 작성하고 컴포넌트도 만들 수 있습니다. `c.html()`은 문자열 기반이라 한계가 명확하니, 좀 더 복잡한 UI는 JSX 쓰는 게 정신 건강에 이롭습니다.
*   **스트리밍 아님**: `c.html()`은 HTML 문자열 전체를 만들어서 한방에 보내는 방식입니다. 만약 페이지가 엄청 크거나, 데이터를 조각조각 받아서 점진적으로 렌더링하고 싶다면 `c.stream()`이나 `c.streamText()` 같은 스트리밍 API를 써야 합니다. "한입만! 아니고 한방에 다 준다!"
*   **큰 HTML 파일은?**: 수십, 수백 줄짜리 HTML을 백틱 안에 다 넣는 건 좀 그렇죠. 그럴 땐 별도 `.html` 파일로 빼고, `fs.readFile` (Node.js 환경) 같은 걸로 읽어와서 `c.html()`에 전달하거나, 아예 전문 템플릿 엔진(EJS, Handlebars 등) 연동을 고려하는 게 좋습니다.