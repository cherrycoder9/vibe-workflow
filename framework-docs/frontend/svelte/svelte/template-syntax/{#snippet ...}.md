```
{#snippet ...}
{#snippet 이름()}...{/snippet}
{#snippet 이름(인자1, 인자2, 인자N)}...{/snippet}
```

스니펫(Snippets)과 렌더 태그(`{@render ...}`)는 컴포넌트 안에서 재사용 가능한 마크업 덩어리를 만드는 방법입니다. 이런 식으로 코드 복붙하며 고통받는 대신에...

```svelte
{#each images as image}
	{#if image.href}
		<a href={image.href}>
			<figure>
				<img src={image.src} alt={image.caption} width={image.width} height={image.height} />
				<figcaption>{image.caption}</figcaption>
			</figure>
		</a>
	{:else}
		<figure>
			<img src={image.src} alt={image.caption} width={image.width} height={image.height} />
			<figcaption>{image.caption}</figcaption>
		</figure>
	{/if}
{/each}
```

...이렇게 깔끔하게 쓸 수 있다는 거죠:

```svelte
{#snippet figure(image)}
	<figure>
		<img src={image.src} alt={image.caption} width={image.width} height={image.height} />
		<figcaption>{image.caption}</figcaption>
	</figure>
{/snippet}

{#each images as image}
	{#if image.href}
		<a href={image.href}>
			{@render figure(image)}
		</a>
	{:else}
		{@render figure(image)}
	{/if}
{/each}
```

함수처럼 스니펫도 여러 개의 파라미터(인자)를 받을 수 있고, 기본값을 정해줄 수도 있고, 각 파라미터를 구조 분해 할당으로 받을 수도 있습니다. 하지만 나머지 파라미터(`...rest`)는 못 씁니다. "딱 정해진 재료만 받는다, 이거예요."

**스니펫 스코프 (얘가 어디까지 힘쓰나)**
스니펫은 컴포넌트 안이라면 어디든 선언할 수 있습니다. `<script>` 태그나 `{#each ...}` 블록처럼 자기 바깥에 선언된 값도 가져다 쓸 수 있고요 (데모 보시죠)...

```svelte
<script>
	let { message = `만나서 아주 그냥 반갑구만, 반가워요!` } = $props();
</script>

{#snippet hello(name)}
	<p>안녕 {name}! {message}!</p>
{/snippet}

{@render hello('앨리스')}
{@render hello('밥')}
```

...그리고 같은 렉시컬 스코프(쉽게 말해, 같은 울타리 안의 형제들이나 그 형제들의 자식들)에 있는 모든 놈들에게 '보입니다':

```svelte
<div>
	{#snippet x()}
		{#snippet y()}...{/snippet}

		<!-- 이건 문제없음 -->
		{@render y()}
	{/snippet}

	<!-- 이건 에러! `y`는 이 구역 담당이 아님 -->
	{@render y()}
</div>

<!-- 이것도 에러! `x`도 이 구역 담당이 아님 -->
{@render x()}
```

스니펫끼리 서로 불러오거나, 심지어 자기가 자기를 부르는 것도 가능합니다 (데모 참고): "내가 나를 소환한다! 무한... 아니, 재귀 호출!"

```svelte
{#snippet blastoff()}
	<span>🚀</span>
{/snippet}

{#snippet countdown(n)}
	{#if n > 0}
		<span>{n}...</span>
		{@render countdown(n - 1)}
	{:else}
		{@render blastoff()}
	{/if}
{/snippet}

{@render countdown(10)}
```

**컴포넌트에 스니펫 넘겨주기**
**명시적 props (대놓고 "이거 써!" 하고 주기)**
템플릿 안에서 스니펫은 다른 변수랑 똑같은 '값' 취급을 받습니다. 그래서 컴포넌트에 props로 휙 넘겨줄 수 있죠 (데모 참고): "자, 이 디자인 틀 줄 테니 알아서 채워 넣어."

```svelte
<script>
	import Table from './Table.svelte';

	const fruits = [
		{ name: '사과', qty: 5, price: 2 },
		{ name: '바나나', qty: 10, price: 1 },
		{ name: '체리', qty: 20, price: 0.5 }
	];
</script>

{#snippet header()}
	<th>과일</th>
	<th>수량</th>
	<th>가격</th>
	<th>총액</th>
{/snippet}

{#snippet row(d)}
	<td>{d.name}</td>
	<td>{d.qty}</td>
	<td>{d.price}</td>
	<td>{d.qty * d.price}</td>
{/snippet}

<Table data={fruits} {header} {row} />
```

데이터 대신 '컨텐츠 조각'을 컴포넌트에 넘겨준다고 생각하면 쉽습니다. 웹 컴포넌트의 슬롯(slot)이랑 비슷한 개념인데, 이게 더 쎄요.

**암묵적 props (슬쩍 끼워 넣으면 알아서 인식)**
개발자 편하라고, 컴포넌트 태그 바로 안쪽에 선언한 스니펫은 그 컴포넌트의 props로 알아서 등록됩니다 (데모 참고): "따로 `header={header}` 안 해도 `<Table>` 안쪽에 `{#snippet header}` 만들면 끝. 개이득."

```svelte
<!-- 이건 위에꺼랑 똑같이 돌아감 -->
<Table data={fruits}>
	{#snippet header()}
		<th>과일</th>
		<th>수량</th>
		<th>가격</th>
		<th>총액</th>
	{/snippet}

	{#snippet row(d)}
		<td>{d.name}</td>
		<td>{d.qty}</td>
		<td>{d.price}</td>
		<td>{d.qty * d.price}</td>
	{/snippet}
</Table>
```

**암묵적 `children` 스니펫**
컴포넌트 태그 사이에 스니펫 선언 말고 그냥 막 적은 내용들은 싸그리 모아서 `children`이라는 이름의 스니펫으로 자동 포장됩니다 (데모 참고): "컴포넌트 사이에 껴두면 `children`으로 변신!"

App.svelte
```svelte
<Button>날 눌러줘!</Button>
```

Button.svelte
```svelte
<script lang="ts">
	let { children } = $props();
</script>

<!-- 결과: <button>날 눌러줘!</button> -->
<button>{@render children()}</button>
```
주의: 컴포넌트 태그 사이에 내용을 넣을 거면, `children`이라는 이름의 prop을 따로 만들면 안 됩니다. 이름 겹치면 골치 아파져요. "애 이름 `children`으로 짓지 마라, 헷갈린다."

**선택적 스니펫 props (있으면 쓰고, 없으면 말고)**
스니펫 props를 '선택 사항'으로 만들 수 있습니다. 옵셔널 체이닝(`?.`) 써서 스니펫이 없으면 아무것도 안 그리거나...

```svelte
<script>
	let { children } = $props();
</script>

{@render children?.()}
```

...`#if` 블록으로 "없으면 이걸로 대신 보여줘" 하고 대체 컨텐츠를 그릴 수도 있습니다:

```svelte
<script>
	let { children } = $props();
</script>

{#if children}
	{@render children()}
{:else}
	없으면 이거라도... (대체 컨텐츠)
{/if}
```

**스니펫 타입 지정 (타입스크립트 쓰는 당신, 멋져)**
스니펫은 `svelte`에서 불러온 `Snippet` 인터페이스를 따릅니다: "타입 명찰 딱 붙여주면 버그가 줄어요."

```svelte
<script lang="ts">
	import type { Snippet } from 'svelte';

	interface Props {
		data: any[];
		children: Snippet; // 파라미터 없는 스니펫
		row: Snippet<[any]>; // 파라미터 하나 (any 타입) 받는 스니펫
	}

	let { data, children, row }: Props = $props();
</script>
```
이렇게 하면, `data` prop이나 `row` 스니펫 빼먹고 컴포넌트 쓰려고 하면 바로 빨간 줄 쫙 그어줍니다. `Snippet`에 넘기는 타입 인자는 튜플(배열 비슷한 거)인데, 스니펫이 파라미터를 여러 개 받을 수 있어서 그렇습니다. "타입 안 맞으면 바로 딱 걸림."

제네릭(generic) 써서 `data`랑 `row`가 같은 타입을 쓰도록 하면 코드가 더 탄탄해집니다:

```svelte
<script lang="ts" generics="T">
	import type { Snippet } from 'svelte';

	let {
		data,
		children,
		row
	}: {
		data: T[];
		children: Snippet;
		row: Snippet<[T]>; // T 타입의 데이터를 받는 스니펫
	} = $props();
</script>
```

**스니펫 바깥으로 내보내기 (Exporting snippets)**
`.svelte` 파일 맨 바깥에 선언한 스니펫은 `<script module>`에서 `export` 해서 다른 컴포넌트에서 `import` 해서 쓸 수 있습니다. 단, 일반 `<script>` 태그 안에 있는 변수 같은 걸 참조하지 않는 '순수한' 녀석이어야 합니다 (데모 참고): "잘 만든 부품, 다른 공장에도 납품 가능! (단, 자체 동력원 금지)"

```svelte
<script module>
	export { add };
</script>

{#snippet add(a, b)}
	{a} + {b} = {a + b}
{/snippet}
```
이 기능 쓰려면 Svelte 5.5.0 버전 이상이어야 합니다.

**프로그래밍 방식으로 스니펫 만들기 (고인물 전용)**
`createRawSnippet` API 쓰면 코드로 스니펫을 찍어낼 수 있습니다. 이건 진짜 고급 기술이니 웬만하면 구경만 하세요.

**스니펫 vs 슬롯 (세대교체)**
Svelte 4에서는 슬롯(slot)으로 컴포넌트에 컨텐츠를 전달했습니다. 스니펫은 슬롯보다 훨씬 강력하고 유연해서, Svelte 5에서는 슬롯이 deprecated(퇴물 취급) 되었습니다. "슬롯의 시대는 갔다. 이제 스니펫 천하!"

---

**얘 뭐 하는 애냐? (기능 및 목적)**
Svelte 5의 `{#snippet ...}`과 `{@render ...}`는 HTML 조각(마크업)을 함수처럼 만들어 재사용하는 기능입니다. 쉽게 말해, "마크업 복붙 방지턱"이자 "코드 정리 요정"이죠. 이걸 쓰면 코드 중복이 줄어들고, 가독성은 확 올라갑니다. 컴포넌트끼리 마크업 덩어리를 주고받는 것도 훨씬 유연해지고요. Svelte 4의 슬롯(slot) 기능을 더 강력하게 업그레이드한 버전이라고 보면 됩니다. "슬롯, 너는 이제 은퇴다!"

**왜 쓰는데?**
1.  **DRY (Don't Repeat Yourself) 원칙 실현**: 똑같은 HTML 구조를 여기저기 복사-붙여넣기 하는 노가다를 끝장냅니다. "Ctrl+C, Ctrl+V는 이제 그만!"
2.  **가독성 향상**: 복잡한 UI 로직이나 긴 HTML 덩어리를 깔끔한 스니펫으로 분리하면, 코드가 한눈에 쏙 들어옵니다. "코드가 술술 읽히네? 마법인가?"
3.  **유연한 컴포넌트 디자인**: 부모 컴포넌트에서 자식 컴포넌트의 특정 부분을 "이런 모양으로 그려줘!" 하고 스니펫을 쓱 전달할 수 있습니다. 자식 컴포넌트는 받은 스니펫을 렌더링만 하면 되니, 커스터마이징이 아주 쉬워지죠. "네모 틀 줄 테니 그림은 네가 그려!"
4.  **로직도 재활용**: 스니펫 안에는 단순 HTML뿐만 아니라 `#if`, `#each` 같은 Svelte의 로직 블록도 넣을 수 있어서, UI 조각과 그 조각을 보여주는 로직까지 한 큐에 재활용 가능합니다.

**언제 불려 나오냐? (언제 사용되냐?)**
1.  컴포넌트 안에서 똑같거나 비슷한 HTML 구조가 자꾸 반복될 때. "어? 이거 아까도 썼는데?" 싶으면 바로 스니펫 각입니다.
2.  부모가 자식 컴포넌트의 특정 구멍(슬롯 같은)에 원하는 HTML을 끼워 넣고 싶을 때.
3.  테이블의 헤더, 로우(행) 템플릿처럼, 데이터는 다른데 구조는 같은 부분을 만들 때.
4.  재귀적인 구조(트리 메뉴 등)를 표현할 때도 유용합니다. 스니펫이 자기 자신을 호출할 수 있거든요.

**쓸 때 꿀팁 및 주의사항:**
*   **스코프는 생명**: 스니펫은 선언된 곳과 그 주변(형제, 자식)에서만 쓸 수 있습니다. "우리 동네에서만 먹히는 쿠폰 같은 거." 밖으로 나가면 `{@render}` 해도 못 찾아요.
*   **파라미터는 요리 재료**: 스니펫에 파라미터를 넘겨서 동적으로 다른 내용을 보여줄 수 있습니다. 기본값 설정, 객체 구조 분해 할당 다 되는데, `...rest` 파라미터는 안됩니다. "레시피는 같아도 재료 따라 맛이 달라지는 법!"
*   **`children` 스니펫의 함정**: `<MyComponent>요기 내용물!</MyComponent>` 이렇게 컴포넌트 태그 사이에 뭘 넣으면, 그건 자동으로 `children`이라는 이름의 스니펫이 됩니다. 이걸 쓰려면 `<MyComponent>` 안에서 `{@render children()}` 하면 되는데, 만약 `MyComponent`가 원래 `children`이라는 이름의 prop을 받고 있었다면? 충돌나겠죠. "prop 이름 `children`은 웬만하면 쓰지 마세요. 헷갈리니까."
*   **타입스크립트랑 찰떡궁합**: `Snippet` 타입을 `svelte`에서 가져와서 쓰면 타입 체크도 빡세게 할 수 있습니다. 제네릭까지 쓰면 금상첨화. "타입 있는 스니펫, 버그 없는 미래!"
*   **`export`로 스니펫 수출**: `<script module>` 안에서 스니펫을 `export`하면 다른 컴포넌트에서 `import`해서 쓸 수 있습니다 (Svelte 5.5.0 이상). 단, 일반 `<script>` 태그 안에 있는 변수나 상태를 참조하는 스니펫은 수출 금지. "순수하게 마크업만 담은 놈만 국외 반출 가능!"
*   **재귀 호출 시 탈출 조건 필수**: 스니펫이 자기 자신을 부를 수 있는데, 이때 무한 루프 안 빠지게 `#if` 등으로 탈출 조건을 꼭 만들어야 합니다. "정신줄 놓으면 브라우저 터진다!"
*   **슬롯은 이제 안녕**: Svelte 4 쓰던 분들은 슬롯(slot)이 익숙하겠지만, Svelte 5에서는 스니펫이 그 자리를 대신합니다. 더 강력하고 유연하니 갈아타세요. "굿바이 슬롯, 헬로 스니펫!"