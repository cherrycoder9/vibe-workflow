라우트 데이터 로딩 (Loading data for a route)

"Shallow routing"이라고, 페이지 전체를 새로고침하지 않고 현재 페이지 안에서 다른 페이지(`+page.svelte`) 내용을 쏙 보여주고 싶을 때가 있어. 예를 들어 사진 썸네일 클릭하면, 새 페이지로 넘어가는 게 아니라 지금 화면에 바로 상세 정보 팝업(모달) 띄우는 거. "페이지 이동은 안 했지만, 새 콘텐츠는 보여줄게!" 뭐 이런 느낌이지.

이게 되려면, 그 새로 보여줄 `+page.svelte`가 필요로 하는 데이터를 미리 불러와야 해. 이걸 편하게 해주는 게 바로 `<a>` 태그의 클릭 핸들러 안에서 `preloadData` 함수를 쓰는 거야. 만약 그 `<a>` 태그나 그 부모 태그에 `data-sveltekit-preload-data` 속성이 걸려있으면, 데이터는 이미 요청됐거나 로딩 중일 거고, `preloadData`는 그 요청을 재활용해서 낼름 가져오지. "어, 이미 누가 시켜놨네? 그럼 그거 먹자!" 하는 거지.

```svelte
<!-- src/routes/photos/+page.svelte -->
<script lang="ts">
	import { preloadData, pushState, goto } from '$app/navigation';
	import { page } from '$app/state';
	import Modal from './Modal.svelte';
	import PhotoPage from './[id]/+page.svelte'; // 상세 페이지 컴포넌트

	let { data } = $props(); // 현재 페이지(/photos)의 데이터
</script>

{#each data.thumbnails as thumbnail}
	<a
		href="/photos/{thumbnail.id}"
		onclick={async (e) => {
			// 화면이 너무 작거나, 새 탭/창으로 열려는 거면 그냥 일반 링크처럼 행동해!
			if (innerWidth < 640 || e.shiftKey || e.metaKey || e.ctrlKey) return;

			// 기본 링크 이동 막기
			e.preventDefault();

			const { href } = e.currentTarget; // 클릭한 링크의 주소

			// load 함수 실행 (또는 data-sveltekit-preload-data 덕분에 이미 실행 중인 load 함수의 결과 가져오기)
			const result = await preloadData(href);

			if (result.type === 'loaded' && result.status === 200) {
				// 데이터 로딩 성공! URL 바꾸고, 로드한 데이터를 history.state에 저장
				pushState(href, { selected: result.data });
			} else {
				// 뭔가 잘못됨! 그냥 해당 주소로 페이지 이동 시도
				goto(href);
			}
		}}
	>
		<img alt={thumbnail.alt} src={thumbnail.src} />
	</a>
{/each}

{#if $page.state.selected}
	<Modal onclose={() => history.back()}>
		{/* SvelteKit이 페이지 이동 시 하듯이, 페이지 데이터를 +page.svelte 컴포넌트에 전달 */}
		<PhotoPage data={$page.state.selected} />
	</Modal>
{/if}
```

---

**얘 뭐 하는 애냐?**
`preloadData(href)`는 이름 그대로 `href`에 해당하는 라우트의 `load` 함수를 미리 실행시켜서 데이터를 가져오는 녀석이야. 페이지 전체를 갈아엎는 풀 내비게이션(full navigation) 없이, 현재 페이지에서 다른 페이지의 데이터만 쏙 빼오고 싶을 때 쓰는 거지. "야, 너네 집 냉장고에 뭐 있는지 문만 열어서 보고 올게!" 이런 느낌.

**왜 쓰는데?**
1.  **부드러운 UX (Shallow Routing):** 썸네일 클릭 시 모달 창으로 상세 정보 보여줄 때, 페이지 전체가 깜빡이며 새로고침되면 사용자 경험이 별로잖아? `preloadData`로 데이터만 샥 가져와서 현재 페이지에 띄워주면 훨씬 부드럽지. "손님, 화면은 그대로 두고 내용물만 바꿔드릴게요."
2.  **성능 최적화 (미리 로딩):** `data-sveltekit-preload-data` 속성이랑 같이 쓰면, 사용자가 마우스를 올리거나 클릭하기 전에 이미 데이터 로딩을 시작할 수 있어. 그래서 클릭했을 때 거의 바로 내용을 보여줄 수 있지. "니가 누를 줄 알고 미리 준비해놨지."
3.  **코드 재활용:** 이미 만들어둔 상세 페이지(`[id]/+page.svelte`)와 그 페이지의 `load` 함수를 그대로 재활용해서 모달에서도 쓸 수 있으니 개발이 편해. "만들어둔 거 또 써먹자, 개꿀!"

**언제 불려 나오냐?**
주로 `<a>` 태그의 `onclick` 이벤트 핸들러 안에서, 사용자가 링크를 클릭했지만 실제 페이지 이동은 막고(`event.preventDefault()`) 해당 링크의 데이터만 가져오고 싶을 때 호출해.

**쓸 때 꿀팁 및 주의사항:**
*   **`data-sveltekit-preload-data`는 짝꿍:** `<a>` 태그에 `data-sveltekit-preload-data="hover"` (마우스 올렸을 때)나 `data-sveltekit-preload-data="tap"` (터치했을 때) 같은 걸 같이 써주면 `preloadData`가 더 빛을 발해. 사용자가 행동하기 전에 미리 데이터를 가져오니까.
*   **에러 처리는 필수:** `preloadData`가 실패할 수도 있잖아? (네트워크 에러, 서버 에러 등). 예제 코드처럼 `result.type`이랑 `result.status` 확인해서 실패하면 `goto(href)`로 그냥 페이지 이동시키는 식으로 fallback 처리해주는 게 좋아. "플랜 B는 항상 준비해둬야지."
*   **`pushState`로 URL 관리:** 데이터 가져온 후에는 `pushState(href, { dataToPass })`로 브라우저 히스토리 스택에 상태를 저장하고 URL도 바꿔줘야 사용자가 뒤로 가기/앞으로 가기 했을 때 자연스럽게 동작해. `$page.state`로 이 데이터에 접근할 수 있고. "왔던 길 기억은 해야 할 거 아냐."
*   **모든 상황에 맞는 건 아님:** 화면 일부만 바꾸는 게 아니라 아예 다른 성격의 페이지로 가는 거면 그냥 일반 내비게이션 쓰는 게 나아. "모든 걸 모달로 해결하려 들면 그게 더 복잡해진다."
*   **조건부 실행:** 예제처럼 화면 너비나 특수키 입력(새 탭으로 열기 등)을 확인해서, `preloadData`를 쓸지 말지 결정하는 로직을 넣어주는 게 사용자 경험에 좋아. "무지성 `preloadData`는 지양하자."