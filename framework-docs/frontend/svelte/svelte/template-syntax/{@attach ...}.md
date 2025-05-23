`{@attach ...}` (어태치먼트)
어태치먼트는 DOM에 요소가 딱 붙거나(마운트) 함수 안에서 읽어 들인 상태값이 바뀔 때, 이펙트(effect) 안에서 실행되는 함수입니다. "요소야, 이 기능 장착해!" 하는 느낌이죠.

선택적으로, 어태치먼트가 다시 실행되기 전이나 요소가 DOM에서 나중에 떨어져 나갈 때 호출되는 함수를 반환할 수도 있습니다. "뒷정리는 하고 가야지?" 하는 거죠.

어태치먼트는 Svelte 5.29 버전부터 쓸 수 있습니다. 구버전에서는 꿈도 꾸지 마세요.

**App.svelte 예시**

```svelte
<script lang="ts">
	import type { Attachment } from 'svelte/attachments';

	const myAttachment: Attachment = (element) => {
		console.log(element.nodeName); // 'DIV' 같은 거 찍힘

		return () => {
			console.log('뒷정리 중...'); // 요소 없어질 때 실행
		};
	};
</script>

<div {@attach myAttachment}>...</div>
```

한 요소에 어태치먼트 여러 개 붙이는 것도 가능합니다. "덕지덕지 붙여도 괜찮아!"

**어태치먼트 팩토리 (Attachment factories)**
이거 아주 유용한 패턴인데, 함수가 어태치먼트를 찍어내서 반환하는 방식입니다. 아래 `tooltip` 예제처럼요 (데모 참고):

**App.svelte 예시 (툴팁 팩토리)**

```svelte
<script lang="ts">
	import tippy from 'tippy.js'; // 툴팁 라이브러리
	import type { Attachment } from 'svelte/attachments';

	let content = $state('헬로!'); // 반응형 상태

	// content 문자열을 받아서 Attachment 함수를 반환하는 팩토리
	function tooltip(content: string): Attachment {
		return (element) => {
			// tippy.js로 툴팁 생성
			const tooltipInstance = tippy(element, { content });
			// 뒷정리 함수로 툴팁 제거 함수를 반환
			return tooltipInstance.destroy;
		};
	}
</script>

<input bind:value={content} />

<button {@attach tooltip(content)}> <!-- content가 바뀔 때마다 tooltip 어태치먼트가 재생성됨 -->
	여기에 마우스 올려봐
</button>
```

`tooltip(content)` 표현식이 이펙트 안에서 실행되기 때문에, `content`가 바뀔 때마다 어태치먼트는 파괴되고 다시 만들어집니다. 어태치먼트 함수가 처음 실행될 때 읽는 다른 상태값이 바뀌어도 마찬가지고요. (이런 거 원치 않으면 "어태치먼트 재실행 제어하기" 부분을 참고하세요.)

**인라인 어태치먼트 (Inline attachments)**
어태치먼트를 그 자리에서 바로 만들 수도 있습니다 (데모 참고):

**App.svelte 예시 (캔버스 인라인 어태치먼트)**

```svelte
<canvas
	width={32}
	height={32}
	{@attach (canvas) => { // canvas 요소를 직접 받음
		const context = canvas.getContext('2d');

		$effect(() => { // color가 바뀔 때마다 이 내부 이펙트만 실행
			context.fillStyle = color;
			context.fillRect(0, 0, canvas.width, canvas.height);
		});
	}}
></canvas>
```

중첩된 `$effect`는 `color`가 바뀔 때마다 실행되지만, 바깥쪽 이펙트(`canvas.getContext(...)` 호출 부분)는 반응형 상태를 읽지 않으므로 딱 한 번만 실행됩니다.

**컴포넌트에 어태치먼트 전달하기 (Passing attachments to components)**
컴포넌트에 `{@attach ...}`를 쓰면, 키가 `Symbol`인 prop이 만들어집니다. 만약 컴포넌트가 그 `props`를 어떤 요소에다 펼쳐놓으면(`{...props}`), 그 요소가 어태치먼트를 받게 됩니다.

이걸로 요소를 강화하는 래퍼 컴포넌트를 만들 수 있습니다 (데모 참고):

**Button.svelte (래퍼 컴포넌트)**

```svelte
<script lang="ts">
	import type { HTMLButtonAttributes } from 'svelte/elements';

	// $props()로 모든 props를 받음 (어태치먼트 포함)
	let { children, ...props }: HTMLButtonAttributes = $props();
</script>

<!-- `props`에 어태치먼트가 포함되어 버튼 요소에 적용됨 -->
<button {...props}>
	{@render children?.()}
</button>
```

**App.svelte (래퍼 컴포넌트 사용)**

```svelte
<script lang="ts">
	import tippy from 'tippy.js';
	import Button from './Button.svelte'; // 위에서 만든 버튼 컴포넌트
	import type { Attachment } from 'svelte/attachments';

	let content = $state('헬로 어게인!');

	function tooltip(content: string): Attachment {
		return (element) => {
			const tooltipInstance = tippy(element, { content });
			return tooltipInstance.destroy;
		};
	}
</script>

<input bind:value={content} />

<Button {@attach tooltip(content)}> <!-- Button 컴포넌트에 어태치먼트 전달 -->
	여기도 마우스 올려봐
</Button>
```

**어태치먼트 재실행 제어하기 (Controlling when attachments re-run)**
어태치먼트는 액션(actions)과 다르게 완전히 반응형입니다: `{@attach foo(bar)}`는 `foo`나 `bar` (또는 `foo` 안에서 읽는 어떤 상태든)가 바뀌면 다시 실행됩니다.

```javascript
function foo(bar) {
	return (node) => {
		엄청_비싼_초기설정_작업(node);
		업데이트_함수(node, bar); // bar가 바뀔 때마다 foo 전체가 재실행됨
	};
}
```

이게 문제가 되는 드문 경우 (예를 들어, `foo`가 피할 수 없는 비싼 초기 설정 작업을 한다면), 데이터를 함수 안에 넣어서 전달하고 자식 이펙트에서 읽는 걸 고려해보세요:

```javascript
function foo(getBar) { // bar 대신 getBar 함수를 받음
	return (node) => {
		엄청_비싼_초기설정_작업(node); // 이건 한 번만 실행되길 원함

		$effect(() => { // getBar()의 결과 (즉, bar)가 바뀔 때 이 내부 이펙트만 실행
			업데이트_함수(node, getBar());
		});
	}
}
```

**프로그래밍 방식으로 어태치먼트 만들기 (Creating attachments programmatically)**
컴포넌트나 요소에 펼쳐놓을 객체에 어태치먼트를 추가하려면, `createAttachmentKey`를 쓰세요. (이건 좀 고급 기술)

---

**얘 뭐 하는 애냐? (기능 및 목적)**
`{@attach ...}`는 DOM 요소에다가 온갖 특수 효과나 부가 기능을 "딱!" 붙여주는 접착제 같은 놈입니다. Svelte 4의 `use:action`이랑 비슷해 보이지만, 훨씬 더 반응성이 좋아서 상태 변경에 따라 자동으로 갈아엎고 새로 붙는다는 게 핵심이죠. "DOM 요소야, 이제부터 넌 이 기능을 탑재한다!" 하고 명령하는 것과 비슷합니다. 요소가 화면에 나타날 때 특정 로직을 실행하고, 사라질 때 뒷정리까지 깔끔하게 처리할 수 있게 해줍니다.

**왜 쓰는데? (장점)**
1.  **완전 자동 반응성**: `use:action`은 업데이트 함수를 따로 둬야 했지만, 얘는 관련된 상태가 바뀌면 알아서 파괴되고 재생성됩니다. "신경 꺼도 알아서 착착!"
2.  **깔끔한 로직 분리**: 복잡한 DOM 조작이나 외부 라이브러리 연동 로직을 컴포넌트 파일 밖으로 빼내거나, 재사용 가능한 "어태치먼트 팩토리" 함수로 만들 수 있습니다. "코드가 깔끔해지는 마법!"
3.  **유연한 조합과 전달**: 한 요소에 여러 개 붙이거나, 심지어 자식 컴포넌트로 어태치먼트를 전달해서 내부 요소에 적용시키는 것도 가능합니다. "레고 블록처럼 조립하는 재미!"
4.  **명확한 뒷정리(cleanup)**: 어태치먼트 함수가 반환하는 함수는 요소가 DOM에서 제거될 때 실행됩니다. 이벤트 리스너 해제나 타이머 제거 같은 메모리 누수 방지 작업을 까먹지 않고 챙길 수 있죠. "썼으면 치워야지, 암!"

**언제 불려 나오냐? (호출 시점)**
*   HTML 요소가 DOM에 처음 그려질 때 (마운트될 때). "안녕, 세상아!"
*   어태치먼트 함수 자체나, 어태치먼트 팩토리 함수, 또는 어태치먼트 함수 내부에서 참조하는 반응형 상태(`$state`, `$derived` 등으로 만들어진 변수) 값이 변경될 때. 이때는 기존 어태치먼트가 파괴되고(반환된 정리 함수 실행) 새로운 값으로 다시 실행됩니다. "어, 상태 바뀌었네? 갈아엎자!"

**쓸 때 꿀팁 및 주의사항:**
*   **Svelte 5.29 이상 필수**: 버전 확인 안 하고 쓰면 "그런 거 없는데?" 에러 맞습니다. "최신 기술은 버전부터 확인!"
*   **반응성 제대로 이해하기**: 어떤 상태가 변경될 때 어태치먼트가 재실행되는지 정확히 알아야 합니다. 의도치 않은 재실행은 성능 문제를 일으킬 수 있어요. "얘가 왜 자꾸 움직이지? 아, 저 변수 때문이구나!"
*   **비싼 초기 작업은 신중하게**: 어태치먼트가 재실행될 때마다 비용이 큰 초기 설정 작업(예: 복잡한 라이브러리 초기화)이 반복된다면 성능이 나빠질 수 있습니다. 이럴 땐 "어태치먼트 재실행 제어하기"에서 본 것처럼, 데이터를 함수로 감싸고 내부 `$effect`에서 처리하는 패턴을 써서 초기 작업은 한 번만 하도록 조절하세요. "무지성 재실행은 성능 저하의 특급 열차!"
*   **액션(Action)과 뭐가 다른데?**: 액션은 요소가 생성될 때 한 번 실행되고, 선택적으로 `update` 함수를 통해 파라미터 변경에 반응합니다. 반면 어태치먼트는 관련된 상태가 바뀌면 통째로 파괴되고 재생성되는, 더 강력한 반응성을 가집니다. "액션은 리모델링, 어태치먼트는 철거 후 재건축." 상황 따라 골라 쓰세요.
*   **뒷정리 함수는 국룰**: 외부 라이브러리 인스턴스 생성, `window`나 `document`에 이벤트 리스너 직접 등록 등의 작업을 했다면, 반드시 반환 함수에서 해당 인스턴스를 `destroy()` 하거나 `removeEventListener()`로 제거해야 합니다. 안 그러면 메모리 누수 파티! "청소는 셀프, 그리고 필수!"
*   **`createAttachmentKey`는 진짜 필요할 때만**: 대부분의 경우 HTML 템플릿에서 `{@attach ...}` 구문 쓰는 걸로 충분합니다. 프로그래밍 방식으로 동적으로 어태치먼트를 만들어서 객체에 심어야 하는 아주 특수한 상황 아니면 쓸 일 잘 없습니다. "굳이 어려운 길을 갈 필요는..."