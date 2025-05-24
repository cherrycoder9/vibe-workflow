페이지 말고도 `+server.js` 파일로 라우트를 정의할 수 있어. 이걸 'API 라우트'나 '엔드포인트'라고 부르기도 하는데, 응답을 완전히 네 마음대로 주무를 수 있게 해줘. `+server.js` 파일은 `GET`, `POST`, `PATCH`, `PUT`, `DELETE`, `OPTIONS`, `HEAD` 같은 HTTP 동사에 해당하는 함수들을 만들어서 내보내. 이 함수들은 `RequestEvent` 객체를 인자로 받고, `Response` 객체를 반환해야 해.

예를 들어 `/api/random-number`라는 주소로 `GET` 요청을 처리하는 핸들러를 만들어 보자:

```javascript
// src/routes/api/random-number/+server.js

import { error } from '@sveltejs/kit';
import type { RequestHandler } from './$types';

export const GET: RequestHandler = ({ url }) => {
	const min = Number(url.searchParams.get('min') ?? '0'); // 쿼리 파라미터에서 min 값 가져오기 (없으면 0)
	const max = Number(url.searchParams.get('max') ?? '1'); // 쿼리 파라미터에서 max 값 가져오기 (없으면 1)

	const d = max - min;

	if (isNaN(d) || d < 0) { // 숫자 아니거나 min이 max보다 크면 에러!
		error(400, 'min and max must be numbers, and min must be less than max');
	}

	const random = min + Math.random() * d; // 랜덤 숫자 생성

	return new Response(String(random)); // 문자열로 바꿔서 응답
};
```

`Response`의 첫 번째 인자로는 `ReadableStream`도 쓸 수 있어서, 대용량 데이터를 스트리밍하거나 서버-센트 이벤트(SSE)를 만들 수도 있어. (단, AWS Lambda처럼 응답을 버퍼링하는 플랫폼에 배포할 때는 안 될 수도 있음)

편의를 위해 `@sveltejs/kit`에서 제공하는 `error`, `redirect`, `json` 같은 헬퍼 함수들을 써도 되고, 안 써도 그만이야.

만약 에러가 발생하면 (`error(...)`로 직접 던지든, 예상치 못한 에러든), 응답은 에러의 JSON 표현이나 대체 에러 페이지(`src/error.html`에서 커스텀 가능)로 나가. 이건 요청의 `Accept` 헤더에 따라 달라져. 이 경우에는 `+error.svelte` 컴포넌트는 렌더링되지 않아. 에러 처리에 대한 자세한 내용은 여기서 더 읽어볼 수 있어.

`OPTIONS` 핸들러를 만들 때는 Vite가 개발 환경에서 `Access-Control-Allow-Origin`이나 `Access-Control-Allow-Methods` 같은 헤더를 자동으로 넣어주는데, 실제 서비스 환경(프로덕션)에서는 네가 직접 추가하지 않으면 이 헤더들이 없다는 걸 알아둬.

`+layout` 파일은 `+server.js` 파일에 아무런 영향도 안 줘. 만약 모든 요청 전에 어떤 로직을 실행하고 싶다면, 서버 `handle` 훅에 추가해야 해.

---

**얘 뭐 하는 애냐?**
`+server.js`는 SvelteKit 앱에서 순수하게 데이터만 주고받는 API 엔드포인트를 만들 때 쓰는 녀석이야. HTML 페이지 보여주는 게 아니라, JSON 데이터, 텍스트, 파일 스트림 등 날것 그대로의 응답을 만들어낼 수 있지. "프론트엔드는 잠시 빠져 있어, 여긴 백엔드 형님들 구역이다!" 이런 느낌.

**왜 쓰는데?**
1.  **전용 API 구축**: 프론트엔드와 완전히 분리된 API 서버 역할을 할 수 있어. 앱 내부뿐만 아니라 외부 서비스나 다른 앱에서도 이 API를 호출해서 쓸 수 있지.
2.  **데이터 처리/제공**: DB에서 데이터 가져와서 가공 후 JSON으로 내려주거나, 사용자 입력 받아서 DB에 저장하는 등의 작업을 해.
3.  **폼(Form) 액션 대안**: SvelteKit의 폼 액션 기능도 좋지만, 더 복잡하거나 유연한 서버 사이드 로직이 필요할 때 `+server.js`가 유용해.
4.  **외부 서비스 연동**: 외부 API 호출해서 결과 받아오고, 그걸 다시 가공해서 클라이언트에게 전달하는 중간 다리 역할도 가능.
5.  **파일 업로드/다운로드, 스트리밍**: 큰 파일 처리나 실시간 데이터 스트리밍 같은 고급 기능 구현할 때 써.

**언제 불려 나오냐?**
클라이언트(브라우저, 다른 서버 등)가 `+server.js` 파일이 위치한 경로로 `GET`, `POST` 등의 HTTP 요청을 날리면, 해당 HTTP 메서드에 매핑된 함수(예: `export const GET = ...`)가 실행돼.

**쓸 때 꿀팁 및 주의사항:**
*   **HTTP 메서드별 함수**: `GET`, `POST`, `PUT`, `DELETE` 등 필요한 HTTP 메서드에 맞춰 함수를 `export` 해야 해. "손님, 주문하신 `GET` 나왔습니다!"
*   **`RequestEvent` 객체 활용**: 인자로 받는 `RequestEvent` 객체 안에는 요청 URL, 파라미터, 헤더, 쿠키, 요청 본문(request body) 등 유용한 정보가 다 들어있어. `url.searchParams.get('key')`로 쿼리 파라미터, `await request.json()`이나 `await request.formData()`로 요청 본문 파싱 가능.
*   **`Response` 객체 직접 생성**: `return new Response('Hello', { status: 200, headers: { 'Content-Type': 'text/plain' } });`처럼 `Response` 객체를 직접 만들어서 반환해야 해. `@sveltejs/kit`의 `json()` 헬퍼 쓰면 `return json({ message: 'Success' });` 이렇게 더 편하게 JSON 응답 가능.
*   **에러 처리는 `error()` 헬퍼로**: `import { error } from '@sveltejs/kit';` 해서 `throw error(400, 'Bad Request');` 식으로 에러 던지면 알아서 적절한 HTTP 에러 응답 만들어줌.
*   **CORS 주의 (특히 `OPTIONS` 핸들러)**: 다른 도메인에서 네 API를 호출하게 하려면 CORS 헤더 설정이 필수야. 개발 중에는 Vite가 편의를 봐주지만, 배포할 때는 직접 `Access-Control-Allow-Origin` 같은 헤더를 응답에 넣어줘야 해. "남의 집에서 우리 집 물건 쓰려면 허락은 받아야지?"
*   **`+layout.js`는 적용 안 됨**: `+server.js`는 HTML 페이지가 아니라서 주변 `+layout.js`의 영향을 받지 않아. 모든 API 요청에 공통 로직을 적용하고 싶으면 `src/hooks.server.js` 파일의 `handle` 함수를 써야 해.
*   **보안! 보안! 보안!**: 사용자 입력값 검증(validation) 철저히 하고, 인증/인가(authentication/authorization) 로직 잘 구현해야 해. SQL 인젝션, XSS 같은 공격에 탈탈 털리고 싶지 않으면. "문단속 잘하자."