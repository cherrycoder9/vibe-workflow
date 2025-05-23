Svelte 5.29 버전부터는 `use:` 지시어 대신 **첨부(attachments)** 기능을 쓰는 걸 고려해보세요. 그게 더 유연하고 조합하기도 좋거든요. "신문물 나왔으니 갈아타는 게 인지상정!"

`use:` 지시어로 추가하는 **액션(Actions)**은 HTML 요소가 화면에 딱 붙었을 때 (마운트될 때) 호출되는 함수입니다. 보통 `$effect`를 같이 써서, 그 요소가 화면에서 떨어져 나갈 때 (언마운트될 때) 자기가 어질러 놓은 것들을 깔끔하게 치우도록 만듭니다.

**App.svelte**
```svelte
<script lang="ts">
	import type { Action } from 'svelte/action';

	const myaction: Action = (node) => {
		// 'node'가 DOM에 딱 붙었음!

		$effect(() => {
			// 여기서 필요한 초기 설정들을 조물조물 해줍니다.

			return () => {
				// 요소가 사라질 때 뒷정리하는 코드. "썼으면 치워야지!"
			};
		});
	};
</script>

<div use:myaction>...</div>
```

액션에 인자를 넘겨줄 수도 있습니다:

**App.svelte**
```svelte
<script lang="ts">
	import type { Action } from 'svelte/action';

	const myaction: Action = (node, data) => {
		// 'data'라는 인자를 받았으니 이걸로 뭔가 하겠죠?
	};
</script>

<div use:myaction={data}>...</div>
```

액션은 딱 한 번만 호출됩니다 (서버에서 화면 그릴 때는 실행 안 됨). 인자가 바뀌어도 다시 실행되지 않아요. "첫 만남이 중요해, 그 뒤론 신경 안 써."

`$effect`라는 마법 주문이 나오기 전에는, 액션이 `update`랑 `destroy`라는 메소드를 가진 객체를 반환할 수 있었습니다. `update`는 인자가 바뀌면 최신 값으로 다시 호출됐었죠. 근데 이젠 `$effect` 쓰는 게 더 좋습니다. "구관이 명관? 아니, 신기술이 짱짱맨!"

**타입 정의하기**

`Action` 인터페이스는 세 가지 선택적 타입 인자를 받을 수 있습니다:
1.  `node` 타입 (모든 요소에 적용된다면 `Element` 사용 가능)
2.  액션에 전달되는 `parameter` 타입
3.  액션이 만들어내는 커스텀 이벤트 핸들러 타입

**App.svelte**
```svelte
<script lang="ts">
	import type { Action } from 'svelte/action';

	// HTMLDivElement에만 적용되고, 인자는 없고, onswiperight/onswipeleft 커스텀 이벤트를 만드는 액션
	const gestures: Action<
		HTMLDivElement,       // 이 액션은 div에만 붙일 수 있어!
		undefined,            // 인자는 안 받는다!
		{                     // 이런 커스텀 이벤트를 만들어서 쏠 거야!
			onswiperight: (e: CustomEvent) => void;
			onswipeleft: (e: CustomEvent) => void;
			// ...
		}
	> = (node) => {
		$effect(() => {
			// ...
			node.dispatchEvent(new CustomEvent('swipeleft')); // "왼쪽으로 스와이프했다!" 이벤트 발사!

			// ...
			node.dispatchEvent(new CustomEvent('swiperight')); // "오른쪽으로 스와이프했다!" 이벤트 발사!
		});
	};

  function next() { /* 다음으로 넘어가는 로직 */ }
  function prev() { /* 이전으로 돌아가는 로직 */ }
</script>

<div
	use:gestures
	onswipeleft={next}
	onswiperight={prev}
>...</div>
```

---

**얘 뭐 하는 애냐? (기능 및 목적)**
`use:actionName`은 Svelte 컴포넌트의 HTML 요소에 자바스크립트 기능을 직접 붙여서 조작할 때 쓰는 문법입니다. "이봐 HTML, 너한테 특별한 능력을 부여해주지!" 같은 느낌이죠. DOM 요소가 생기고 없어지는 생명주기에 맞춰 특정 코드를 실행하거나, 요소에 직접 이벤트 리스너를 달거나, 외부 라이브러리랑 연동할 때 유용합니다. 한마디로 "HTML 요소에 직접 마법 부여하기"입니다.

**왜 쓰는데?**
1.  **DOM 직접 제어**: Svelte의 선언적인 방식만으로는 부족할 때, 특정 DOM 요소에 직접 접근해서 뭔가 하고 싶을 때 씁니다. 예를 들어, 외부 라이브러리 (차트 라이브러리, 지도 라이브러리 등)를 특정 `div`에 붙일 때. "Svelte로는 부족해, 직접 만져야겠어!"
2.  **재사용 가능한 로직 캡슐화**: 여러 요소에 반복적으로 적용해야 하는 DOM 관련 로직 (예: 드래그앤드롭, 클릭 외부 감지, 스크롤 애니메이션 등)을 액션 함수로 만들어두면 깔끔하게 재사용 가능합니다. "이 마법은 여기저기 써먹을 수 있지!"
3.  **요소 생명주기 관리**: 요소가 DOM에 추가될 때 초기화하고, 제거될 때 정리하는 작업을 `$effect`와 함께 깔끔하게 처리할 수 있습니다. 메모리 누수 방지에도 굿. "태어날 때 축복, 떠날 때 작별 인사."
4.  **커스텀 이벤트 만들기**: 위 예시처럼 `node.dispatchEvent`를 써서 요소가 자신만의 커스텀 이벤트를 발생시키게 할 수 있습니다. 이걸로 부모 컴포넌트와 소통 가능.

**언제 불려 나오냐?**
HTML 요소가 DOM에 마운트될 때 딱 한 번 호출됩니다. 서버 사이드 렌더링(SSR) 중에는 실행되지 않아요. `$effect` 내부의 코드는 의존성이 변경될 때마다 다시 실행될 수 있지만, 액션 함수 자체는 최초 마운트 시 한 번입니다.

**쓸 때 꿀팁 및 주의사항:**
*   **Svelte 5.29+ 이면 `attachments` 고려**: 더 새롭고 유연한 방식이 나왔으니, 가능하다면 `attachments`를 검토해보세요. "옛날 기술에 너무 매달리지 말자."
*   **`$effect`는 단짝친구**: 액션 내부에서 DOM 조작이나 이벤트 리스너 추가/제거 등의 작업을 할 때는 `$effect`를 활용해서 클린업 함수를 꼭 반환해주세요. 안 그러면 요소가 사라져도 이벤트 리스너 같은 게 좀비처럼 남아 메모리를 야금야금 먹습니다. "뒷정리는 필수!"
*   **인자 변경 시 재실행 안 됨**: `use:myaction={someData}`에서 `someData`가 바뀌어도 `myaction` 함수 자체가 다시 호출되지는 않습니다. 인자 변경에 반응해야 한다면 `$effect` 내부에서 그 인자를 의존성으로 사용해야 합니다. "첫인상만 보고 판단하니, 바뀌는 건 네가 알아서 캐치해."
*   **타입스크립트와 찰떡궁합**: `Action` 타입을 사용하면 액션 함수의 `node` 타입, `parameter` 타입, 그리고 발생시킬 커스텀 이벤트 타입을 명확하게 정의할 수 있어서 코드 안정성이 올라갑니다. "타입... 그거슨 빛..."
*   **서버 사이드 렌더링(SSR)에서는 동작 안 함**: 액션은 브라우저 환경에서 DOM이 실제로 존재할 때 의미가 있습니다. SSR 환경에서는 `node`가 없으니 당연히 실행될 수 없죠. "서버에서는 눈 감고 있는 녀석."
*   **너무 남용하면 스파게티 코드 주의**: 뭐든 과하면 독입니다. 간단한 DOM 조작은 Svelte의 기본 기능으로 충분할 수 있습니다. 액션은 정말 필요할 때, 재사용성을 높이거나 복잡한 DOM 상호작용을 캡슐화할 때 현명하게 사용하세요. "망치로 모든 걸 해결하려 들지 마."