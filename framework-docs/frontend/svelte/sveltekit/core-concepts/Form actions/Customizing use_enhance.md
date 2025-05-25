# Customizing use_enhance

`use:enhance` 커스터마이징하기
동작을 직접 만지고 싶다면, 폼이 제출되기 직전에 실행되는 `SubmitFunction`을 제공하고, 선택적으로 `ActionResult`와 함께 실행되는 콜백을 반환할 수 있어. 만약 콜백을 반환하면, 위에서 언급된 기본 동작은 실행되지 않는다는 점에 유의해. 기본 동작을 다시 쓰고 싶으면 `update` 함수를 호출해야 해. "기본 세팅도 좋지만, 내 입맛대로 바꾸고 싶을 때가 있잖아?"

```svelte
<form
	method="POST"
	use:enhance={({ formElement, formData, action, cancel, submitter }) => {
		// `formElement`는 이 `<form>` 요소를 가리켜.
		// `formData`는 곧 제출될 `FormData` 객체야.
		// `action`은 폼이 전송될 URL이지.
		// `cancel()`을 호출하면 제출을 막을 수 있어.
		// `submitter`는 폼 제출을 유발한 `HTMLElement` (예: 클릭된 버튼)야.

		// 여기가 바로 폼 제출 직전에 실행되는 로직을 넣는 곳!
		// 예를 들어, 로딩 스피너를 보여준다거나, 폼 데이터를 한번 더 검증한다거나.

		return async ({ result, update }) => {
			// `result`는 `ActionResult` 객체야. (서버 액션의 결과)
			// `update`는 이 콜백이 설정되지 않았을 때 실행될 기본 로직을 실행하는 함수야.
			// 여기서 로딩 스피너를 숨기거나, 성공/실패에 따른 UI 변경 등을 처리할 수 있지.
			// 만약 `update()`를 호출하지 않으면, 폼 리셋이나 데이터 무효화 같은 기본 동작은 알아서 안 해줘.
		};
	}}
>
	<!-- 폼 내용 -->
</form>
```

이 함수들을 사용해서 로딩 UI를 보여주거나 숨기는 등의 작업을 할 수 있어. "로딩 중입니다... 뺑뺑이 돌려!"

만약 콜백을 반환한다면, 성공적인 응답 시 모든 데이터를 무효화하지 않으면서 `use:enhance`의 기본 동작 중 일부를 직접 구현해야 할 수도 있어. 이때 `applyAction`을 사용하면 돼:

`src/routes/login/+page.svelte` (원문에서는 `+page`라고만 되어 있지만, Svelte 파일이므로 확장자를 명시)

```svelte
<script lang="ts">
	import { enhance, applyAction } from '$app/forms';
	import { goto } from '$app/navigation'; // goto는 $app/navigation에서 가져와야 해.
	import type { ActionData, PageData } from './$types'; // PageProps 대신 PageData, form 데이터는 ActionData일 가능성이 높음.
	
	export let data: PageData;
	export let form: ActionData; // SvelteKit에서는 form 액션 결과를 이렇게 받음. $props()는 Rune.
</script>

<form
	method="POST"
	use:enhance={({ formElement, formData, action, cancel }) => {
		// 제출 전 로직 (예: 로딩 상태 true)
		return async ({ result }) => {
			// 제출 후 로직 (예: 로딩 상태 false)
			// `result`는 `ActionResult` 객체야.
			if (result.type === 'redirect') {
				goto(result.location); // invalidateAll 없이 리다이렉트만 하고 싶을 때
			} else {
				// 성공, 실패, 에러 시 기본 동작 중 일부(상태 업데이트, 에러 바운더리 렌더링)를 수행
				await applyAction(result);
			}
		};
	}}
>
	<!-- 폼 내용 -->
</form>
```

`applyAction(result)`의 동작은 `result.type`에 따라 달라져:

*   `success`, `failure` — `page.status`를 `result.status`로 설정하고, `form`과 `page.form`을 `result.data`로 업데이트해 (어디서 제출하든 상관없이, `enhance`의 `update`와는 대조적이지).
*   `redirect` — `goto(result.location, { invalidateAll: true })`를 호출해. "성공했으니 다음 페이지로 꺼져... 아니, 이동시켜!"
*   `error` — `result.error`를 사용해 가장 가까운 `+error` 경계를 렌더링해.
모든 경우에 포커스는 리셋될 거야.

---

얘 뭐 하는 애냐?
SvelteKit의 `use:enhance` 기능을 기본 설정 그대로 쓰는 게 아니라, 개발자가 직접 폼 제출 전후의 동작을 세밀하게 제어하고 싶을 때 쓰는 방법이야. "기본 맛도 좋지만, 가끔은 특제 소스를 뿌리고 싶을 때가 있잖아?"

왜 쓰는데? (왜 이런 커스터마이징이 필요한데?)
1.  **세밀한 UX 제어**: 폼 제출 시 로딩 인디케이터를 보여주거나, 버튼을 비활성화하거나, 성공/실패 시 특정 UI 요소만 부드럽게 업데이트하고 싶을 때. "단순히 '됐다/안됐다' 말고, 좀 더 있어 보이게 만들고 싶을 때."
2.  **선택적 데이터 무효화**: 기본 `use:enhance`는 성공 시 `invalidateAll`로 모든 데이터를 새로고침하는데, 이게 필요 없을 때가 있어. 예를 들어, 댓글 하나 달았는데 페이지 전체 데이터를 다시 불러올 필요는 없잖아? 그럴 때 `applyAction`을 쓰거나 `update`를 선택적으로 호출해서 조절하는 거야.
3.  **복잡한 클라이언트 로직 통합**: 폼 제출 데이터에 클라이언트에서만 아는 정보를 추가하거나, 제출 전에 복잡한 유효성 검사를 한 번 더 하고 싶을 때 유용해.

언제 불려 나오냐? (언제 이 커스텀 로직이 실행되냐?)
`use:enhance`에 함수를 전달하면, 그 함수(`SubmitFunction`)는 HTML 폼이 제출되기 직전에 호출돼. 이 함수가 또 다른 비동기 함수(콜백)를 반환하면, 그 콜백은 서버 액션이 처리되고 난 후에 `ActionResult`와 함께 호출돼.

쓸 때 꿀팁 및 주의사항:
*   **콜백 반환 시 기본 동작 증발**: `SubmitFunction`에서 콜백 함수를 `return`하면, `use:enhance`의 편리한 기본 동작들(폼 리셋, `page.form` 업데이트 등)이 자동으로 안 일어나. "네가 직접 한다고 했으니, 이제 네 책임이다!" 이 기본 동작을 원하면 콜백 안에서 `update()` 함수를 명시적으로 호출해야 해.
*   **`update()` vs `applyAction()`**:
    *   `update()`: `use:enhance`의 기본 동작(현재 페이지 액션일 때만 `form` 업데이트, 성공 시 `invalidateAll` 등)을 그대로 실행.
    *   `applyAction(result)`: `result.type`에 따라 선별적으로 동작. `success`/`failure` 시에는 현재 페이지가 아니더라도 `form`과 `page.form`을 업데이트하고, `redirect` 시에는 `invalidateAll: true` 옵션으로 `goto`를 호출. "상황 봐서 골라 쓰는 재미."
*   **`cancel()`**: 제출 직전 함수에서 `cancel()`을 호출하면 폼 제출 자체가 취소돼. "잠깐, 이거 뭔가 이상한데? 제출 보류!"
*   **`goto`는 직접 임포트**: 리다이렉션 처리를 위해 `goto` 함수를 쓰려면 `$app/navigation`에서 직접 가져와야 해. `applyAction`은 내부적으로 `goto`를 쓰지만, 직접 호출할 땐 필요.
*   **에러 처리**: `result.type === 'error'`일 때 `applyAction(result)`는 가장 가까운 `+error.svelte` 파일을 렌더링해줘. 커스텀 에러 핸들링도 가능하지만, 기본 처리가 편할 때도 많지.
*   **타입 주의**: `ActionResult`, `SubmitFunction` 등의 타입은 SvelteKit이 제공하니까, 타입스크립트 쓸 거면 잘 활용해서 안전하게 코딩해. "타입... 그거슨 빛."