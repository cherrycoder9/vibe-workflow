# Named Actions (이름 있는 액션)

한 페이지에 기본 액션 하나만 두는 대신, 필요한 만큼 여러 개의 "이름 있는 액션"을 가질 수 있어.

**src/routes/login/+page.server.js 예시:**
```javascript
import type { Actions } from './$types';

export const actions = {
	login: async (event) => {
		// TODO: 로그인 처리 로직
	},
	register: async (event) => {
		// TODO: 회원가입 처리 로직
	}
} satisfies Actions;
```

이름 있는 액션을 호출하려면, 폼의 `action` 속성에 액션 이름 앞에 `/` 문자를 붙인 쿼리 파라미터를 추가하면 돼.

**src/routes/login/+page.svelte 예시 (같은 페이지 내 액션 호출):**
```html
<form method="POST" action="?/register">
	<!-- 폼 내용 -->
</form>
```

**src/routes/+layout.svelte 예시 (다른 페이지의 액션 호출):**
```html
<form method="POST" action="/login?/register">
	<!-- /login 경로의 register 액션을 호출 -->
</form>
```

`<form>` 태그의 `action` 속성뿐만 아니라, 버튼의 `formaction` 속성을 사용해서 같은 폼 데이터를 부모 `<form>`의 `action`과 다른 액션으로 보낼 수도 있어.

**src/routes/login/+page.svelte 예시 (한 폼, 여러 액션 버튼):**
```html
<form method="POST" action="?/login">
	<label>
		이메일
		<input name="email" type="email">
	</label>
	<label>
		비밀번호
		<input name="password" type="password">
	</label>
	<button>로그인</button> <!-- 이 버튼은 form의 action="?/login"을 따름 -->
	<button formaction="?/register">회원가입</button> <!-- 이 버튼은 "?/register" 액션을 호출 -->
</form>
```

**주의할 점:** 이름 있는 액션과 기본 액션(이름 없는 `default` 액션)을 같이 쓸 수는 없어. 왜냐하면 이름 있는 액션으로 POST 요청을 보냈는데 리다이렉트가 안 되면, 그 액션 이름이 URL의 쿼리 파라미터에 계속 남아있게 돼. 그러면 그 다음에 기본 액션으로 POST를 보내려고 해도, 이전에 남아있던 이름 있는 액션으로 요청이 가버리는 문제가 생기거든. "주소창에 꼬리표 남기면 다음 손님도 그 꼬리표 보고 들어온다 이거야."

---

**얘 뭐 하는 애냐?**
SvelteKit에서 한 페이지 안에 여러 종류의 폼 제출(POST 요청)을 처리할 수 있게 해주는 기능이야. 각 폼 제출마다 고유한 이름을 붙여서 구분하는 거지. "로그인 폼, 회원가입 폼, 비밀번호 찾기 폼... 한 페이지에서 다 처리하고 싶을 때 쓰는 거."

**왜 쓰는데? (목적)**
1.  **코드 재사용 및 정리**: 관련된 여러 기능을 한 `+page.server.js` 파일 안에 묶어서 관리할 수 있게 해줘. 예를 들어 로그인, 회원가입, 비밀번호 재설정 기능을 `/auth`라는 한 경로에서 모두 처리 가능. "파일 여러 개 만들 필요 없이 한 방에 해결!"
2.  **UI/UX 개선**: 사용자가 여러 작업을 하기 위해 페이지를 계속 옮겨 다닐 필요 없이, 한 페이지 내에서 다양한 폼 제출을 할 수 있게 만들어. "이 페이지 저 페이지 왔다 갔다 안 해도 되니 편하잖아?"
3.  **명확한 서버 로직 분리**: 각 액션마다 이름이 있으니, 서버 사이드 코드(`+page.server.js`)에서 어떤 폼 요청인지 명확하게 구분하고 각각 다른 로직을 실행할 수 있어.

**언제 불려 나오냐? (언제 사용되냐?)**
*   사용자가 웹페이지에서 `<form method="POST">`를 통해 데이터를 서버로 제출할 때.
*   폼의 `action` 속성이나 버튼의 `formaction` 속성에 `?/액션이름` 형식이 지정되어 있을 때, SvelteKit은 해당 `+page.server.js` 파일에서 `export const actions` 객체 안에 정의된 `액션이름`에 해당하는 함수를 찾아 실행해.

**쓸 때 꿀팁 및 주의사항:**
*   **액션 이름은 `/`로 시작하는 쿼리 파라미터**: 폼의 `action` 속성값은 `?/login`이나 `?/register`처럼 물음표 뒤에 슬래시, 그리고 액션 이름을 적어야 해. "이거 안 지키면 SvelteKit이 못 알아먹는다."
*   **기본 액션이랑 같이 못 씀**: 한 파일에 `export const actions = { default: ..., login: ... }` 이런 식으로 기본 액션과 이름 있는 액션을 섞어 쓰면 에러 나거나 의도치 않게 동작할 수 있어. "하나만 골라, 욕심부리지 말고."
*   **`formaction` 활용**: 한 폼 안에 여러 제출 버튼(예: "저장", "임시저장", "삭제")을 두고 각 버튼마다 다른 액션을 연결하고 싶을 때 `formaction` 속성이 아주 유용해. "버튼마다 다른 길을 가게 하라."
*   **리다이렉트 생활화**: 액션 처리 후에는 보통 `throw redirect(303, '/다음페이지')` 같은 걸로 다른 페이지로 보내주는 게 좋아. 그래야 사용자가 새로고침했을 때 폼이 다시 제출되는 걸 막을 수 있고, URL에 남아있는 `?/액션이름` 쿼리 파라미터도 깔끔하게 정리돼. "뒷정리는 확실하게!"
*   **타입스크립트**: `import type { Actions } from './$types';`를 쓰고 `satisfies Actions`를 붙여주면 액션 정의할 때 타입 체크의 은총을 받을 수 있어. 오타나 잘못된 인자 사용을 줄여주니 웬만하면 쓰자.