**Secure Headers 미들웨어와 함께 사용하기**
`hono/css` 헬퍼를 Secure Headers 미들웨어와 같이 쓰고 싶다면, `<Style />` 컴포넌트에 `nonce` 속성을 추가해서 `css` 헬퍼 때문에 발생할 수 있는 콘텐츠 보안 정책(Content-Security-Policy, CSP) 문제를 피할 수 있습니다. `nonce` 값으로는 `c.get('secureHeadersNonce')`를 사용하면 됩니다.

```typescript
// secureHeaders랑 NONCE를 hono/secure-headers에서 가져오자.
import { secureHeaders, NONCE } from 'hono/secure-headers'

// 모든 요청에 대해 Secure Headers 미들웨어 적용!
app.get(
  '*',
  secureHeaders({
    contentSecurityPolicy: {
      // style-src 지시어에 미리 정의된 NONCE 값을 설정.
      // 이러면 <style nonce="이값"> 태그만 허용한다는 뜻.
      styleSrc: [NONCE],
    },
  })
)

app.get('/', (c) => {
  const headerClass = css`
    background-color: orange;
    color: white;
    padding: 1rem;
  `
  return c.html(
    <html>
      <head>
        {/* 
          css 헬퍼의 style (그리고 script) 요소에 nonce 속성 설정!
          secureHeaders 미들웨어가 생성한 nonce 값을 c.get('secureHeadersNonce')로 가져와서 박아준다.
        */}
        <Style nonce={c.get('secureHeadersNonce')} />
      </head>
      <body>
        <h1 class={headerClass}>Hello!</h1>
      </body>
    </html>
  )
})
```

**꿀팁**
VS Code를 쓴다면, `vscode-styled-components` 확장을 설치해봐. `css` 태그가 붙은 템플릿 리터럴 안에서도 문법 강조(Syntax highlighting)랑 자동 완성(IntelliSense) 기능을 쓸 수 있어서 개발이 한결 편해질 거야. (마치 어두운 동굴에서 손전등 얻은 기분이랄까?)

```typescript
// 이건 그냥 일반적인 hono/css 사용 예시. 위 CSP 내용과는 별개.
app.get('/', (c) => {
  const headerClass = css`
    background-color: orange;
    color: white;
    padding: 1rem;
  `; // 세미콜론은 취향껏. 없어도 돌아감.

  return c.html(
    <html>
      <head>
        {/* <Style /> 컴포넌트로 위에서 만든 CSS를 HTML에 주입! */}
        <Style />
      </head>
      <body>
        <h1 class={headerClass}>Hello!</h1>
      </body>
    </html>
  );
});
```

---

**얘네 뭐 하는 애들이냐? (Secure Headers + `hono/css` 조합)**
`Secure Headers` 미들웨어는 웹사이트 보안을 강화하려고 HTTP 응답 헤더에 여러 가지 보안 설정을 추가해주는 녀석이야. 그중 하나가 CSP(콘텐츠 보안 정책)인데, "우리 사이트에서는 이런 종류의 스크립트나 스타일만 허용할 거야!" 하고 브라우저에 알려주는 거지. `hono/css`는 `<style>` 태그를 동적으로 페이지에 꽂아 넣는데, CSP가 빡세게 설정되어 있으면 "야, 너 뭔데 허락도 없이 스타일 집어넣어? 위험해!" 하고 막아버릴 수 있어.

이때 `nonce`라는 해결사가 등장해. 서버가 요청받을 때마다 아주 잠깐 동안만 유효한, 예측 불가능한 암표 같은 걸(`nonce` 값) 만들어서 CSP 설정과 `<Style />` 태그 양쪽에 똑같이 나눠주는 거야. 그러면 브라우저는 "오, 너 암표 있네? 통과!" 하고 스타일을 허용해주는 거지. "클럽 입장할 때 도장 찍어주는 거랑 비슷해."

**왜 쓰는데? (이 조합을)**
1.  **XSS 공격 방어 강화**: CSP는 악의적인 스크립트가 웹사이트에 주입되어 실행되는 걸 막는 데 아주 효과적이야. `hono/css`를 쓰면서도 이 강력한 보안 기능을 포기할 수 없으니 `nonce`를 쓰는 거지.
2.  **인라인 스타일/스크립트 안전하게 사용**: 원래 CSP는 `<style>` 태그나 `<script>` 태그 안에 직접 내용을 적는 인라인 방식이나, `eval()` 같은 위험한 셔틀은 막는 게 기본이야. 근데 `hono/css`는 `<style>` 태그를 만들어 넣으니까 문제가 될 수 있거든. `nonce`는 "얘는 내가 허락한 안전한 인라인 스타일이야"라고 콕 집어 알려주는 역할을 해.

**언제 불려 나오냐? (이 조합이)**
Hono 앱에서 `hono/secure-headers` 미들웨어를 사용해서 CSP를 설정하고, 동시에 `hono/css`로 동적 스타일을 주입하려고 할 때 이 조합이 필요해.

1.  먼저 `secureHeaders` 미들웨어를 앱에 등록하면서 `contentSecurityPolicy` 옵션에 `styleSrc: [NONCE]` (또는 `scriptSrc` 등 필요에 따라)를 설정해. `NONCE`는 `hono/secure-headers`에서 임포트한 특별한 상수야.
2.  그리고 `hono/css`의 `<Style />` 컴포넌트를 렌더링할 때 `nonce` 속성값으로 `c.get('secureHeadersNonce')`를 넣어줘. `secureHeaders` 미들웨어가 요청 컨텍스트(`c`)에 해당 요청에 대한 `nonce` 값을 저장해두거든.

**쓸 때 꿀팁 및 주의사항:**
*   **`nonce`는 매 요청마다 새로 생성**: `secureHeaders` 미들웨어가 알아서 해주는 부분이지만, `nonce` 값은 보안을 위해 요청마다 달라야 하고 예측 불가능해야 해. 고정된 값을 쓰면 있으나 마나야. "매번 다른 암호를 쓰는 금고!"
*   **`NONCE` 상수 정확히 사용**: `hono/secure-headers`에서 제공하는 `NONCE` 상수를 써야 미들웨어가 "아, 여기에 실제 nonce 값을 채워 넣으라는 말이구나" 하고 알아들어. 그냥 문자열로 `'nonce-xxxx'` 이렇게 하드코딩하면 안 돼.
*   **CSP 다른 지시어들과의 관계**: `styleSrc` 말고도 `scriptSrc`, `defaultSrc` 등 다양한 CSP 지시어가 있어. `hono/css`는 주로 스타일에 관련되지만, 만약 `hono/jsx/dom` 같은 걸로 스크립트도 동적으로 넣는다면 `scriptSrc`에도 `NONCE`를 적용해야 할 수 있어. "보안 설정은 하나만 챙기면 구멍 나기 십상."
*   **`c.get('secureHeadersNonce')`의 존재**: 이 값은 `secureHeaders` 미들웨어가 먼저 실행되어야 컨텍스트에 담기는 거야. 미들웨어 등록 순서가 꼬이면 `undefined`가 나올 수도 있으니 주의.
*   **VS Code 확장 (`vscode-styled-components`)**: 이건 `hono/css` 자체를 쓸 때 편의성을 높여주는 팁이야. CSP랑 직접적인 관련은 없지만, 코드 짜는 경험을 부드럽게 해주니 설치하면 좋지. "개발도 장비빨!"
*   **너무 빡센 CSP는 개발을 방해할 수도**: 보안은 중요하지만, 너무 모든 걸 막아버리면 개발할 때나 특정 기능 구현할 때 힘들어질 수 있어. 적절한 수준에서 타협점을 찾는 게 중요해. "쇠창살 없는 감옥은 없지만, 숨 막히는 감옥도 문제."