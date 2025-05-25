# Custom Event Listener for Progressive Enhancement

커스텀 이벤트 리스너
`use:enhance` 없이, `<form>`에 일반적인 이벤트 리스너를 달아서 직접 점진적 향상을 구현할 수도 있어:

`src/routes/login/+page.svelte` (원문에서는 `+page`라고만 되어 있지만, Svelte 파일이므로 확장자를 명시)

```svelte
<script lang="ts">
	import { invalidateAll, goto } from '$app/navigation'; // goto는 이 예제에서 직접 사용되진 않지만, 일반적으로 필요할 수 있는 유틸리티
	import { applyAction, deserialize } from '$app/forms';
	import type { PageData } from './$types'; // 원문은 PageProps지만, SvelteKit에서는 PageData가 일반적. form을 받으려면 ActionData도 고려.
	import type { ActionResult } from '@sveltejs/kit';

	// export let form: PageData['form']; // $props()는 Rune 문법. 일반적인 props 전달 방식은 이렇거나, export let data: PageData; 후 data.form 접근.
	// 원문 코드 스니펫을 최대한 따르기 위해 아래와 같이 유지.
	let { form }: { form?: any } = $props(); // PageProps가 $props()와 함께 쓰이는 것은 Rune 문맥. 타입은 실제 데이터 구조에 맞게 조정 필요.

	async function handleSubmit(event: SubmitEvent & { currentTarget: EventTarget & HTMLFormElement}) {
		event.preventDefault(); // 기본 폼 제출(페이지 새로고침)을 막는다.
		const data = new FormData(event.currentTarget); // 폼 데이터를 가져온다.

		const response = await fetch(event.currentTarget.action, { // 폼의 action URL로 POST 요청
			method: 'POST',
			body: data
		});

		// 응답 텍스트를 SvelteKit의 ActionResult 형태로 변환.
		// Date, BigInt 같은 타입도 제대로 처리하려면 JSON.parse() 대신 deserialize 사용 필수.
		const result: ActionResult = deserialize(await response.text());

		if (result.type === 'success') {
			// 성공적인 업데이트 후 모든 `load` 함수를 다시 실행해서 페이지 데이터를 최신으로.
			await invalidateAll();
		}

		// SvelteKit이 알아서 $page.form, $page.status 업데이트하고,
		// 리다이렉션이나 에러 페이지 렌더링 등을 처리하게 함.
		applyAction(result);
	}
</script>

<form method="POST" on:submit={handleSubmit}> {/* submit 이벤트 발생 시 handleSubmit 함수 실행 */}
	<!-- 폼 내용 -->
</form>
```

폼 액션은 `load` 함수처럼 `Date`나 `BigInt` 객체 반환도 지원하기 때문에, 응답을 처리하기 전에 `$app/forms`에서 제공하는 해당 메서드(`deserialize`)를 사용해서 역직렬화해야 한다는 점에 유의해. `JSON.parse()`만으로는 충분하지 않아. "그냥 JSON.parse() 쓰면 데이터 깨질 수도 있다, 이 말이야."

만약 `+page.server.js` 파일 옆에 `+server.js` 파일도 같이 있다면, `fetch` 요청은 기본적으로 `+server.js`로 라우팅될 거야. 대신 `+page.server.js`에 있는 액션으로 POST 요청을 보내려면, 커스텀 `x-sveltekit-action` 헤더를 사용해야 해:

```javascript
const response = await fetch(this.action, { // this.action 대신 event.currentTarget.action 사용 권장
	method: 'POST',
	body: data,
	headers: {
		'x-sveltekit-action': 'true' // "이거 페이지 액션으로 보내주세요!" 라는 신호
	}
});
```

---

얘 뭐 하는 애냐?
SvelteKit에서 `use:enhance`라는 편리한 기능 대신, 개발자가 직접 `<form>`의 `submit` 이벤트를 잡아서 폼 제출 로직을 짜는 방법이야. `fetch` API 써서 서버에 데이터 보내고, 응답 받아서 처리하는 전 과정을 수동으로 컨트롤하는 거지. "DIY 폼 핸들링이라고 보면 된다."

왜 쓰는데?
1.  **세밀한 제어**: `use:enhance`가 해주는 기본 동작 말고 뭔가 더 복잡한 걸 하고 싶을 때. 예를 들어, 특정 조건에 따라 요청을 다르게 보내거나, 응답 데이터를 특별하게 가공하거나, 로딩 상태를 아주 디테일하게 관리하고 싶을 때 유용해. "내 손으로 다 만져야 직성이 풀리는 사람들에게 딱이지."
2.  **특정 라이브러리 연동**: 외부 폼 관리 라이브러리나 특정 UI 컴포넌트와 깊게 연동해야 할 때, 직접 이벤트 리스너를 다루는 게 더 편할 수 있어.
3.  **학습 목적**: SvelteKit이 내부적으로 폼을 어떻게 처리하는지, 점진적 향상이 어떤 원리로 돌아가는지 깊이 이해하고 싶을 때 직접 구현해보는 것만큼 좋은 게 없지. "백문이 불여일타."

언제 불려 나오냐?
사용자가 `<form method="POST" on:submit={handleSubmit}>` 이렇게 설정된 폼의 제출 버튼을 누르면, `handleSubmit` 함수가 호출돼. 여기서 `event.preventDefault()`로 기본 페이지 이동을 막고, 우리가 짠 로직이 실행되는 거야.

쓸 때 꿀팁 및 주의사항:
*   **`event.preventDefault()`는 필수**: 이거 안 하면 그냥 옛날 방식대로 페이지 전체가 새로고침되니까, 점진적 향상의 의미가 없어져. "시작부터 틀리면 다 망하는 거다."
*   **`FormData` 활용**: 폼 데이터 긁어올 때 `new FormData(event.currentTarget)` 쓰면 편해.
*   **`deserialize` 꼭 쓰기**: 서버 액션에서 `Date`나 `BigInt` 같은 거 반환하면 `JSON.parse()`로는 제대로 못 받아. SvelteKit이 주는 `deserialize`를 써야 안전해. "데이터 타입, 소중하니까."
*   **`applyAction`으로 마무리**: 서버 응답(`result`)을 받아서 `applyAction(result)` 호출하면, SvelteKit이 알아서 `$page.form` 업데이트, 리다이렉트 처리, 에러 표시 등을 해줘. 우리가 직접 할 일을 줄여주지.
*   **`invalidateAll()`로 데이터 동기화**: 폼 제출 성공 후 화면에 보이는 데이터를 최신으로 바꾸려면 `await invalidateAll()`을 호출해서 관련된 `load` 함수들을 다시 실행시켜야 해.
*   **`x-sveltekit-action` 헤더**: 만약 같은 URL 경로에 `+page.server.js`의 액션이랑 `+server.js`의 API 엔드포인트가 둘 다 있다면, `fetch` 요청 보낼 때 헤더에 `'x-sveltekit-action': 'true'`를 넣어줘야 페이지 액션으로 제대로 가. 안 그러면 `+server.js`가 요청을 가로챌 수 있어. "택배 보낼 때 주소 정확히 적는 거랑 똑같다."
*   **에러 처리**: `fetch` 요청이나 응답 처리 과정에서 네트워크 에러, 서버 에러 등이 터질 수 있으니 `try...catch`로 잘 감싸서 사용자에게 적절한 피드백을 줘야 해. "예외 처리는 선택이 아니라 필수."