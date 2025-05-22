쿠키 이름에 `__Secure-`나 `__Host-` 같은 접두사를 붙여서 보안을 강화하는 기능에 대한 얘기네. Hono의 쿠키 헬퍼 함수들이 이 접두사를 지원한다는 거야.

**접두사 붙은 쿠키 가져올 때 (확인)**

쿠키 이름에 이런 특수 접두사가 붙어있는지 확인할 때는 `prefix` 옵션을 명시해주면 돼.

```typescript
// '__Secure-' 접두사가 붙은 'yummy_cookie'를 가져오고 싶을 때
const securePrefixCookie = getCookie(c, 'yummy_cookie', 'secure')
// '__Host-' 접두사가 붙은 'yummy_cookie'를 가져오고 싶을 때
const hostPrefixCookie = getCookie(c, 'yummy_cookie', 'host')

// 서명된 쿠키도 마찬가지!
// '__Secure-' 접두사가 붙은 서명된 'fortune_cookie'
const securePrefixSignedCookie = await getSignedCookie(
  c,
  secret, // 서명에 사용된 비밀키
  'fortune_cookie',
  'secure' // '나 __Secure- 붙은 놈이야!'
)
// '__Host-' 접두사가 붙은 서명된 'fortune_cookie'
const hostPrefixSignedCookie = await getSignedCookie(
  c,
  secret,
  'fortune_cookie',
  'host' // '나는 __Host- 붙은 놈이지롱!'
)
```

**접두사 붙여서 쿠키 설정할 때**

쿠키를 만들 때부터 "얘는 특별한 놈이야!" 하고 접두사를 지정하고 싶으면, 역시 `prefix` 옵션에 원하는 값을 넣어주면 돼.

```typescript
// 'delicious_cookie'라는 이름 앞에 '__Secure-'를 붙여서 설정
setCookie(c, 'delicious_cookie', 'macha', { // 값은 'macha'
  prefix: 'secure', // 또는 'host'를 쓸 수도 있음
})

// 서명된 쿠키도 동일하게!
await setSignedCookie(
  c,
  'delicious_cookie', // 쿠키 이름
  'macha', // 쿠키 값
  'secret choco chips', // 서명용 비밀키
  {
    prefix: 'secure', // '__Secure-' 접두사 추가! (또는 'host')
  }
)
```

**베스트 프랙티스를 따르자!**

새로운 쿠키 RFC (일명 cookie-bis)나 CHIPS 같은 표준 문서에는 개발자들이 따라야 할 쿠키 설정 권장 사항들이 있어. Hono는 이런 권장 사항들을 잘 지키려고 노력한다는 얘기야. 예를 들면:

*   `Max-Age`/`Expires` (쿠키 유효기간) 제한
*   `__Host-`/`__Secure-` 접두사 사용 규칙
*   `Partitioned` (CHIPS 관련) 쿠키 제한

그래서 Hono의 쿠키 헬퍼는 다음과 같은 상황에서 쿠키를 파싱할 때 에러를 뱉어낼 거야. (잘못 쓰면 혼난다는 뜻!)

*   쿠키 이름이 `__Secure-`로 시작하는데, `secure` 옵션이 설정 안 됐을 때. ("야, `__Secure-` 붙이려면 `secure: true`는 기본 아니냐?")
*   쿠키 이름이 `__Host-`로 시작하는데, `secure` 옵션이 설정 안 됐을 때. ("`__Host-`도 `secure`는 깔고 가야지!")
*   쿠키 이름이 `__Host-`로 시작하는데, `path`가 `/` (루트 경로)가 아닐 때. ("`__Host-`는 루트 경로 전용이다, 아무 데나 쓰지 마라.")
*   쿠키 이름이 `__Host-`로 시작하는데, `domain`이 설정되어 있을 때. ("`__Host-`는 도메인 지정 못 한다, 정신 차려!")
*   `maxAge` 옵션 값이 400일을 초과할 때. ("쿠키 유효기간 너무 길게 잡지 마, 적당히 해라.")
*   `expires` 옵션 값이 현재 시간으로부터 400일보다 먼 미래일 때. (위랑 같은 말, 표현만 다름)

---

**얘 뭐 하는 애냐?**
`__Secure-`와 `__Host-` 접두사는 쿠키를 더 안전하게 만들려고 브라우저랑 서버가 미리 약속한 특별한 이름 규칙이야. 이 접두사가 붙은 쿠키는 특정 보안 조건을 만족해야만 브라우저가 제대로 취급해주거든. Hono의 쿠키 관련 함수들(`getCookie`, `setCookie`, `getSignedCookie`, `setSignedCookie`)은 이 접두사 규칙을 이해하고, 개발자가 쉽게 이런 보안 강화 쿠키를 만들고 읽을 수 있도록 도와주는 거지. "쿠키에 보안 딱지 붙여주는 기능!"

**왜 쓰는데?**
1.  **HTTPS 강제 (`__Secure-`, `__Host-`)**: 이 접두사가 붙은 쿠키는 반드시 HTTPS 연결을 통해서만 전송돼. 실수로 HTTP 연결에서 쿠키를 설정하거나 전송하려고 하면 브라우저가 "이 쿠키는 HTTPS 전용이거든?" 하고 거부해버려. 중간자 공격(MITM)으로 쿠키가 탈취될 위험을 줄여주지. "암호화 통신 아니면 취급 안 함!"
2.  **도메인 및 경로 제한 (`__Host-`)**: `__Host-` 접두사는 더 빡센 규칙을 적용해.
    *   반드시 `Secure` 속성 (HTTPS)이 있어야 하고,
    *   반드시 `Path=/` (루트 경로)로 설정해야 하며,
    *   `Domain` 속성을 설정하면 안 돼 (즉, 현재 호스트에만 국한됨).
    이러면 서브도메인에서 악의적으로 메인 도메인의 쿠키를 덮어쓰는 공격(Cookie Tossing) 같은 걸 막는 데 도움이 돼. "우리 집 안방 키는 우리 집에서만 써야지, 옆집에서 쓰면 안 되잖아?"

**언제 불려 나오냐?**
주로 로그인 세션 ID나 중요한 사용자 설정처럼 민감한 정보를 담는 쿠키를 다룰 때, 보안을 한층 강화하고 싶으면 이 접두사를 사용해. `setCookie`나 `setSignedCookie` 할 때 `prefix` 옵션에 `'secure'`나 `'host'`를 지정해서 쿠키를 생성하고, `getCookie`나 `getSignedCookie` 할 때도 해당 `prefix` 옵션을 명시해서 읽어오면 돼.

**쓸 때 꿀팁 및 주의사항:**
*   **`__Secure-` vs. `__Host-`**:
    *   `__Secure-MyCookie`: HTTPS 연결에서만 전송. `Path`나 `Domain` 설정은 자유로움.
    *   `__Host-MyCookie`: HTTPS 연결 + `Path=/` + `Domain` 없음. 가장 강력한 보안 레벨.
    "보안 레벨 골라 먹는 재미!"
*   **HTTPS는 기본 중의 기본**: 이 접두사 쓰려면 웹사이트 전체가 HTTPS로 서비스되어야 의미가 있어. HTTP 환경에서는 접두사 붙여봤자 브라우저가 무시하거나 에러 뿜는다. "자물쇠 채우려면 문부터 튼튼해야지."
*   **Hono가 알아서 검사해줌 (땡큐!)**: 위에 언급된 것처럼, Hono는 `__Host-` 접두사 규칙 (Secure, Path=/, Domain 없음)을 어기거나, `__Secure-`인데 Secure 속성 빼먹거나, 유효기간 너무 길게 잡으면 에러를 발생시켜. 개발자가 실수로 보안 구멍 만드는 걸 막아주는 고마운 기능이지. "Hono가 잔소리꾼 엄마처럼 챙겨준다."
*   **브라우저 호환성**: 요즘 웬만한 최신 브라우저는 이 접두사 규칙을 다 지원하지만, 아주 오래된 구닥다리 브라우저에서는 제대로 작동 안 할 수도 있다는 점은 염두에 둬야 해 (근데 그런 브라우저 쓰는 사람 있으면 빨리 업그레이드하라고 등짝 스매싱 날려야...).
*   **이름이 곧 스펙**: `__Secure-`나 `__Host-`는 단순한 이름이 아니라, 그 자체로 쿠키의 동작 방식을 규정하는 약속이야. 그래서 쿠키 이름 지을 때 신중해야 하고, 이 접두사를 쓸 거면 그 규칙을 정확히 따라야 해. "이름값 못하면 쫓겨난다."
*   **서명된 쿠키와 함께 쓰면 금상첨화**: 민감한 정보는 서명된 쿠키(`setSignedCookie`)로 내용물 변조 방지하고, 거기에 `__Secure-`나 `__Host-` 접두사까지 붙여서 전송 보안까지 챙기면 보안 레벨이 한층 더 올라가지. "이중 잠금장치!"