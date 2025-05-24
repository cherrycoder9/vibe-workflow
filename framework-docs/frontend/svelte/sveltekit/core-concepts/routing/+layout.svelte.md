모든 페이지에 똑같이 적용되는 레이아웃을 만들고 싶으면 `src/routes/+layout.svelte` 파일을 만들면 돼. 네가 직접 안 만들면 SvelteKit이 기본으로 쓰는 디폴트 레이아웃은 대충 이렇게 생겼어:

```svelte
<script>
	let { children } = $props();
</script>

{@render children()}
```

근데 여기다가 원하는 마크업, 스타일, 동작 아무거나 다 때려 넣을 수 있어. 딱 하나 지켜야 할 건, 페이지 내용물이 들어갈 자리에 `{@render children()}` 태그를 꼭 넣어줘야 한다는 거. 예를 들어 네비게이션 바를 추가해 보자고:

```svelte
<!-- src/routes/+layout.svelte -->
<script lang="ts">
	let { children } = $props();
</script>

<nav>
	<a href="/">홈</a>
	<a href="/about">소개</a>
	<a href="/settings">설정</a>
</nav>

{@render children()}
```

만약 `/`, `/about`, `/settings` 페이지를 만들면...

```svelte
<!-- src/routes/+page.svelte -->
<h1>홈</h1>
```

```svelte
<!-- src/routes/about/+page.svelte -->
<h1>소개</h1>
```

```svelte
<!-- src/routes/settings/+page.svelte -->
<h1>설정</h1>
```

...이렇게 해도 네비게이션 바는 항상 보이고, 세 페이지 사이를 왔다 갔다 해도 `<h1>` 태그 내용만 바뀔 거야.

레이아웃은 중첩도 가능해. 만약 `/settings` 페이지만 있는 게 아니라, `/settings/profile`이나 `/settings/notifications`처럼 하위 페이지들이 있고 얘네들이 공통 서브메뉴를 쓴다고 쳐보자 (실제 예시로는 github.com/settings 같은 거).

이럴 땐 `/settings` 아래 페이지에만 적용되는 레이아웃을 만들 수 있어 (물론 최상위 네비게이션 바가 있는 루트 레이아웃은 그대로 상속받고):

```svelte
<!-- src/routes/settings/+layout.svelte -->
<script lang="ts">
	import type { LayoutProps } from './$types';

	let { data, children }: LayoutProps = $props();
</script>

<h1>설정</h1>

<div class="submenu">
	{#each data.sections as section}
		<a href="/settings/{section.slug}">{section.title}</a>
	{/each}
</div>

{@render children()}
```

`LayoutProps`는 2.16.0 버전에 추가됐어. 그전에는 속성 타입을 일일이 수동으로 적어줘야 했지.

데이터가 어떻게 채워지는지는 바로 다음 섹션에 있는 `+layout.js` 예제 보면 알 수 있을 거야.

기본적으로 각 레이아웃은 자기 위에 있는 레이아웃을 상속받아. 가끔 이게 원치 않는 동작일 수도 있는데, 그럴 땐 고급 레이아웃 기능이 도움이 될 수 있어.

---

**얘 뭐 하는 애냐?**
웹사이트의 '틀'을 만드는 녀석이야. 헤더, 푸터, 네비게이션 바처럼 여러 페이지에서 똑같이 보여야 하는 부분들을 한 곳에 모아놓고 재활용하는 거지. "페이지마다 똑같은 거 또 만들지 말고, 내가 한 방에 처리해 줄게!" 이런 느낌. `{@render children()}` 이라는 주문을 외우면 그 자리에 실제 페이지 내용물이 쏙 들어온다.

**왜 쓰는데?**
1.  **코드 중복 최소화 (DRY 원칙)**: 모든 페이지에 네비게이션 바 넣는다고 복붙하다간 손목 나간다. `+layout.svelte`에 한 번만 만들면 끝. "개발자의 손목은 소중하니까."
2.  **일관된 사용자 경험**: 어느 페이지를 가든 똑같은 헤더, 푸터가 보이니까 사용자가 "어? 이 사이트 맞나?" 하고 헷갈릴 일이 줄어든다.
3.  **유지보수 용이성**: 네비게이션 메뉴 하나 바꾸려고 수십 개 파일 뒤질 필요 없이, `+layout.svelte` 하나만 고치면 땡. "수정은 한 곳에서, 적용은 모든 곳에!"
4.  **계층적 구조**: 루트 레이아웃(전체 적용) 밑에 특정 경로에만 적용되는 서브 레이아웃을 만들 수 있어서, 복잡한 사이트 구조도 깔끔하게 관리 가능. 마치 폴더 정리하듯.

**언제 불려 나오냐?**
해당 레이아웃 파일이 위치한 경로 및 그 하위 경로의 페이지들이 로드될 때마다 적용돼. 예를 들어 `src/routes/+layout.svelte`는 모든 페이지에, `src/routes/settings/+layout.svelte`는 `/settings`와 그 아래 페이지들 (`/settings/profile` 등)에 적용되는 식.

**쓸 때 꿀팁 및 주의사항:**
*   **`{@render children()}`은 생명줄**: 이거 빼먹으면 페이지 내용이 안 나와서 "어? 왜 화면이 하얗지?" 하고 멘붕 온다. "이거 없으면 팥 없는 찐빵, 속 없는 만두."
*   **데이터는 `+layout.js` (또는 `+layout.server.js`)에서**: 레이아웃에 필요한 데이터(예: 사용자 정보, 메뉴 목록)는 짝꿍인 `+layout.js` 파일의 `load` 함수에서 가져와야 해. `+page.js`랑 원리는 비슷.
*   **루트 레이아웃은 기본 제공**: `src/routes/+layout.svelte` 안 만들면 SvelteKit이 아주 심플한 기본 레이아웃을 알아서 넣어주긴 하는데, 보통은 직접 만들게 되지.
*   **상속 끊고 싶으면 고급 레이아웃**: "나는 부모 레이아웃 디자인 따위 필요 없어! 나만의 길을 간다!" 싶으면 `@` 기호를 사용한 고급 레이아웃으로 상속 관계를 리셋할 수 있어. (예: `src/routes/admin/+layout@.svelte` - 이러면 루트 레이아웃 무시)
*   **스타일링 주의**: 레이아웃에 넣은 스타일은 하위 페이지에도 영향을 줄 수 있으니, CSS 스코프(scope)나 전역 스타일 관리에 신경 써야 해. "내 스타일이 니 스타일 침범할 수 있다!"
*   **`$props()` 활용**: Svelte 5부터는 `let { children, data } = $props();` 이런 식으로 프롭스를 받아서 사용한다. 이전 버전이랑 문법이 다르니 주의.