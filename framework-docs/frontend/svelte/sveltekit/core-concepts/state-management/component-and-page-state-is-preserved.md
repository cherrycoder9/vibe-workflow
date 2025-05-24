자, "Component and page state is preserved"에 대해 알아보자. 번역하면 "컴포넌트랑 페이지 상태가 보존된다"는 뜻이야.

**컴포넌트와 페이지 상태 보존**

네가 만든 앱 여기저기를 돌아다닐 때, 스벨트킷은 기존에 있던 레이아웃이나 페이지 컴포넌트를 재활용해. 예를 들어 이런 라우트가 있다고 쳐보자:

```html
<!-- src/routes/blog/[slug]/+page -->
<script lang="ts">
	import type { PageProps } from './$types';

	let { data }: PageProps = $props();

	// 이 코드는 버그가 있다!
	const wordCount = data.content.split(' ').length; // 단어 수 계산
	const estimatedReadingTime = wordCount / 250; // 예상 독서 시간
</script>

<header>
	<h1>{data.title}</h1>
	<p>읽는데 {Math.round(estimatedReadingTime)}분 걸림</p>
</header>

<div>{@html data.content}</div>
```

이 상태에서 `/blog/짧은글`에서 `/blog/긴글`로 이동해도, 레이아웃, 페이지, 그리고 그 안에 있는 다른 컴포넌트들이 파괴됐다가 다시 만들어지는 게 아니야. 대신 `data` 프롭(prop) (결과적으로 `data.title`이랑 `data.content`도)만 업데이트될 뿐이지. 이건 다른 스벨트 컴포넌트들이 업데이트되는 방식이랑 똑같아. 그리고 중요한 건, 코드가 다시 실행되는 게 아니기 때문에 `onMount`나 `onDestroy` 같은 생명주기 함수들도 다시 실행되지 않고, `estimatedReadingTime`도 재계산되지 않아. "어? 나 분명 페이지 옮겼는데 왜 계산이 그대로지?" 싶을 수 있다는 거지.

이 문제를 해결하려면, 값을 반응형(reactive)으로 만들어야 해:

```html
<!-- src/routes/blog/[slug]/+page -->
<script lang="ts">
	import type { PageProps } from './$types';

	let { data }: PageProps = $props();

	// $derived를 써서 data.content가 바뀔 때마다 wordCount도 자동으로 다시 계산되게 함
	let wordCount = $derived(data.content.split(' ').length);
	// wordCount가 바뀌면 estimatedReadingTime도 자동으로 다시 계산됨
	let estimatedReadingTime = $derived(wordCount / 250);
</script>
```

만약 `onMount`나 `onDestroy`에 있는 코드가 페이지 이동 후에도 다시 실행되어야 한다면, 각각 `afterNavigate`랑 `beforeNavigate` 함수를 쓰면 돼.

이렇게 컴포넌트를 재활용하면 사이드바 스크롤 위치 같은 게 그대로 유지되고, 값이 바뀔 때 애니메이션 효과도 쉽게 넣을 수 있다는 장점이 있어. 그래도 "아, 난 진짜 이 컴포넌트 완전히 부쉈다가 새로 만들고 싶은데?" 싶을 땐, 이런 패턴을 쓰면 돼:

```html
<script>
	import { page } from '$app/state';
</script>

{#key page.url.pathname}
	<BlogPost title={data.title} content={data.content} />
{/key}
```
`{#key ...}` 블록은 `page.url.pathname` (현재 페이지 주소)이 바뀔 때마다 그 안의 내용을 강제로 파괴하고 다시 만들어줘.

---

**얘 뭐 하는 애냐?**
스벨트킷이 페이지 이동할 때 기존 컴포넌트를 최대한 재활용해서 성능도 챙기고 사용자 경험도 부드럽게 하려는 똑똑한 전략이야. 마치 이사 갈 때 쓰던 가구 버리고 새로 사는 게 아니라, 쓸만한 건 그대로 가져가서 배치만 살짝 바꾸는 느낌이지. "낭비는 줄이고, 효율은 올리고!"

**왜 쓰는데?**
1.  **성능 최적화:** 컴포넌트를 매번 부수고 새로 만드는 것보다 있는 거 업데이트하는 게 훨씬 빨라. 로딩 시간 줄어들고 앱이 쌩쌩하게 느껴지지. "로딩창 그만 보고 싶다!"는 유저들의 염원을 담았달까.
2.  **상태 유지:** 사이드바 스크롤 위치, 입력 폼에 쓰던 내용, 열려있던 드롭다운 메뉴 같은 자잘한 상태들이 페이지 이동해도 그대로 유지돼. 사용자가 "아까 하던 거 어디 갔어!" 하고 빡치는 일을 막아줘.
3.  **부드러운 전환 효과:** 컴포넌트가 유지되니까 값 변경에 따른 애니메이션 같은 걸 자연스럽게 넣기 좋아. 페이지가 뚝뚝 끊기는 느낌 대신 스르륵 바뀌는 고급진 경험을 줄 수 있지.

**언제 불려 나오냐?**
스벨트킷 앱 내에서 같은 레이아웃 구조를 공유하는 페이지들 사이를 이동할 때 기본적으로 이렇게 동작해. 예를 들어 `/blog/post-1`에서 `/blog/post-2`로 갈 때, `[slug]` 부분만 바뀌고 나머지 레이아웃이나 페이지 컴포넌트 구조가 같다면 재활용 대상이 되는 거야.

**쓸 때 꿀팁 및 주의사항:**
*   **반응성! 반응성이 생명이다! (`$derived` 또는 `$:`)**: 페이지 이동 시 `data` 프롭만 바뀌고 `<script>` 블록 코드가 다시 실행되지 않으니까, `data`에 의존하는 계산값들은 반드시 `$derived` (룬즈 모드)나 `$: ` (일반 스벨트)를 써서 반응형으로 만들어야 해. 안 그러면 "데이터는 바뀌었는데 왜 화면은 그대로죠?" 버그 파티 열린다.
*   **`onMount` / `onDestroy`는 한 번만 (기본적으로):** 컴포넌트가 재활용되면 얘네는 처음 마운트될 때랑 진짜 파괴될 때만 실행돼. 페이지 이동할 때마다 뭔가 하고 싶으면 `afterNavigate` (이동 후) / `beforeNavigate` (이동 전) 훅을 써야 해. "이사 안 갔는데 입주 청소 또 할 필요 없잖아?"
*   **강제 재마운트는 `{#key page.url.pathname}`:** 정말 어쩔 수 없이 컴포넌트를 완전히 새로고침해야 한다면, `{#key page.url.pathname}`으로 감싸는 트릭을 써. URL 경로가 바뀔 때마다 키 값이 달라져서 내부 컴포넌트가 파괴되고 재생성돼. "정 안되면 리셋 버튼 누르는 심정으로..."
*   **레이아웃 컴포넌트도 마찬가지:** 이 원칙은 페이지 컴포넌트뿐만 아니라 그걸 감싸는 레이아웃 컴포넌트에도 똑같이 적용돼.
*   **데이터 로딩 함수 (`load`)는 다시 실행됨:** 착각하면 안 되는 게, 컴포넌트 자체는 재활용돼도 페이지 이동 시 해당 페이지의 `load` 함수는 다시 실행돼서 새로운 `data`를 가져와. 그래서 `data` 프롭이 업데이트되는 거야. "밥은 새로 차려주는데, 식탁이랑 그릇은 그대로 쓴다."