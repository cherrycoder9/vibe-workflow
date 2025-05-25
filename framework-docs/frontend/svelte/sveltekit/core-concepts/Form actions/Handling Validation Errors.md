# Handling Validation Errors

데이터가 유효하지 않아서 요청을 처리할 수 없는 경우, 사용자가 다시 시도할 수 있도록 이전에 제출했던 폼 값과 함께 유효성 검사 오류를 돌려보낼 수 있어. `fail` 함수를 사용하면 HTTP 상태 코드(유효성 검사 오류의 경우 보통 400이나 422)와 데이터를 함께 반환할 수 있지. 이 상태 코드는 `page.status`를 통해, 데이터는 `form`을 통해 접근할 수 있어:

**src/routes/login/+page.server.js 예시:**
```javascript
import { fail } from '@sveltejs/kit'; // fail 함수 임포트
import * as db from '$lib/server/db'; // 데이터베이스 관련 로직
import type { Actions } from './$types'; // 타입 임포트

export const actions = {
	login: async ({ cookies, request }) => {
		const data = await request.formData();
		const email = data.get('email');
		const password = data.get('password');

		// 이메일 필드가 비어있는 경우
		if (!email) {
			// 400 상태 코드와 함께 오류 데이터 반환
			// 사용자가 입력했던 이메일 값(비었겠지만)과 'missing' 플래그를 전달
			return fail(400, { email, missing: true });
		}

		const user = await db.getUser(email); // 이메일로 사용자 조회

		// 사용자가 없거나 비밀번호가 일치하지 않는 경우
		// (db.hash(password)는 예시일 뿐, 실제로는 안전한 해시 비교 방식을 사용해야 함)
		if (!user || user.password !== db.hash(password)) {
			// 400 상태 코드와 함께 오류 데이터 반환
			// 사용자가 입력했던 이메일 값과 'incorrect' 플래그를 전달
			return fail(400, { email, incorrect: true });
		}

		// 로그인 성공 시 세션 생성 및 쿠키 설정
		cookies.set('sessionid', await db.createSession(user), { path: '/' });

		return { success: true }; // 성공 응답
	},
	register: async (event) => {
		// TODO: 사용자 등록 로직 구현
	}
} satisfies Actions;
```
주의할 점은, 보안을 위해 페이지에는 이메일 값만 돌려주고 비밀번호는 돌려주지 않는다는 거야. "비번은 소중하니까, 서버 밖으로 내보내지 마라."

**src/routes/login/+page.svelte 예시:**
```svelte
<form method="POST" action="?/login">
	{#if form?.missing}<p class="error">이메일 필드는 필수입니다.</p>{/if}
	{#if form?.incorrect}<p class="error">잘못된 정보입니다!</p>{/if}
	<label>
		이메일
		<input name="email" type="email" value={form?.email ?? ''}>
	</label>
	<label>
		비밀번호
		<input name="password" type="password">
	</label>
	<button>로그인</button>
	<button formaction="?/register">회원가입</button>
</form>
```
반환되는 데이터는 JSON으로 직렬화할 수 있어야 해. 그 외의 구조는 전적으로 네 마음대로 정하면 돼. 예를 들어, 페이지에 여러 폼이 있다면, 반환된 폼 데이터가 어떤 `<form>`을 가리키는지 `id` 속성 같은 걸로 구분할 수도 있겠지.

---

**얘 뭐 하는 애냐?**
SvelteKit 액션에서 폼 데이터가 유효하지 않을 때, 사용자에게 "야, 너 이거 잘못 입력했어!"라고 알려주고, 뭐가 문제인지, 그리고 이전에 뭘 입력했었는지 (비밀번호 빼고!) 다시 보여주는 기능이야. `fail` 함수가 이 모든 걸 처리해주는 마법사 같은 놈이지. "틀렸으면 다시 입력해, 친절하게 알려줄게."

**왜 쓰는데? (목적)**
1.  **사용자 친화적 피드백**: 그냥 "에러!" 하고 끝내는 게 아니라, 뭐가 문제인지, 어떤 값을 다시 입력해야 하는지 명확하게 알려줘서 사용자 경험을 개선해. "사용자는 바보가 아니다, 하지만 친절은 언제나 옳다."
2.  **데이터 유지**: 잘못 입력했어도 사용자가 이미 입력한 값들(예: 아이디, 이메일)은 그대로 유지시켜줘서 다시 다 입력하는 수고를 덜어줘. 단, 비밀번호 같은 민감 정보는 빼고.
3.  **명확한 HTTP 상태 코드**: 성공(2xx)이 아닌, 클라이언트 오류(주로 400 Bad Request 또는 422 Unprocessable Entity)임을 명시적으로 알려줘서 네트워크 통신 관점에서도 깔끔해.

**언제 불려 나오냐? (사용 시점)**
*   `+page.server.js` 파일 내의 `actions` 함수 안에서, `request.formData()`로 받은 데이터가 비즈니스 로직이나 유효성 검사 규칙에 맞지 않을 때. 예를 들어, "이메일 형식이 아니네?", "비밀번호 너무 짧은데?" 같은 상황.

**쓸 때 꿀팁 및 주의사항:**
*   **`fail(statusCode, data)`**: 첫 번째 인자는 HTTP 상태 코드 (보통 400이나 422), 두 번째 인자는 페이지로 전달할 데이터 객체. 이 데이터는 페이지의 `form` 프롭으로 넘어가.
*   **민감 정보는 NO!**: 사용자 편의도 좋지만, 비밀번호 같은 민감 정보는 절대 `fail` 함수로 다시 돌려보내면 안 돼. 보안은 기본 중의 기본. "비번 다시 보여주면 해커한테 조공하는 셔틀."
*   **JSON 직렬화 가능**: `fail`로 넘기는 데이터는 JSON으로 바꿀 수 있는 형태여야 해. 함수나 `Date` 객체 같은 건 알아서 처리하거나 문자열로 바꿔야 함.
*   **페이지에서 활용**: `+page.svelte`에서는 `form` 프롭을 통해 `fail`이 넘긴 데이터에 접근하고, `page.status` (SvelteKit 2 이전엔 `error.status`)로 HTTP 상태 코드를 확인할 수 있어. 이걸로 조건부 렌더링해서 에러 메시지 보여주면 돼.
*   **여러 폼 구분**: 한 페이지에 폼이 여러 개면, `fail`로 넘기는 데이터에 `formId: 'loginForm'` 같은 식별자를 추가해서 어떤 폼에서 발생한 오류인지 구분할 수 있게 만들면 좋아. "이 에러가 그 에러인지, 저 에러인지 헷갈리면 안 되잖아."