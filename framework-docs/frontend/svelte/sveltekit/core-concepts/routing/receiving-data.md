`+server.js` 파일에 `POST`, `PUT`, `PATCH`, `DELETE`, `OPTIONS`, `HEAD` 같은 HTTP 요청 처리 함수(핸들러)를 만들어서 내보내면, 이걸로 완전한 API를 구축할 수 있어.

예를 들어, 더하기 기능을 하는 API를 만든다고 치자.

먼저, 사용자 화면 쪽 (`src/routes/add/+page.svelte`):
```svelte
<script lang="ts">
	let a = 0;
	let b = 0;
	let total = 0;

	async function add() {
		// '/api/add' 경로로 POST 요청을 보낸다.
		// 본문(body)에는 a와 b 값을 JSON 형태로 담고,
		// 헤더에는 내용물이 JSON이라고 알려준다.
		const response = await fetch('/api/add', {
			method: 'POST',
			body: JSON.stringify({ a, b }),
			headers: {
				'content-type': 'application/json'
			}
		});

		// 서버에서 돌려준 결과(JSON)를 받아서 total에 저장한다.
		total = await response.json();
	}
</script>

<input type="number" bind:value={a}> +
<input type="number" bind:value={b}> =
{total}

<button onclick={add}>Calculate</button>
```

그리고 서버 쪽 API 로직 (`src/routes/api/add/+server.js`):
```javascript
import { json } from '@sveltejs/kit';
import type { RequestHandler } from './$types';

// POST 요청을 처리하는 함수
export const POST: RequestHandler = async ({ request }) => {
	// 요청 본문에서 JSON 데이터를 꺼내서 a와 b를 얻는다.
	const { a, b } = await request.json();
	// a와 b를 더한 결과를 JSON 형태로 응답한다.
	return json(a + b);
};
```

근데 솔직히 말하면, 브라우저에서 서버로 데이터를 보낼 때는 폼 액션(form actions) 쓰는 게 더 나은 방법이야.

만약 `GET` 핸들러를 내보내면, `HEAD` 요청이 왔을 때 그 `GET` 핸들러가 반환할 내용물의 `content-length` (본문 길이)를 알려주게 돼.

---

**얘 뭐 하는 애냐?**
`+server.js`는 SvelteKit 프로젝트 안에 나만의 API 엔드포인트를 만드는 녀석이야. 프론트엔드 페이지(`+page.svelte`)랑 별개로, 서버에서 데이터만 주고받는 창구를 만드는 거지. "여기는 데이터 맛집, 주문은 HTTP 메서드로!"

**왜 쓰는데?**
1.  **백엔드 API 구축**: DB 조회, 외부 API 연동, 파일 처리 등 서버에서 해야 할 일들을 처리하는 API를 만들 수 있어. SvelteKit 하나로 풀스택 개발 쌉가능.
2.  **데이터 전용 창구**: 화면 그리는 거랑 상관없이 순수하게 데이터만 주고받고 싶을 때 써. 클라이언트 앱(웹, 모바일 등)이랑 데이터 통신할 때 유용하지.
3.  **HTTP 메서드별 처리**: `GET` (읽기), `POST` (생성), `PUT` (전체 수정), `PATCH` (부분 수정), `DELETE` (삭제) 등 HTTP 메서드에 맞춰서 각각 다른 로직을 태울 수 있어. RESTful API 만들기 딱 좋음.

**언제 불려 나오냐?**
클라이언트(브라우저, 다른 서버 등)에서 `+server.js` 파일이 위치한 경로로 특정 HTTP 메서드 요청을 보내면, 그에 해당하는 이름의 함수(예: `export const POST = ...`)가 호출돼. 예를 들어 `/api/add`로 `POST` 요청 보내면, `src/routes/api/add/+server.js` 파일 안의 `POST` 함수가 실행되는 식.

**쓸 때 꿀팁 및 주의사항:**
*   **`json()` 헬퍼는 사랑입니다**: 서버에서 클라이언트로 JSON 데이터 보낼 때 `import { json } from '@sveltejs/kit'` 해서 `return json(데이터)` 쓰면 알아서 `Content-Type` 헤더도 맞춰주고 편해.
*   **`request` 객체 활용**: 핸들러 함수는 `request` 객체를 인자로 받는데, 이걸로 요청 헤더, 본문, 파라미터 등을 다룰 수 있어. `await request.json()` (JSON 본문 파싱), `await request.formData()` (폼 데이터 파싱) 등 유용한 메서드가 많으니 잘 써먹자.
*   **응답은 `Response` 객체로**: 모든 핸들러는 반드시 `Response` 객체를 반환해야 해. `json()` 헬퍼도 결국 `Response` 객체를 만들어주는 놈이야. 직접 `new Response(본문, { status: 상태코드, headers: 헤더객체 })` 이렇게 만들 수도 있어.
*   **폼 데이터는 `form actions` 고려**: 단순 데이터 제출은 `+server.js`보다 `+page.server.js`의 `form actions`가 더 편하고 SvelteKit 기능(점진적 향상 등)도 잘 써먹을 수 있어. `+server.js`는 좀 더 범용적인 API 만들 때 좋지.
*   **에러 처리는 확실하게**: API에서 문제 생기면 적절한 HTTP 상태 코드랑 에러 메시지를 담은 `Response`를 반환해야 클라이언트가 "아, 망했구나" 하고 제대로 알 수 있어. `import { error } from '@sveltejs/kit'` 써서 에러 응답 만들 수도 있고.
*   **`GET`과 `HEAD`의 관계**: `GET` 핸들러 만들어두면 `HEAD` 요청은 자동으로 `GET` 응답의 헤더 정보만 가져가. 본문 데이터 없이 메타데이터만 필요할 때 유용.
*   **보안, 보안, 보안!**: API는 외부로 열린 창구니까 인증/인가 처리, 입력값 검증 철저히 해야 해. 안 그러면 "우리집 금고 비번은 1234!" 하는 거랑 똑같다.