# API Endpoints with server_js

폼 액션(Form actions)이 서버로 데이터를 보내는 데는 더 선호되는 방식이야. 점진적 향상도 가능하고 말이지. 하지만 `+server.js` 파일을 써서 (예를 들면) JSON API 같은 걸 만들 수도 있어. 그런 상호작용이 어떤 모습일지 한번 보자고:

`src/routes/send-message/+page.svelte` (원문은 +page지만, Svelte 파일이므로 확장자 명시)

```svelte
<script lang="ts">
	async function rerun() { // 비동기 함수로 선언하는 것이 일반적
		try {
			const response = await fetch('/api/ci', { // fetch는 Promise를 반환하므로 await 사용
				method: 'POST'
			});
			// 여기서 응답 처리 로직 추가 가능 (예: response.ok, response.json())
			console.log('CI 재실행 요청 성공:', response);
		} catch (error) {
			console.error('CI 재실행 요청 실패:', error);
		}
	}
</script>

<button on:click={rerun}>CI 재실행</button> <!-- Svelte에서는 onclick 대신 on:click -->
```

`src/routes/api/ci/+server.js`

```typescript
import type { RequestHandler } from './$types';
import { json } from '@sveltejs/kit'; // JSON 응답을 위해 json 헬퍼 사용 권장

export const POST: RequestHandler = async ({ request }) => { // request 객체 사용 가능
	// 여기서 뭔가 작업을 수행 (예: 데이터베이스 업데이트, 외부 API 호출 등)
	console.log('CI 재실행 API 호출됨');

	// 성공적으로 처리되었음을 알리는 응답 반환
	return json({ message: 'CI 재실행 작업이 시작되었습니다.' }, { status: 200 });
	// 혹은 단순 성공 응답
	// return new Response(null, { status: 204 });
};
```

---

얘 뭐 하는 애냐?
SvelteKit에서 `+server.js` (또는 `.ts`) 파일은 서버 사이드 로직을 실행하고 HTTP 요청에 응답하는 API 엔드포인트를 만드는 데 쓰여. 페이지를 보여주는 게 아니라, 데이터(주로 JSON)를 주고받거나 특정 작업을 처리하는 "창구" 역할을 하는 거지. "웹사이트의 뒷문 같은 존재랄까? 데이터 전용 출입구."

왜 쓰는데?
1.  **순수 API 제공**: 클라이언트(웹 브라우저, 모바일 앱 등)가 데이터를 요청하거나 전송할 수 있는 전용 API를 만들 때 써. `fetch` 같은 걸로 여기서 데이터를 가져가거나 보내는 거지.
2.  **폼 액션 외의 서버 로직**: 폼 제출과 직접 관련 없는 서버 작업, 예를 들어 주기적인 작업 트리거, 웹훅(webhook) 수신, 특정 이벤트에 대한 응답 등을 처리할 때 유용해.
3.  **외부 서비스 연동**: 외부 API와 통신하거나, 서버에서만 안전하게 처리해야 하는 로직(API 키 사용 등)을 수행할 때 중간 다리 역할을 해.

언제 불려 나오냐?
클라이언트에서 해당 경로(`src/routes/api/ci/+server.js`의 경우 `/api/ci`)로 `fetch` 등을 사용해 HTTP 요청(GET, POST, PUT, DELETE 등)을 보내면, `+server.js` 파일에 정의된 해당 HTTP 메서드(예: `export const POST = ...`) 함수가 실행돼. "누가 똑똑 노크하면 그때 일하는 애야."

쓸 때 꿀팁 및 주의사항:
*   **HTTP 메서드 함수**: `GET`, `POST`, `PUT`, `PATCH`, `DELETE` 등 필요한 HTTP 메서드에 맞춰 함수를 `export` 하면 돼. `export const GET = () => { ... };` 이런 식으로.
*   **`RequestEvent` 객체**: 핸들러 함수는 `RequestEvent` 객체를 인자로 받아. 여기엔 `request` (요청 정보), `cookies`, `params` (동적 라우트 파라미터), `url` (요청 URL 정보) 등이 들어있어서 유용하게 써먹을 수 있어. "요청에 대한 모든 정보가 담긴 보물상자."
*   **`Response` 객체 반환**: 핸들러 함수는 반드시 `Response` 객체를 반환해야 해. SvelteKit의 `json()` 헬퍼를 쓰면 JSON 응답 만들기가 편해 (`return json({ key: 'value' });`). 그냥 텍스트나 다른 형식으로 응답하려면 `new Response('Hello', { headers: { 'Content-Type': 'text/plain' } });`처럼 직접 만들면 돼.
*   **점진적 향상 X**: `+server.js`는 폼 액션처럼 `use:enhance` 같은 점진적 향상 기능을 기본으로 제공하지 않아. 순수 API 엔드포인트라서, 클라이언트에서 직접 `fetch` 로직과 상태 관리를 해줘야 해. "여긴 수동 기어야. 클러치랑 기어봉 직접 조작해야 한다고."
*   **보안**: API 엔드포인트는 외부에 노출되니까 CORS 설정, 인증/인가 로직을 꼼꼼하게 구현해야 해. 민감한 작업은 아무나 호출하면 안 되잖아.
*   **파일 이름**: `api/users.json/+server.js`처럼 경로 중간에 `.json` 같은 걸 넣어서 이게 JSON API 엔드포인트라는 걸 명시적으로 나타낼 수도 있어. 이건 필수는 아니지만, 가독성이나 경로 충돌 방지에 도움이 될 수 있지.