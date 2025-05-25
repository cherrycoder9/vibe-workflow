# SvelteKit Form Actions

`+page.server.js` 파일은 `actions`라는 것을 `export` 할 수 있는데, 이걸 사용하면 `<form>` 태그를 통해 서버로 데이터를 `POST` 방식으로 보낼 수 있어.

`<form>`을 사용할 때, 클라이언트 측 자바스크립트는 필수가 아니야. 하지만 사용자 경험을 최상으로 끌어올리고 싶다면, 자바스크립트를 사용해서 폼 상호작용을 점진적으로 향상시키는 건 아주 쉬워. "노 자바스크립트도 OK, 자바스크립트 곁들이면 더 좋고!"

**기본 액션 (Default Actions)**
가장 간단한 경우, 페이지는 `default` 액션을 선언해:

**src/routes/login/+page.server.js 예시:**
```javascript
import type { Actions } from './$types';

export const actions = {
	default: async (event) => {
		// TODO: 여기에 사용자 로그인 처리 로직 구현
		// event 객체를 통해 폼 데이터, 쿠키, 요청 정보 등에 접근 가능
		// 예: const data = await event.request.formData();
		//     const email = data.get('email');
	}
} satisfies Actions;
```

이 액션을 `/login` 페이지에서 호출하려면, 그냥 `<form>` 태그만 추가하면 돼. 자바스크립트 따위 필요 없어:

**src/routes/login/+page.svelte 예시:**
```html
<form method="POST">
	<label>
		이메일
		<input name="email" type="email">
	</label>
	<label>
		비밀번호
		<input name="password" type="password">
	</label>
	<button>로그인</button>
</form>
```

만약 누군가 저 버튼을 클릭하면, 브라우저는 폼 데이터를 `POST` 요청으로 서버에 보내고, `default` 액션이 실행될 거야.

액션은 항상 `POST` 요청을 사용해. 왜냐하면 `GET` 요청은 절대로 부수 효과(side-effects, 예를 들어 데이터 변경 같은 거)를 가져서는 안 되거든. "GET은 조회만, POST는 작업 처리!" 이건 웹의 기본 상식이지.

다른 페이지에서도 이 액션을 호출할 수 있어 (예를 들어, 최상위 레이아웃의 네비게이션 바에 로그인 위젯이 있는 경우). `<form>` 태그에 `action` 속성을 추가하고, 해당 액션이 있는 페이지 경로를 지정하면 돼:

**src/routes/+layout.svelte 예시:**
```html
<form method="POST" action="/login">
	<!-- 폼 내용물 -->
</form>
```
이렇게 하면 `/login` 경로의 `default` 액션으로 폼 데이터가 전송돼. "어디서든 로그인 폼을 부를 수 있다, 이 말씀!"

---

**얘 뭐 하는 애냐?**
HTML `<form>` 태그를 통해 사용자가 입력한 데이터를 서버로 안전하고 쉽게 전송하고, 서버에서 그 데이터를 받아서 처리(예: 로그인, 회원가입, 글쓰기 등)할 수 있게 해주는 SvelteKit의 기능이야. "폼 데이터, 서버로 직행 KTX 태워주는 기능!"

**왜 쓰는데? (목적)**
1.  **서버 사이드 로직 간편화**: 클라이언트에서 `fetch` 같은 거 복잡하게 안 쓰고, 그냥 `<form>` 태그만으로 서버랑 데이터 주고받을 수 있게 해줘. "복잡한 API 호출 코드? 이제 안녕~"
2.  **점진적 향상 (Progressive Enhancement)**: 자바스크립트 없이도 기본적인 폼 제출 기능이 동작하고, 원한다면 자바스크립트를 추가해서 더 부드러운 사용자 경험(예: 페이지 새로고침 없이 결과 보여주기)을 만들 수 있어. "자바스크립트 없어도 굴러가고, 있으면 더 잘 굴러간다."
3.  **보안 및 데이터 무결성**: `POST` 요청만 사용해서 데이터 변경 작업을 처리하므로, `GET` 요청으로 인한 의도치 않은 부수 효과를 방지해. CSRF(Cross-Site Request Forgery) 보호 기능도 내장되어 있는 경우가 많아. "안전벨트는 기본 장착!"

**언제 불려 나오냐? (호출 시점/동작 방식)**
*   사용자가 `<form method="POST">`로 정의된 폼의 제출 버튼(예: `<button type="submit">`)을 클릭할 때.
*   폼에 `action` 속성이 없으면 현재 페이지의 `+page.server.js`에 정의된 `default` 액션이 호출돼.
*   폼에 `action="/some/path"`처럼 경로가 지정되어 있으면, `/some/path`에 해당하는 `+page.server.js`의 `default` 액션이 호출돼. (만약 `action="/some/path?/actionName"`처럼 `?` 뒤에 이름이 붙으면 해당 이름의 '명명된 액션'이 호출됨).
*   서버의 해당 액션 함수(`async (event) => { ... }`)가 실행되고, 폼 데이터는 `event.request.formData()`를 통해 접근할 수 있어.

**쓸 때 꿀팁 및 주의사항:**
*   **`+page.server.js`에만 정의**: 액션은 서버에서만 돌아가야 하므로, 반드시 `+page.server.js` 파일 안에 만들어야 해. `+page.js`에 만들면 에러 난다. "서버 일은 서버에서!"
*   **`POST`는 국룰**: 액션은 무조건 `POST` 요청이야. `<form method="GET">`으로 액션 못 불러.
*   **기본 액션 vs 명명된 액션**: 한 페이지에 여러 종류의 폼 제출을 처리하고 싶으면, `default` 액션 말고 `export const actions = { login: async () => {}, register: async () => {} }`처럼 이름을 붙인 "명명된 액션"을 쓸 수 있어. 이때 폼에서는 `<form method="POST" action="?/login">`처럼 `action` 속성에 `?`와 함께 액션 이름을 지정해. "이름표 붙여주면 알아서 찾아간다."
*   **점진적 향상 활용**: SvelteKit의 `use:enhance` 디렉티브를 `<form>`에 사용하면, 자바스크립트가 활성화된 환경에서는 페이지 전체 새로고침 없이 폼을 제출하고 결과를 업데이트할 수 있어. 사용자 경험이 훨씬 부드러워지지. "JS 한 스푼이면 UX가 달라진다."
*   **결과 처리 및 리다이렉트**: 액션 함수 내에서 작업 성공/실패에 따라 특정 페이지로 리다이렉트 시키거나 (예: `throw redirect(303, '/dashboard')`), 폼 페이지에 에러 메시지나 성공 메시지를 전달할 수 있어. `return { success: true }`나 `return fail(400, { error: 'Invalid input' })` 같은 헬퍼 함수를 활용해. "처리 결과는 확실하게 알려주자."
*   **파일 업로드**: 파일 업로드도 액션을 통해 처리할 수 있어. `event.request.formData()`로 파일 데이터에 접근하면 돼. 단, `<form enctype="multipart/form-data">` 설정은 필수!