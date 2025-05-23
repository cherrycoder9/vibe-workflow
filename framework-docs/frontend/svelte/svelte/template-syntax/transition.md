Svelte 트랜지션은 상태 변경으로 인해 화면에서 요소가 나타나거나 사라질 때 부드러운 애니메이션 효과를 줍니다. "뿅!" 하고 나타나거나 "슝~" 하고 사라지는 마법 같은 거죠.

`{#if ...}` 같은 블록이 사라질 때, 그 안의 모든 요소는 트랜지션이 있든 없든 블록 내 모든 트랜지션이 끝날 때까지 화면에 남아있습니다. "아직 안 끝났어, 가지 마!"

`transition:` 지시어는 양방향 트랜지션을 의미합니다. 즉, 트랜지션 진행 중에 부드럽게 반대 방향으로 전환될 수 있습니다. "어? 다시 돌아와? 알았어, 스무스하게!"

```svelte
<script>
	import { fade } from 'svelte/transition'; // Svelte 기본 제공 fade 효과 가져오기

	let visible = $state(false); // $state로 반응형 변수 선언 (Svelte 5 스타일)
</script>

<button onclick={() => visible = !visible}>보였다 안 보였다</button>

{#if visible}
	<div transition:fade>스르륵 나타났다 사라짐</div>
{/if}
```

**로컬 vs 글로벌**
트랜지션은 기본적으로 **로컬**입니다. 로컬 트랜지션은 자신이 속한 블록이 생기거나 없어질 때만 작동합니다. 부모 블록이 생기거나 없어질 때는 신경 안 써요. "내 구역에서만 움직인다!"

```svelte
{#if x}
	{#if y}
		<p transition:fade>y가 바뀔 때만 스르륵</p>

		<p transition:fade|global>x 또는 y가 바뀔 때 스르륵 (글로벌!) </p>
	{/if}
{/if}
```
`|global`을 붙이면 부모 블록의 변경에도 반응합니다. "어디든 상관없다, 보여줘!"

**내장 트랜지션**
`svelte/transition` 모듈에서 다양한 기본 트랜지션을 가져와 쓸 수 있습니다. `fade`, `fly`, `slide`, `scale`, `draw` 등이 있죠. "Svelte가 이미 다 만들어 놨으니 골라 쓰세요."

**트랜지션 매개변수**
트랜지션에 매개변수를 전달해서 동작을 바꿀 수 있습니다.
`(중괄호 {{ }} 두 개는 특별한 문법이 아니라, 표현식 태그 안에 객체 리터럴을 쓴 겁니다.)`

```svelte
{#if visible}
	<div transition:fade={{ duration: 2000 }}>2초 동안 스르륵</div>
{/if}
```
`duration` (지속 시간), `delay` (지연 시간), `easing` (가속도 함수) 등을 설정할 수 있습니다.

**커스텀 트랜지션 함수**
직접 트랜지션 함수를 만들 수도 있습니다.
`transition = (node: HTMLElement, params: any, options: { direction: 'in' | 'out' | 'both' }) => { ... }`
이 함수는 객체를 반환해야 하며, 이 객체에는 다음과 같은 속성들이 들어갈 수 있습니다:
*   `delay?`: 지연 시간 (밀리초)
*   `duration?`: 지속 시간 (밀리초)
*   `easing?`: `(t: number) => number` 형태의 가속도 함수. `t`는 0에서 1 사이 값.
*   `css?`: `(t: number, u: number) => string` 형태의 함수. Svelte가 웹 애니메이션용 키프레임을 생성합니다.
    *   `t`: 이징 함수가 적용된 0과 1 사이의 값. `in` 트랜지션은 0에서 1로, `out` 트랜지션은 1에서 0으로 실행됩니다. 즉, 1이 요소의 원래 상태.
    *   `u`: `1 - t` 값.
*   `tick?`: `(t: number, u: number) => void` 형태의 함수. `css` 대신 자바스크립트로 직접 애니메이션을 제어할 때 씁니다.

**커스텀 CSS 트랜지션 예시 (`whoosh`)**
```svelte
<script lang="ts">
	import { elasticOut } from 'svelte/easing'; // Svelte 기본 제공 이징 함수
	export let visible: boolean;

	function whoosh(node: HTMLElement, params: { delay?: number, duration?: number, easing?: (t: number) => number }) {
		const existingTransform = getComputedStyle(node).transform.replace('none', ''); // 기존 transform 값 유지

		return {
			delay: params.delay || 0,
			duration: params.duration || 400,
			easing: params.easing || elasticOut, // 기본 이징은 elasticOut
			css: (t, u) => `transform: ${existingTransform} scale(${t})` // t 값에 따라 스케일 조절
		};
	}
</script>

{#if visible}
	<div in:whoosh>쉭! 하고 나타남</div>
{/if}
```
`in:whoosh`처럼 `in:`이나 `out:`을 붙여서 들어올 때나 나갈 때만 특정 트랜지션을 적용할 수도 있습니다.

**커스텀 Tick 트랜지션 예시 (`typewriter`)**
`tick` 함수는 트랜지션 도중에 `t`와 `u` 값을 받아서 호출됩니다.
`css` 대신 `tick`을 쓸 수도 있지만, 가능하면 `css`를 쓰는 게 좋습니다. 웹 애니메이션은 메인 스레드 밖에서 돌 수 있어서 느린 기기에서도 버벅임(jank)을 줄여줍니다. "CSS 애니메이션이 성능 면에선 갑이다!"

```svelte
<script lang="ts">
	export let visible = false;

	function typewriter(node: HTMLElement, { speed = 1 }: { speed?: number }) {
		// 이 트랜지션은 자식으로 텍스트 노드 하나만 있는 요소에만 작동
		const valid = node.childNodes.length === 1 && node.childNodes[0].nodeType === Node.TEXT_NODE;

		if (!valid) {
			throw new Error(`이 트랜지션은 단일 텍스트 노드 자식만 있는 요소에서 작동합니다.`);
		}

		const text = node.textContent;
		const duration = text.length / (speed * 0.01); // 속도에 따라 지속 시간 계산

		return {
			duration,
			tick: (t) => { // t 값에 따라 텍스트를 잘라서 보여줌
				const i = ~~(text.length * t);
				node.textContent = text.slice(0, i);
			}
		};
	}
</script>

{#if visible}
	<p in:typewriter={{ speed: 1 }}>타자기처럼 한 글자씩 따다닥</p>
{/if}
```

**함수 반환 트랜지션 (Crossfade 등)**
트랜지션 함수가 트랜지션 객체 대신 함수를 반환하면, 그 함수는 다음 마이크로태스크에서 호출됩니다. 이걸로 여러 트랜지션이 서로 협력해서 크로스페이드 같은 효과를 만들 수 있습니다. "팀플이 가능하다 이 말이야."

트랜지션 함수는 `node`, `params` 외에 세 번째 인자로 `options` 객체를 받습니다.
`options` 객체 내용:
*   `direction`: `in`, `out`, `both` 중 하나. 트랜지션의 방향을 알려줍니다.

**트랜지션 이벤트**
트랜지션이 적용된 요소는 표준 DOM 이벤트 외에 다음 이벤트들을 발생시킵니다:
*   `introstart`: `in` 트랜지션 시작
*   `introend`: `in` 트랜지션 끝
*   `outrostart`: `out` 트랜지션 시작
*   `outroend`: `out` 트랜지션 끝

```svelte
{#if visible}
	<p
		transition:fly={{ y: 200, duration: 2000 }}
		onintrostart={() => (status = '등장 시작!')}
		onoutrostart={() => (status = '퇴장 시작!')}
		onintroend={() => (status = '등장 완료!')}
		onoutroend={() => (status = '퇴장 완료!')}
	>
		위아래로 날아다님
	</p>
{/if}
```
이 이벤트들을 이용해 트랜지션 각 단계에 맞춰 추가적인 작업을 할 수 있습니다. "지금 뭐 하는 중인지 알려줄게."

---

**얘 뭐 하는 애냐? (기능 및 목적)**
Svelte 트랜지션은 웹 페이지에서 요소들이 나타나고 사라질 때 시각적인 즐거움을 더해주는 애니메이션 기능입니다. 딱딱하게 뿅 나타나고 뿅 사라지는 게 아니라, 스르륵, 휙, 통통 튀는 등 다양한 효과를 줘서 사용자 경험(UX)을 부드럽고 세련되게 만듭니다. 한마디로 "밋밋한 화면에 생기를 불어넣는 마법 가루!"

**왜 쓰는데?**
1.  **UX 향상**: 부드러운 전환 효과는 사용자에게 안정감을 주고, 앱이 더 잘 만들어졌다는 인상을 줍니다. "보기 좋은 떡이 먹기도 좋다."
2.  **시선 유도**: 중요한 정보가 나타나거나 사라질 때 사용자의 시선을 자연스럽게 끌 수 있습니다. "여기 좀 보세요!"
3.  **상태 변화 인지**: 요소의 변화를 명확하게 보여줘서 사용자가 현재 무슨 일이 일어나고 있는지 쉽게 파악하도록 돕습니다. "어? 뭐가 바뀌었네?"
4.  **개발 편의성**: 복잡한 CSS 애니메이션이나 자바스크립트 코드를 직접 짜지 않아도, Svelte가 제공하는 간편한 지시어와 함수로 쉽게 구현 가능. "애니메이션, 어렵지 않아요~"

**언제 불려 나오냐?**
`{#if ...}`, `{#each ...}`, `{#await ...}`, `<svelte:component ...>` 등 Svelte의 블록 구조나 컴포넌트가 DOM에 추가되거나 제거될 때, 해당 요소에 `transition:`, `in:`, `out:` 지시어가 붙어있으면 자동으로 호출됩니다.

**쓸 때 꿀팁 및 주의사항:**
*   **과유불급**: 너무 많은 트랜지션이나 너무 길고 현란한 효과는 오히려 사용자를 피곤하게 만듭니다. "애니메이션 파티는 적당히!"
*   **성능 고려**: `tick` 기반 커스텀 트랜지션보다는 `css` 기반 트랜지션을 우선 사용하세요. 특히 모바일이나 저사양 기기에서는 성능 차이가 클 수 있습니다. "버벅이면 아무리 예뻐도 소용없다."
*   **`local` vs `global`**: 기본은 `local`입니다. 중첩된 `{#if}` 블록에서 안쪽 요소가 부모의 변화에도 반응하게 하려면 `|global`을 명시해야 합니다. 헷갈리면 일단 `local`로 두고 필요할 때 `global`을 추가하는 게 좋습니다.
*   **양방향(`transition:`) vs 단방향(`in:`/`out:`)**: 들어올 때와 나갈 때 같은 효과를 줄 거면 `transition:fade`처럼 쓰고, 다른 효과를 주거나 한쪽만 필요하면 `in:fly` / `out:slide`처럼 나눠 쓰세요.
*   **이징(Easing) 함수 활용**: `svelte/easing`에서 제공하는 다양한 이징 함수(예: `elasticOut`, `bounceIn`)를 쓰거나 직접 만들어서 애니메이션에 개성을 더해보세요. "밋밋한 직선 운동은 재미없잖아?"
*   **접근성(Accessibility)**: 움직임에 민감한 사용자를 위해 `prefers-reduced-motion` 미디어 쿼리를 확인해서 애니메이션을 줄이거나 끄는 옵션을 제공하는 것이 좋습니다. "모두를 위한 디자인!"
*   **`$state` (Svelte 5+)**: 예제 코드의 `let visible = $state(false);`는 Svelte 5의 새로운 반응성 시스템인 "룬(Runes)"을 사용한 것입니다. 이전 버전에서는 `let visible = false;`로 선언하고 할당 시 반응성이 트리거됩니다. "버전 따라 문법도 진화한다!"