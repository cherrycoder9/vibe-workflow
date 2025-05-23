`$props`
컴포넌트로 전달되는 입력값을 프롭(props)이라고 부릅니다. 'properties'의 줄임말이죠. HTML 엘리먼트에 속성을 전달하듯이 컴포넌트에 프롭을 전달합니다:

**App.svelte**
```svelte
<script lang="ts">
	import MyComponent from './MyComponent.svelte';
</script>

<MyComponent adjective="cool" />
```

반대편, 즉 `MyComponent.svelte` 안에서는 `$props` 룬(rune)으로 프롭을 받을 수 있습니다...

**MyComponent.svelte**
```svelte
<script lang="ts">
	let props = $props();
</script>

<p>이 컴포넌트는 {props.adjective}합니다</p>
```

...하지만 보통은 프롭을 구조 분해 할당(destructure)해서 씁니다:

**MyComponent.svelte**
```svelte
<script lang="ts">
	let { adjective } = $props();
</script>

<p>이 컴포넌트는 {adjective}합니다</p>
```

**대체값 (Fallback values)**
구조 분해 할당을 사용하면 대체값을 선언할 수 있습니다. 부모 컴포넌트가 특정 프롭을 설정하지 않았거나 값이 `undefined`일 때 이 대체값이 사용됩니다:

```svelte
let { adjective = 'happy' } = $props();
```
대체값은 반응형 상태 프록시(reactive state proxies)로 변환되지 않습니다 (자세한 내용은 '프롭 업데이트' 참조).

**프롭 이름 변경 (Renaming props)**
구조 분해 할당을 사용해서 프롭 이름을 바꿀 수도 있습니다. 프롭 이름이 유효한 식별자가 아니거나, `super`처럼 자바스크립트 예약어일 때 필요합니다:

```svelte
let { super: trouper = 'lights are gonna find me' } = $props();
```

**나머지 프롭 (Rest props)**
마지막으로, 나머지 속성(rest property)을 사용해서 말 그대로 나머지 프롭들을 가져올 수 있습니다:

```svelte
let { a, b, c, ...others } = $props();
```

**프롭 업데이트 (Updating props)**
컴포넌트 내부에서 프롭을 참조하는 값은 프롭 자체가 업데이트될 때 함께 업데이트됩니다 — `App.svelte`에서 `count`가 바뀌면, `Child.svelte` 안의 `count`도 바뀝니다. 하지만 자식 컴포넌트는 프롭 값을 임시로 덮어쓸 수 있는데, 이는 저장되지 않은 임시 상태(ephemeral state)에 유용할 수 있습니다 (데모 참고):

**App.svelte**
```svelte
<script lang="ts">
	import Child from './Child.svelte';

	let count = $state(0);
</script>

<button onclick={() => (count += 1)}>
	클릭 (부모): {count}
</button>

<Child {count} />
```

**Child.svelte**
```svelte
<script lang="ts">
	let { count } = $props();
</script>

<button onclick={() => (count += 1)}>
	클릭 (자식): {count}
</button>
```
프롭을 임시로 재할당할 수는 있지만, 바인딩 가능한(`bindable`) 프롭이 아니라면 프롭을 직접 변경(mutate)해서는 안 됩니다.

프롭이 일반 객체라면, 변경은 아무 효과가 없습니다 (데모 참고):

**App.svelte**
```svelte
<script lang="ts">
	import Child from './Child.svelte';
</script>

<Child object={{ count: 0 }} />
```

**Child.svelte**
```svelte
<script lang="ts">
	let { object } = $props();
</script>

<button onclick={() => {
	// 아무 효과 없음
	object.count += 1
}}>
	클릭: {object.count}
</button>
```
하지만 프롭이 반응형 상태 프록시(reactive state proxy)라면, 변경 사항이 적용되긴 하지만 `ownership_invalid_mutation` 경고가 뜰 겁니다. 왜냐하면 컴포넌트가 자기 '소유'가 아닌 상태를 변경하고 있기 때문입니다 (데모 참고):

**App.svelte**
```svelte
<script lang="ts">
	import Child from './Child.svelte';

	let object = $state({count: 0});
</script>

<Child {object} />
```

**Child.svelte**
```svelte
<script lang="ts">
	let { object } = $props();
</script>

<button onclick={() => {
	// 아래 count는 업데이트되지만,
	// 경고가 발생합니다. 당신 소유가
	// 아닌 객체는 변경하지 마세요!
	object.count += 1
}}>
	클릭: {object.count}
</button>
```
`$bindable`로 선언되지 않은 프롭의 대체값은 건드려지지 않습니다 — 반응형 상태 프록시로 변환되지 않으므로 — 변경해도 업데이트가 발생하지 않습니다 (데모 참고).

**Child.svelte**
```svelte
<script lang="ts">
	let { object = { count: 0 } } = $props();
</script>

<button onclick={() => {
	// 대체값이 사용된 경우 아무 효과 없음
	object.count += 1
}}>
	클릭: {object.count}
</button>
```
요약: 프롭을 직접 변경하지 마세요. 변경 사항을 전달하려면 콜백 프롭을 사용하거나, 부모와 자식이 같은 객체를 공유해야 한다면 `$bindable` 룬을 사용하세요.

**타입 안전성 (Type safety)**
다른 변수 선언과 마찬가지로 프롭에 타입을 지정하여 컴포넌트에 타입 안전성을 추가할 수 있습니다. 타입스크립트에서는 이렇게 보일 겁니다...

```svelte
<script lang="ts">
	let { adjective }: { adjective: string } = $props();
</script>
```

...JSDoc에서는 이렇게 할 수 있습니다:

```svelte
<script>
	/** @type {{ adjective: string }} */
	let { adjective } = $props();
</script>
```
물론 타입 선언을 어노테이션과 분리할 수도 있습니다:

```svelte
<script lang="ts">
	interface Props {
		adjective: string;
	}

	let { adjective }: Props = $props();
</script>
```
네이티브 DOM 엘리먼트에 대한 인터페이스는 `svelte/elements` 모듈에서 제공됩니다 (래퍼 컴포넌트 타이핑 참조).

타입을 추가하는 것을 권장합니다. 컴포넌트를 사용하는 사람들이 어떤 프롭을 제공해야 하는지 쉽게 알 수 있도록 보장하기 때문입니다.

**`$props.id()`**
이 룬은 버전 5.20.0에 추가되었으며, 현재 컴포넌트 인스턴스에 고유한 ID를 생성합니다. 서버에서 렌더링된 컴포넌트를 하이드레이션할 때, 서버와 클라이언트 간에 값이 일관되게 유지됩니다.

이는 `for`나 `aria-labelledby` 같은 속성을 통해 엘리먼트들을 연결할 때 유용합니다.

```svelte
<script>
	const uid = $props.id();
</script>

<form>
	<label for="{uid}-firstname">이름: </label>
	<input id="{uid}-firstname" type="text" />

	<label for="{uid}-lastname">성: </label>
	<input id="{uid}-lastname" type="text" />
</form>
```

---

**얘 뭐 하는 애냐? (기능 및 목적)**
`$props()`는 Svelte 5부터 도입된 룬(Rune) 시스템의 핵심 멤버로, 부모 컴포넌트가 자식 컴포넌트에게 "야, 이거 받아 써!" 하고 데이터를 넘겨줄 때 쓰는 통로입니다. 쉽게 말해 컴포넌트판 "아빠 찬스" 또는 "엄마가 싸준 도시락" 같은 거죠. 부모가 정성껏 준비한 데이터를 자식이 받아서 화면에 예쁘게 보여주거나 내부 로직에 활용합니다. Svelte 4 시절 `export let`으로 프롭 받던 방식을 대체하는 더 명시적이고 강력한 방법입니다.

**왜 쓰는데?**
1.  **컴포넌트 재활용 만렙**: 똑같은 컴포넌트라도 `$props`로 다른 데이터를 넣어주면 전혀 다른 모습과 기능을 하게 만들 수 있습니다. "같은 틀로 다양한 붕어빵 찍어내기!"
2.  **데이터 흐름은 위에서 아래로**: 부모가 자식에게 데이터를 전달하는 단방향 데이터 흐름을 명확하게 보여줍니다. 데이터 출처가 명확해지니 코드 추적도 쉬워지죠. "물은 위에서 아래로, 데이터도 부모에서 자식으로!"
3.  **선언적이고 깔끔한 코드**: `$props()` 하나로 "나 프롭 받는다!" 선언 끝. 코드가 간결해지고 의도가 명확해집니다.
4.  **타입스크립트랑 찰떡궁합**: 프롭에 타입을 지정해서 개발 단계에서부터 오류를 줄이고, 자동완성의 축복도 누릴 수 있습니다. "타입 정의는 미래의 나를 위한 보험."

**언제 불려 나오냐?**
자식 컴포넌트의 `<script>` 태그 안에서, 부모 컴포넌트로부터 전달받은 데이터를 사용하고 싶을 때 `let { ... } = $props();` 형태로 호출합니다.

**쓸 때 꿀팁 및 주의사항:**
*   **구조 분해 할당은 국룰**: `let { prop1, prop2 = '기본값' } = $props();` 처럼 쓰면 가독성도 좋고, 프롭이 안 넘어왔을 때 쓸 기본값 설정도 편합니다. "하나하나 꺼내 쓰기 귀찮잖아?"
*   **프롭은 읽기 전용, 수정은 금지 (웬만하면)**: 자식 컴포넌트 안에서 `$props`로 받은 값을 직접 바꾸는 건 안티패턴입니다. 부모가 준 용돈을 자식이 마음대로 불리거나 줄이면 안 되는 것처럼요. 데이터 변경은 부모에게 요청(콜백 함수)하거나, 양방향 바인딩이 필요하면 `$bindable` 룬을 쓰세요. "네 물건 아니면 함부로 손대지 마라!"
*   **타입 정의는 습관처럼**: `let { name }: { name: string } = $props();` 처럼 타입스크립트 인터페이스나 인라인 타입으로 프롭의 형태를 명확히 하세요. 나중에 "이거 뭐였지?" 하는 불상사를 막아줍니다.
*   **`$props.id()`는 웹 접근성 도우미**: `<label for="...">`와 `<input id="...">`를 연결할 때 고유 ID가 필요한데, 이때 `$props.id()`를 쓰면 서버 사이드 렌더링(SSR) 환경에서도 서버와 클라이언트 간 ID가 일치해서 하이드레이션 오류를 막아줍니다. "모두를 위한 웹, ID부터 챙기자."
*   **Svelte 4의 `export let`과 작별**: Svelte 5에서는 `$props()`가 그 자리를 대신합니다. 더 명시적이고 룬 시스템과 잘 어울리죠. "구관이 명관이라지만, 신문물이 더 좋을 때도 있다."