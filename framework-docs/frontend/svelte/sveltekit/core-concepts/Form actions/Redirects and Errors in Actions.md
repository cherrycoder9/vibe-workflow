# Redirects and Errors in Actions

리다이렉트 (그리고 에러 처리)는 `load` 함수에서와 똑같이 작동해:

**src/routes/login/+page.server.js 예시:**
```javascript
import { fail, redirect } from '@sveltejs/kit'; // SvelteKit에서 제공하는 fail, redirect 함수 임포트
import * as db from '$lib/server/db';         // 데이터베이스 관련 로직
import type { Actions } from './$types';       // 타입 임포트

export const actions = {
	login: async ({ cookies, request, url }) => {
		const data = await request.formData();
		const email = data.get('email');
		const password = data.get('password');

		const user = await db.getUser(email); // 이메일로 사용자 조회
		if (!user) {
			// 사용자가 없으면 400 에러와 함께 'email' 필드에 대한 정보, 'missing: true' 반환
			// 이 데이터는 페이지의 'form' 속성으로 전달됨
			return fail(400, { email, missing: true });
		}

		// 입력된 비밀번호를 해시 처리한 값과 DB의 사용자 비밀번호가 다르면
		if (user.password !== db.hash(password)) { // db.hash는 예시일 뿐, 실제로는 안전한 해싱 및 비교 함수 사용해야 함
			// 400 에러와 함께 'email' 필드에 대한 정보, 'incorrect: true' 반환
			return fail(400, { email, incorrect: true });
		}

		// 로그인 성공: 세션 생성 및 쿠키 설정
		cookies.set('sessionid', await db.createSession(user), { path: '/' });

		// URL 쿼리 파라미터에 'redirectTo'가 있으면 해당 경로로 리다이렉트
		if (url.searchParams.has('redirectTo')) {
			// 303 See Other 상태 코드로 리다이렉트 (POST 요청 후 GET으로 리다이렉트 시 권장)
			redirect(303, url.searchParams.get('redirectTo'));
		}

		// 리다이렉트할 곳이 없으면 성공 데이터 반환
		return { success: true };
	},
	register: async (event) => {
		// TODO: 사용자 등록 로직 구현
	}
} satisfies Actions;
```

---

**얘 뭐 하는 애냐?**
SvelteKit의 `actions` (주로 폼 처리 로직) 안에서 사용자를 다른 페이지로 보내거나(리다이렉트), 폼 처리 중 문제가 생겼을 때 에러 정보를 클라이언트에게 알려주는 기능이야. "폼 제출하고 나서 '딴 데로 가!' 하거나 '야, 이거 틀렸어!' 하고 알려주는 역할이지."

**왜 쓰는데? (목적)**
1.  **사용자 경험 향상**: 로그인 성공하면 마이페이지로 보내주고, 실패하면 "아이디 틀림" 같은 명확한 피드백을 줘서 사용자가 뭘 해야 할지 알기 쉽게 해줘. "헤매지 않게 길 안내 잘 해주는 네비게이션."
2.  **정상적인 요청 흐름 제어**: 폼 데이터 처리 후 다음 단계로 자연스럽게 넘어가도록 유도. 예를 들어, 글쓰기 폼 제출 후 작성된 글 상세 페이지로 이동시키는 거.
3.  **명확한 오류 전달**: `fail` 함수를 쓰면 HTTP 상태 코드와 함께 구체적인 에러 데이터를 폼으로 다시 보낼 수 있어서, "어떤 입력 칸이 왜 잘못됐는지" 사용자에게 정확히 알려줄 수 있어.

**언제 불려 나오냐? (적용 시점)**
*   `+page.server.js` 파일 내의 `actions` 객체 안에 정의된 함수 내부에서 특정 조건이 만족될 때.
*   로그인 성공/실패, 데이터 유효성 검사 실패, 권한 없음 등 서버에서 폼 요청을 처리하다가 페이지 이동이나 명시적인 에러 반환이 필요할 때 호출돼.

**쓸 때 꿀팁 및 주의사항:**
*   **`redirect`는 `throw` 한다**: `redirect` 함수를 호출하면 내부적으로 예외를 발생시켜서 코드 실행을 중단하고 브라우저에 리다이렉트 응답을 보내. 그러니까 `redirect` 호출 밑에 있는 코드는 실행 안 된다고 보면 돼. "일단 던지고 본다."
*   **`fail`은 데이터를 반환**: `fail(status, data)`은 HTTP `status` 코드(주로 4xx)와 함께 `data` 객체를 페이지의 `form` 속성으로 돌려줘. 이걸로 "이메일 형식이 틀렸어요" 같은 상세 에러 메시지를 폼 옆에 보여줄 수 있지. `fail`은 예외를 던지지 않고 그냥 값을 반환하는 거라, 그 뒤에 코드가 더 있으면 실행될 수도 있으니 `return fail(...)` 형태로 쓰는 게 일반적이야.
*   **HTTP 상태 코드**: `redirect` 시에는 주로 `303 See Other` (POST 요청 후 결과 페이지를 GET으로 보여줄 때)나 `302 Found` (일시적 이동), `301 Moved Permanently` (영구 이동) 등을 상황에 맞게 써야 해. `fail`은 주로 `400 Bad Request`, `401 Unauthorized`, `403 Forbidden`, `422 Unprocessable Entity` 등을 사용. "상황에 맞는 옷을 입혀주자."
*   **`url.searchParams` 활용**: 로그인 후 원래 가려던 페이지로 보내주는 기능 만들 때 유용해. 로그인 페이지 URL에 `?redirectTo=/profile` 이런 식으로 붙여놓고, 로그인 성공하면 `url.searchParams.get('redirectTo')` 값으로 리다이렉트 시키는 거지. "똑똑한 리다이렉션."