`json()`
JSON을 `Content-Type: application/json`으로 렌더링합니다.

```javascript
app.get('/api', (c) => {
  return c.json({ message: 'Hello!' })
})
```

---

**얘 뭐 하는 애냐?**
`c.json()`은 Hono 핸들러 안에서 자바스크립트 객체나 배열을 JSON 문자열로 변환하고, HTTP 응답 헤더에 `Content-Type: application/json`을 자동으로 뙇! 찍어서 보내주는 아주 편리한 녀석입니다. "야, 이 데이터 JSON으로 포장해서 손님한테 갖다 드려!" 한마디면 끝나는 거죠.

**왜 쓰는데?**
1.  **API 개발 국룰**: 요즘 웹 서비스나 앱 만들 때 서버랑 클라이언트가 데이터를 주고받는 표준 방식이 바로 JSON입니다. `c.json()`은 이 JSON 응답을 만드는 과정을 아주 간단하게 만들어주죠. "API 만들려면 일단 이거부터!"
2.  **귀차니즘 해결사**: 원래대로라면 `JSON.stringify()`로 객체를 문자열로 바꾸고, `c.header('Content-Type', 'application/json')`으로 헤더도 직접 설정해야 하는데, `c.json()`은 이 모든 걸 한 방에 해결해줍니다. 개발자 손가락 관절 보호에 이바지합니다.
3.  **실수 방지**: `Content-Type` 헤더를 깜빡하거나 오타를 내면 클라이언트가 "이게 뭔 데이터야?" 하고 못 알아먹는 경우가 생깁니다. `c.json()`은 그런 실수를 원천 봉쇄해주죠. "묻지도 따지지도 않고 알아서 착착!"

**언제 불려 나오냐?**
Hono 라우트 핸들러 함수 안에서, 클라이언트에게 JSON 형태로 데이터를 응답하고 싶을 때 최종 반환 값으로 사용됩니다. `return c.json({ key: 'value' })` 이런 식으로요. "자, 이제 JSON으로 응답할 시간!"

**쓸 때 꿀팁 및 주의사항:**
*   **인자는 뭐든 OK (JSON으로 변환 가능하다면)**: 자바스크립트 객체 (`{}`), 배열 (`[]`), 문자열, 숫자, 불리언, `null` 등을 넘길 수 있습니다. Hono가 알아서 `JSON.stringify()` 처리해줍니다.
*   **`Content-Type` 자동 설정**: `c.json()`을 쓰면 기본적으로 `Content-Type: application/json;charset=UTF-8` 헤더가 자동으로 설정됩니다. UTF-8까지 챙겨주는 센스!
*   **상태 코드 변경 가능**: 기본 응답 코드는 `200 OK`입니다. 만약 다른 상태 코드(예: `201 Created`, `400 Bad Request`)를 보내고 싶다면 두 번째 인자로 상태 코드를 넣어주면 됩니다: `return c.json({ message: 'Created!' }, 201)` 또는 `return c.json({ error: 'Bad input' }, { status: 400 })`. 후자가 좀 더 명시적이죠.
*   **커스텀 헤더 추가**: `c.json()`으로 응답하기 전에 `c.header()`를 사용해서 다른 헤더를 추가할 수 있습니다. 아니면 `c.json(data, { headers: { 'X-My-Header': 'Hello' } })`처럼 `c.json()`의 두 번째 인자로 `ResponseInit` 객체를 넘겨서 헤더를 포함시킬 수도 있습니다.
*   **직렬화 불가능한 값 주의**: 함수(function)나 순환 참조(circular reference)가 있는 객체처럼 `JSON.stringify()`가 처리 못 하는 값을 넘기면 에러가 발생합니다. "JSON: 이건 못 먹는다 전해라~" 이런 건 미리 걸러내거나 변환해야 합니다.
*   **`pretty` 옵션 (Hono v4부터)**: `return c.json({ a: 1, b: 2 }, { pretty: true })` 이렇게 하면 JSON 응답이 예쁘게 들여쓰기 돼서 나옵니다. 개발 중에 API 응답 확인할 때 꿀이죠. 다만, 프로덕션에서는 불필요한 데이터 전송량을 늘릴 수 있으니 주의. "개발할 땐 이쁘게, 서비스할 땐 알뜰하게!"
*   **`c.text(JSON.stringify(data))`와의 차이**: `c.text()`로 직접 JSON 문자열을 보내도 되지만, 그럼 `Content-Type` 헤더를 수동으로 설정해야 합니다. `c.json()`은 그 수고를 덜어주니 웬만하면 `c.json()` 쓰세요. "편한 길 놔두고 왜 돌아가나?"