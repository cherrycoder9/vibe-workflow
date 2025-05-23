`$effect`
이펙트는 상태가 업데이트될 때 실행되는 함수로, 외부 라이브러리 호출, `<canvas>` 요소에 그림 그리기, 네트워크 요청 등에 사용될 수 있습니다. 서버 사이드 렌더링 중에는 실행되지 않고 브라우저에서만 돌아갑니다.

일반적으로 이펙트 안에서 상태를 업데이트하는 건 피해야 합니다. 코드가 복잡해지고 끝없는 업데이트 루프에 빠지기 쉽거든요. 만약 그렇게 하고 있다면, `$effect`를 쓰지 말아야 할 경우를 참고해서 다른 방법을 찾아보세요.

`$effect` 룬(rune)으로 이펙트를 만들 수 있습니다 (데모):

```svelte
<script>
	let size = $state(50);
	let color = $state('#ff3e00');

	let canvas;

	$effect(() => {
		const context = canvas.getContext('2d');
		context.clearRect(0, 0, canvas.width, canvas.height);

		// 'color'나 'size'가 바뀔 때마다 다시 실행됨
		context.fillStyle = color;
		context.fillRect(0, 0, size, size);
	});
</script>

<canvas bind:this={canvas} width="100" height="100"></canvas>
```

Svelte가 이펙트 함수를 실행할 때, 어떤 상태 (및 파생 상태)가 접근되는지 추적하고 (untrack 안에서 접근된 경우는 제외), 나중에 그 상태가 변경되면 함수를 다시 실행합니다.

`$effect`가 왜 다시 실행되는지 또는 실행되지 않는지 이해하기 어렵다면, 의존성 이해하기를 보세요. Svelte 4에서 사용하던 `$: 블록`과는 다르게 이펙트가 트리거됩니다.

**생명주기 이해하기**
이펙트는 컴포넌트가 DOM에 마운트된 후, 그리고 상태 변경 후 마이크로태스크(microtask)에서 실행됩니다. 재실행은 일괄 처리되며 (즉, `color`와 `size`를 동시에 변경해도 두 번 실행되지 않음), DOM 업데이트가 적용된 후에 발생합니다.

`$effect`는 컴포넌트 최상단뿐만 아니라 어디든 사용할 수 있지만, 부모 이펙트가 실행 중일 때 호출되어야 합니다.

Svelte는 내부적으로 템플릿의 로직과 표현식을 나타내기 위해 이펙트를 사용합니다. `<h1>hello {name}!</h1>`이 `name`이 변경될 때 업데이트되는 방식이죠.

이펙트는 이펙트가 다시 실행되기 직전에 실행될 정리(teardown) 함수를 반환할 수 있습니다 (데모).

```svelte
<script>
	let count = $state(0);
	let milliseconds = $state(1000);

	$effect(() => {
		// 'milliseconds'가 바뀔 때마다 다시 생성됨
		const interval = setInterval(() => {
			count += 1;
		}, milliseconds);

		return () => {
			// 정리 함수가 제공되면,
			// a) 이펙트가 다시 실행되기 직전에
			// b) 컴포넌트가 파괴될 때 실행됨
			clearInterval(interval);
		};
	});
</script>

<h1>{count}</h1>

<button onclick={() => (milliseconds *= 2)}>느리게</button>
<button onclick={() => (milliseconds /= 2)}>빠르게</button>
```

정리 함수는 이펙트가 파괴될 때도 실행되는데, 이는 부모가 파괴되거나 (예: 컴포넌트가 언마운트될 때) 부모 이펙트가 다시 실행될 때 발생합니다.

**의존성 이해하기**
`$effect`는 함수 본문 내에서 동기적으로 읽히는 모든 반응형 값(`$state`, `$derived`, `$props` - 함수 호출을 통해 간접적으로 읽는 것도 포함)을 자동으로 감지하고 의존성으로 등록합니다. 이러한 의존성이 변경되면 `$effect`는 재실행을 예약합니다.

만약 `$state`와 `$derived`가 `$effect` 내부에서 직접 사용되면 (예: 반응형 클래스 생성 중), 해당 값들은 의존성으로 취급되지 않습니다.

비동기적으로 읽히는 값들 — 예를 들어 `await` 이후나 `setTimeout` 내부에서 읽히는 값들 — 은 추적되지 않습니다. 아래 예시에서 캔버스는 `color`가 변경될 때 다시 그려지지만, `size`가 변경될 때는 그렇지 않습니다 (데모):

```svelte
$effect(() => {
	const context = canvas.getContext('2d');
	context.clearRect(0, 0, canvas.width, canvas.height);

	// 'color'가 바뀔 때마다 다시 실행됨...
	context.fillStyle = color;

	setTimeout(() => {
		// ...하지만 'size'가 바뀔 때는 아님
		context.fillRect(0, 0, size, size);
	}, 0);
});
```

이펙트는 읽는 객체가 변경될 때만 다시 실행되며, 그 객체 내부의 속성이 변경될 때는 실행되지 않습니다. (개발 중에 객체 내부의 변경 사항을 관찰하고 싶다면 `$inspect`를 사용할 수 있습니다.)

```svelte
<script>
	let state = $state({ value: 0 });
	let derived = $derived({ value: state.value * 2 });

	// 이건 한 번만 실행됨, 'state'는 재할당되지 않고 변이만 되기 때문
	$effect(() => {
		state;
	});

	// 이건 'state.value'가 바뀔 때마다 실행됨...
	$effect(() => {
		state.value;
	});

	// ...이것도 마찬가지, 'derived'는 매번 새로운 객체이기 때문
	$effect(() => {
		derived;
	});
</script>

<button onclick={() => (state.value += 1)}>
	{state.value}
</button>

<p>{state.value}의 두 배는 {derived.value}</p>
```

이펙트는 마지막 실행 시 읽었던 값에만 의존합니다. 이는 조건부 코드가 있는 이펙트에 흥미로운 영향을 미칩니다.

예를 들어, 아래 코드 조각에서 `condition`이 참이면 `if` 블록 안의 코드가 실행되고 `color`가 평가됩니다. 따라서 `condition`이나 `color` 중 하나라도 변경되면 이펙트가 다시 실행됩니다.

반대로 `condition`이 거짓이면 `color`는 평가되지 않으므로, 이펙트는 `condition`이 변경될 때만 다시 실행됩니다.

```svelte
import confetti from 'canvas-confetti';

let condition = $state(true);
let color = $state('#ff3e00');

$effect(() => {
	if (condition) {
		confetti({ colors: [color] });
	} else {
		confetti();
	}
});
```

`$effect.pre`
드물게 DOM이 업데이트되기 전에 코드를 실행해야 할 수도 있습니다. 이를 위해 `$effect.pre` 룬을 사용할 수 있습니다:

```svelte
<script>
	import { tick } from 'svelte';

	let div = $state();
	let messages = $state([]);

	// ...

	$effect.pre(() => {
		if (!div) return; // 아직 마운트되지 않음

		// 'messages' 배열 길이를 참조하여 변경될 때마다 이 코드가 다시 실행되도록 함
		messages.length;

		// 새 메시지가 추가될 때 자동 스크롤
		if (div.offsetHeight + div.scrollTop > div.scrollHeight - 20) {
			tick().then(() => {
				div.scrollTo(0, div.scrollHeight);
			});
		}
	});
</script>

<div bind:this={div}>
	{#each messages as message}
		<p>{message}</p>
	{/each}
</div>
```

타이밍을 제외하고 `$effect.pre`는 `$effect`와 똑같이 작동합니다.

`$effect.tracking`
`$effect.tracking` 룬은 코드가 이펙트나 템플릿 내부와 같은 추적 컨텍스트 내에서 실행 중인지 여부를 알려주는 고급 기능입니다 (데모):

```svelte
<script>
	console.log('컴포넌트 설정 중:', $effect.tracking()); // false

	$effect(() => {
		console.log('이펙트 내부:', $effect.tracking()); // true
	});
</script>

<p>템플릿 내부: {$effect.tracking()}</p> <!-- true -->
```

이는 `createSubscriber`와 같은 추상화를 구현하는 데 사용되며, 반응형 값을 업데이트하는 리스너를 생성하지만 해당 값이 추적되고 있을 때만 (예를 들어 이벤트 핸들러 내부에서 읽히는 것이 아니라) 생성합니다.

`$effect.root`
`$effect.root` 룬은 자동 정리되지 않는 비추적 범위를 생성하는 고급 기능입니다. 수동으로 제어하려는 중첩된 이펙트에 유용합니다. 이 룬은 또한 컴포넌트 초기화 단계 외부에서 이펙트를 생성할 수 있게 해줍니다.

```javascript
const destroy = $effect.root(() => {
	$effect(() => {
		// 설정
	});

	return () => {
		// 정리
	};
});

// 나중에...
destroy();
```

`$effect`를 쓰지 말아야 할 경우
일반적으로 `$effect`는 자주 사용하는 도구라기보다는 분석이나 직접적인 DOM 조작과 같은 일종의 "탈출구"로 간주하는 것이 가장 좋습니다. 특히 상태를 동기화하는 데 사용하는 것을 피하세요. 이렇게 하는 대신...

```svelte
<script>
	let count = $state(0);
	let doubled = $state();

	// 이렇게 하지 마세요!
	$effect(() => {
		doubled = count * 2;
	});
</script>
```

...이렇게 하세요:

```svelte
<script>
	let count = $state(0);
	let doubled = $derived(count * 2);
</script>
```

`count * 2`와 같은 간단한 표현식보다 복잡한 경우에는 `$derived.by`를 사용할 수도 있습니다.

파생된 값을 재할당할 수 있도록 (예: 낙관적 UI 구축) 이펙트를 사용하고 있다면, Svelte 5.25부터 파생값을 직접 덮어쓸 수 있다는 점에 유의하세요.

하나의 값을 다른 값에 연결하기 위해 이펙트를 복잡하게 사용하고 싶을 수도 있습니다. 다음 예는 서로 연결된 "사용한 돈"과 "남은 돈"에 대한 두 개의 입력을 보여줍니다. 하나를 업데이트하면 다른 하나도 그에 따라 업데이트되어야 합니다. 이를 위해 이펙트를 사용하지 마세요 (데모):

```svelte
<script>
	let total = 100;
	let spent = $state(0);
	let left = $state(total);

	$effect(() => {
		left = total - spent;
	});

	$effect(() => {
		spent = total - left;
	});
</script>

<label>
	<input type="range" bind:value={spent} max={total} />
	{spent}/{total} 사용
</label>

<label>
	<input type="range" bind:value={left} max={total} />
	{left}/{total} 남음
</label>
```

대신 `oninput` 콜백이나, 가능하다면 더 나은 방법인 함수 바인딩을 사용하세요 (데모):

```svelte
<script>
	let total = 100;
	let spent = $state(0);
	let left = $state(total);

	function updateSpent(value) {
		spent = value;
		left = total - spent;
	}

	function updateLeft(value) {
		left = value;
		spent = total - left;
	}
</script>

<label>
	<input type="range" bind:value={() => spent, updateSpent} max={total} />
	{spent}/{total} 사용
</label>

<label>
	<input type="range" bind:value={() => left, updateLeft} max={total} />
	{left}/{total} 남음
</label>
```

만약 이펙트 내에서 `$state`를 반드시 업데이트해야 하고 동일한 `$state`를 읽고 쓰기 때문에 무한 루프에 빠진다면, `untrack`을 사용하세요.

---

**얘 뭐 하는 애냐? (기능 및 목적)**
`$effect`는 Svelte에서 "상태가 바뀌면 나도 움직인다!"를 담당하는 녀석입니다. 특정 상태(`$state`나 `$derived`로 만들어진 값)가 변할 때마다 특정 코드를 실행하고 싶을 때 씁니다. 주로 Svelte의 반응성 시스템 바깥세상과 소통할 때, 예를 들어 바닐라 JS 라이브러리를 연동하거나, `<canvas>`에 직접 그림을 그리거나, 서버에 데이터 요청 보낼 때 활약합니다. 중요한 건, 이놈은 서버에서 렌더링할 땐 잠자코 있다가 브라우저에 딱 그려지고 나서야 활동 개시합니다.

**왜 쓰는데?**
1.  **외부 세계와의 연결고리**: Svelte 컴포넌트 내부의 상태 변화를 외부 라이브러리나 브라우저 API(DOM 조작, `localStorage` 등)에 알리고 뭔가 작업을 시켜야 할 때. "야, `size` 바뀌었으니까 캔버스 다시 그려!"
2.  **수동 DOM 조작**: Svelte가 알아서 해주는 DOM 업데이트 말고, 직접 특정 DOM 요소를 붙잡고 뭔가 해야 할 때. (예: 특정 요소에 포커스 주기, 스크롤 위치 조작)
3.  **클린업(뒷정리) 작업**: 이펙트가 더 이상 필요 없거나, 컴포넌트가 사라질 때 `setInterval` 같은 거 멈추거나, 이벤트 리스너 제거하는 등의 뒷정리 작업이 필요할 때. 이펙트 함수에서 `return`으로 정리 함수를 넘겨주면 알아서 처리해줍니다. "시작했으면 끝도 깔끔하게!"

**언제 불려 나오냐?**
컴포넌트가 화면에 그려진 후(`mounted`), 그리고 `$effect`가 지켜보는 상태값이 바뀔 때마다 호출됩니다. 여러 상태값이 한꺼번에 바뀌어도 똑똑하게 한 번만 실행됩니다 (배치 처리).

**쓸 때 꿀팁 및 주의사항:**
*   **이펙트 안에서 상태 업데이트는 웬만하면 금지!**: `$effect` 안에서 또 다른 `$state`를 바꾸면 코드가 꼬이고, 심하면 "무한 업데이트 지옥"에 빠질 수 있습니다. "내가 나를 또 부르고, 또 부르고..." 상태를 파생시켜야 한다면 `$derived`를 쓰세요. 그게 국룰입니다.
*   **의존성 자동 감지, 하지만 함정도 존재**: `$effect` 함수 안에서 직접 읽는 `$state`, `$derived` 값들은 알아서 감지해서 걔네가 바뀔 때마다 이펙트가 다시 실행됩니다. 하지만 `setTimeout`이나 `await` 뒤처럼 비동기적으로 읽는 값은 추적 못 합니다. "눈앞에 있는 것만 본다!" 객체 자체를 읽는 것과 객체 내부 속성을 읽는 것도 다르게 반응하니 주의.
*   **정리 함수는 선택 아닌 필수 (필요하다면)**: `setInterval`, `addEventListener`처럼 시작했으면 끝내줘야 하는 작업들은 반드시 정리 함수를 반환해서 뒷감당을 시켜야 합니다. 안 그러면 메모리 누수 나거나 버그 터집니다. "썼으면 치워라!"
*   **`$effect.pre`는 DOM 업데이트 전 선수 치기**: 아주 가끔, DOM이 업데이트되기 *전에* 뭔가 해야 할 때 씁니다. 예를 들어 스크롤 위치를 미리 저장해뒀다가 DOM 업데이트 후 복원하는 식. 일반 `$effect`는 DOM 업데이트 *후에* 실행됩니다.
*   **`$effect.tracking()`은 지금 추적 중?**: 현재 코드가 Svelte의 반응성 추적 시스템 안에서 도는지 알려주는 고급 기능. 라이브러리 만들 때나 쓸 법한 기능이니 일반 앱 개발자는 크게 신경 안 써도 됩니다.
*   **`$effect.root()`는 독립적인 이펙트 왕국 건설**: 부모에게서 독립해서 자신만의 생명주기를 갖고, 자동 정리도 안 되는 특별한 이펙트 스코프를 만듭니다. 이것도 고급 기능. "나는 내 갈 길을 간다!"
*   **"이거 꼭 `$effect` 써야 하나?" 다시 한번 생각**: 상태를 단순히 변환하거나 조합하는 거면 `$derived`나 `$derived.by`가 훨씬 깔끔하고 Svelte스러운 방법입니다. `$effect`는 정말 "어쩔 수 없을 때" 쓰는 최후의 수단 느낌으로 접근하는 게 좋습니다. "망치로 모든 걸 해결하려 들지 마라."