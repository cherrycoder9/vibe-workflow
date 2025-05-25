# Anatomy of an Action
각 액션(action)은 `RequestEvent` 객체를 받아서, 이 객체의 `request.formData()` 메서드를 통해 폼 데이터를 읽어올 수 있게 해줘. 요청을 처리한 후 (예를 들어, 쿠키를 설정해서 사용자를 로그인시키는 것처럼), 액션은 데이터를 반환할 수 있는데, 이 데이터는 해당 페이지의 `form` 속성을 통해서나 앱 전역의 `page.form`을 통해서 다음 업데이트가 있을 때까지 사용 가능해. "서버에서 할 일 하고, 결과는 프론트에 살짝 찔러주는 거지."

**src/routes/login/+page.server.js 예시:**
```javascript
import * as db from '$lib/server/db'; // 데이터베이스 관련 로직이 담긴 모듈이라고 가정
import type { PageServerLoad, Actions } from './$types'; // SvelteKit 타입 임포트

// 페이지가 로드될 때 실행되는 함수
export const load: PageServerLoad = async ({ cookies }) => {
	// 쿠키에서 'sessionid' 값을 가져와 사용자 정보를 조회
	const user = await db.getUserFromSession(cookies.get('sessionid'));
	return { user }; // 조회된 사용자 정보를 페이지 데이터로 반환
};

// 폼 제출을 처리하는 액션들의 모음
export const actions = {
	// 'login'이라는 이름의 액션
	login: async ({ cookies, request }) => {
		const data = await request.formData(); // 요청으로부터 폼 데이터 추출
		const email = data.get('email');       // 'email' 필드 값 가져오기
		const password = data.get('password'); // 'password' 필드 값 가져오기

		// 이메일로 사용자 정보 조회 (실제로는 비밀번호 검증 로직도 필요)
		const user = await db.getUser(email);
		// 세션 생성 후, 'sessionid'라는 이름으로 쿠키에 저장 (경로는 '/')
		cookies.set('sessionid', await db.createSession(user), { path: '/' });

		return { success: true }; // 액션 처리 성공 여부와 함께 데이터 반환
	},
	// 'register'라는 이름의 액션
	register: async (event) => {
		// TODO: 여기에 사용자 등록 로직을 구현해야 함
	}
} satisfies Actions; // 'actions' 객체가 'Actions' 타입을 만족하는지 TypeScript가 검사
```

**src/routes/login/+page.svelte 예시:**
```svelte
<script lang="ts">
	import type { PageProps } from './$types'; // 페이지 속성 타입 임포트

	// Svelte 5의 $props() 룬을 사용하여 'data'와 'form' 속성을 받음
	// 'data'는 load 함수의 반환값, 'form'은 액션의 반환값
	let { data, form }: PageProps = $props();
</script>

{#if form?.success}
	<!-- 이 메시지는 일시적임; 폼 제출에 대한 응답으로 페이지가 렌더링될 때만 존재.
	     사용자가 페이지를 새로고침하면 사라짐. -->
	<p>성공적으로 로그인되었습니다! 환영합니다, {data.user.name}님.</p>
{/if}
```

`PageProps`는 SvelteKit 2.16.0 버전에 추가되었어. 그 이전 버전에서는 `data`와 `form` 속성의 타입을 각각 개별적으로 지정해야 했지:

**+page.svelte (이전 버전):**
```svelte
<script lang="ts">
	import type { PageData, ActionData } from './$types';

	// data와 form 속성을 각각의 타입으로 선언하여 받음
	let { data, form }: { data: PageData, form: ActionData } = $props();
</script>
```
Svelte 4에서는 `$props()` 대신 `export let data`와 `export let form`을 사용해서 속성을 선언했어. "버전 따라 문법도 바뀌는 건 국룰이지."

---

**얘 뭐 하는 애냐?**
주로 HTML `<form>`을 통해 서버로 데이터를 보낼 때, 그 데이터를 받아서 처리하는 서버 사이드 함수야. 로그인, 회원가입, 게시글 작성/수정/삭제처럼 데이터베이스에 뭔가 변화를 주는 작업(CUD: Create, Update, Delete)을 할 때 쓰이지. "클라이언트의 요청을 서버에서 받아 처리하는 해결사랄까."

**왜 쓰는데? (목적)**
1.  **점진적 향상 (Progressive Enhancement)**: 자바스크립트가 동작하지 않는 환경에서도 폼 제출 같은 핵심 기능은 문제없이 돌아가게 만들 수 있어. "JS 없어도 기본은 한다!"
2.  **보안 강화**: 데이터 유효성 검사나 중요 로직을 클라이언트가 아닌 서버에서 처리하니까, 클라이언트 측 코드 조작으로 인한 보안 문제를 막을 수 있어. "중요한 건 서버에서, 알지?"
3.  **간결한 상태 관리**: 폼 처리 결과(성공, 실패, 반환 데이터 등)를 페이지로 쉽게 넘겨줄 수 있고, SvelteKit이 로딩 상태나 에러 처리도 어느 정도 알아서 해줘서 개발이 편해져.

**언제 불려 나오냐? (호출 시점)**
*   HTML `<form>` 태그에 `method="POST"` 속성을 주고, 해당 폼이 서버로 제출될 때.
*   폼의 `action` 속성에 `?/actionName` 형태로 특정 액션을 지정하면, `+page.server.js` 파일 내 `actions` 객체에 정의된 해당 `actionName` 함수가 호출돼.
*   SvelteKit의 `use:enhance` 디렉티브를 사용하면 페이지 새로고침 없이 AJAX처럼 폼을 제출하고 액션을 실행할 수도 있어.

**쓸 때 꿀팁 및 주의사항:**
*   **서버 전용**: 액션은 무조건 `+page.server.js` 파일 안에 정의해야 해. 브라우저에서는 돌아가지 않아.
*   **여러 액션 정의**: `actions` 객체 안에 `login`, `register`, `deletePost`처럼 여러 개의 액션을 만들어두고, 폼에서 `action="?/deletePost"` 같이 골라 쓸 수 있어.
*   **결과 처리**: 액션 함수는 데이터를 반환할 수 있고, 이 데이터는 페이지 컴포넌트의 `form` 프롭으로 전달돼. 보통 성공 여부, 메시지, 혹은 새로 생성된 데이터 ID 같은 걸 넘겨주지. 이 `form` 데이터는 일시적이라, 페이지가 다시 로드되거나 다른 액션이 실행되면 사라져.
*   **리다이렉트와 에러**: 액션 처리 후 다른 페이지로 보내고 싶으면 `redirect` 헬퍼를, 심각한 오류가 나면 `error` 헬퍼를 사용해. 둘 다 `@sveltejs/kit`에서 가져올 수 있어.
*   **`use:enhance` 활용**: 이걸 쓰면 폼 제출 시 페이지 전체가 깜빡이는 걸 막고, 마치 SPA처럼 부드럽게 데이터를 처리할 수 있어. 사용자 경험(UX)이 훨씬 좋아지지. "깜빡임 없이 부드럽게, 이게 요즘 대세."
*   **입력값 검증은 필수**: 클라이언트에서 넘어온 데이터는 절대 믿으면 안 돼. 항상 서버 사이드 액션에서 다시 한번 철저하게 검증해야 해. "사용자 입력은 일단 의심부터 하고 보자."
*   **CSRF 보호**: SvelteKit은 알아서 CSRF(Cross-Site Request Forgery) 공격을 막아주니까 크게 신경 쓸 건 없지만, 이런 게 있다는 건 알아두면 좋아.