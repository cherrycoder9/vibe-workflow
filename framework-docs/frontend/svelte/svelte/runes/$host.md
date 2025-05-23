`$host`
Svelte 컴포넌트를 커스텀 엘리먼트(custom element)로 컴파일할 때, `$host` 룬(rune)은 그 커스텀 엘리먼트 자체(호스트 엘리먼트)에 접근할 수 있게 해줍니다. 이걸로 뭘 할 수 있냐고요? 예를 들어 커스텀 이벤트를 발생시킬 수 있죠 (데모 참고):

**Stepper (카운터 부품)**

```svelte
<svelte:options customElement="my-stepper" />

<script lang="ts">
	function dispatch(type: string) {
		// $host()가 바로 이 <my-stepper> 태그 자체를 가리킴
		$host().dispatchEvent(new CustomEvent(type));
	}
</script>

<button on:click={() => dispatch('decrement')}>빼기</button>
<button on:click={() => dispatch('increment')}>더하기</button>
```

**App (부품 사용하는 곳)**

```svelte
<script lang="ts">
	import './Stepper.svelte'; // 위에서 만든 Stepper.svelte 파일을 가져옴

	let count = $state(0); // Svelte 5의 반응형 상태 변수
</script>

<!-- Stepper.svelte에서 정의한 "my-stepper" 태그를 HTML처럼 사용 -->
<my-stepper
	on:decrement={() => count -= 1}  // 'decrement' 커스텀 이벤트 발생 시 count 감소
	on:increment={() => count += 1}  // 'increment' 커스텀 이벤트 발생 시 count 증가
></my-stepper>

<p>횟수: {count}</p>
```

---

**얘 뭐 하는 애냐? (기능 및 목적)**
`$host`는 Svelte 컴포넌트가 웹 컴포넌트(Custom Element)라는 특수한 가면을 쓰고 바깥세상(DOM)과 소통할 때, 그 가면 자체, 즉 커스텀 엘리먼트 DOM 노드를 가리키는 마법의 지팡이입니다. Svelte 컴포넌트 안에서 "야, 나를 감싸고 있는 HTML 태그(<my-stepper> 같은 거) 좀 불러와 봐!" 할 때 쓰는 거죠. 주로 커스텀 엘리먼트가 자기 자신으로부터 이벤트를 뿅 하고 쏘거나, 자기 자신의 속성(attribute)을 직접 건드려야 할 때 등장합니다.

**왜 쓰는데?**
1.  **커스텀 이벤트 방출**: Svelte 컴포넌트 내부의 일이 바깥세상(이 컴포넌트를 사용하는 다른 HTML이나 자바스크립트)에 알려야 할 때 씁니다. 위의 예제처럼 버튼 눌린 걸 `decrement`나 `increment`라는 이름표를 붙여서 "나 눌렸어요!" 하고 외치는 거죠. "내부 사정, 외부 보고!"
2.  **호스트 엘리먼트 직접 조작**: 가끔은 컴포넌트가 자기 자신의 HTML 태그에 직접 스타일을 먹이거나, 특별한 속성을 추가/제거해야 할 때가 있습니다. 이럴 때 `$host()`로 엘리먼트를 잡아서 `setAttribute`, `style.setProperty` 같은 DOM API를 쓸 수 있습니다. "내 몸은 내가 꾸민다!" (하지만 남용은 금물!)

**언제 불려 나오냐?**
`<svelte:options customElement="태그-이름" />`으로 컴포넌트를 커스텀 엘리먼트로 만들었을 때, 그 컴포넌트의 `<script>` 태그 안에서 필요할 때 호출합니다. 특히 `dispatchEvent`로 커스텀 이벤트를 만들어서 밖으로 신호 보낼 때 단골로 쓰입니다.

**쓸 때 꿀팁 및 주의사항:**
*   **커스텀 엘리먼트 전용**: `$host`는 오직 `<svelte:options customElement="..." />`가 선언된 컴포넌트 안에서만 의미가 있습니다. 일반 Svelte 컴포넌트에서는 "그런 거 없는데?" 하고 에러 뿜습니다.
*   **이벤트 이름은 소문자로**: 커스텀 이벤트 이름 지을 때 `on:camelCaseEvent`처럼 쓰면 HTML에서는 `oncamelcaseevent`로 인식해서 못 알아먹습니다. `on:kebab-case-event`나 `on:lowercaseevent`처럼 소문자로 작명하는 게 국룰입니다. 예제처럼 `ondecrement`는 사실 `on:decrement`의 축약형이죠.
*   **`$host()`는 함수 호출**: `$host` 자체가 엘리먼트가 아니라, `$host()`라고 호출해야 엘리먼트를 반환합니다. 괄호 빼먹으면 "이게 왜 안돼?" 시전합니다.
*   **Svelte의 반응성과는 별개**: `$host()`로 가져온 엘리먼트에 직접 변경을 가하는 건 Svelte의 반응성 시스템 바깥에서 일어나는 일입니다. 꼭 필요할 때만 최소한으로 사용하고, 웬만한 건 Svelte의 데이터 바인딩이나 props를 활용하는 게 좋습니다. "최후의 수단으로 아껴 쓰자."
*   **타입스크립트 쓸 때**: `$host()`는 기본적으로 `HTMLElement` 타입을 반환합니다. 만약 특정 커스텀 엘리먼트 클래스에만 있는 속성이나 메소드를 쓰고 싶다면, 적절한 타입 단언(type assertion)이 필요할 수 있습니다. (예: `$host() as MySpecificElement`) "타입까지 챙겨야 프로."