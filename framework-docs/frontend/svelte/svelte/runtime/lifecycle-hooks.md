Svelte 5 라이프사이클 훅 (Lifecycle hooks)

Svelte 5에서 컴포넌트 라이프사이클은 딱 두 단계로 끝납니다: **생성**과 **소멸**. 그 사이, 특정 상태가 업데이트되는 건 컴포넌트 전체와는 상관없어요. 오직 그 상태 변화에 반응해야 하는 부분만 알림을 받습니다. 왜냐하면 내부적으로 변경의 최소 단위는 컴포넌트가 아니라, 컴포넌트 초기화 시 설정되는 (렌더) 이펙트(effects)이기 때문이죠. 그래서 "업데이트 전/후" 훅 같은 건 없습니다.

**onMount**

`onMount` 함수는 컴포넌트가 DOM에 딱 붙자마자 실행될 콜백을 예약합니다. 반드시 컴포넌트 초기화 중에 호출되어야 해요 (하지만 컴포넌트 내부에 있을 필요는 없고, 외부 모듈에서 호출해도 됩니다).

`onMount`는 서버에서 렌더링되는 컴포넌트 내부에서는 실행되지 않습니다.

```svelte
<script>
	import { onMount } from 'svelte';

	onMount(() => {
		console.log('컴포넌트 마운트됨!'); // "나 이제 화면에 나왔다!"
	});
</script>
```

만약 `onMount`에서 함수를 반환하면, 그 함수는 컴포넌트가 제거될 때(unmounted) 호출됩니다.

```svelte
<script>
	import { onMount } from 'svelte';

	onMount(() => {
		const interval = setInterval(() => {
			console.log('삑!'); // 1초마다 "삑!"
		}, 1000);

		return () => clearInterval(interval); // "나갈 때 뒷정리는 하고 가야지!" 인터벌 청소.
	});
</script>
```

이 동작은 `onMount`에 전달된 함수가 **동기적으로** 값을 반환할 때만 작동합니다. `async` 함수는 항상 Promise를 반환하므로, 동기적으로 함수를 반환할 수 없어요. "비동기는 타이밍이 안 맞아서 뒷정리 못 맡긴다!"

**onDestroy**

컴포넌트가 제거되기 직전에 실행될 콜백을 예약합니다.

`onMount`, `beforeUpdate`, `afterUpdate`, `onDestroy` 중에서 유일하게 서버 사이드 컴포넌트 내부에서 실행되는 녀석입니다.

```svelte
<script>
	import { onDestroy } from 'svelte';

	onDestroy(() => {
		console.log('컴포넌트 파괴 직전!'); // "나 이제 간다, 잘 있어라!"
	});
</script>
```

**tick**

"업데이트 후" 훅은 없지만, `tick`을 사용하면 UI가 업데이트된 후에 다음 작업을 진행하도록 보장할 수 있습니다. `tick`은 보류 중인 모든 상태 변경이 적용되거나, 변경 사항이 없으면 다음 마이크로태스크에서 완료되는 Promise를 반환합니다.

```svelte
<script>
	import { tick } from 'svelte';

	$effect.pre(() => {
		console.log('컴포넌트 곧 업데이트 예정'); // "잠시만, 화면 바뀐다!"
		tick().then(() => {
				console.log('컴포넌트 방금 업데이트됨'); // "화면 다 바꿨다!"
		});
	});
</script>
```

**Deprecated: beforeUpdate / afterUpdate (이제 쓰지 마세요)**

Svelte 4에는 컴포넌트 전체가 업데이트되기 전과 후에 실행되는 훅이 있었습니다. 하위 호환성을 위해 Svelte 5에서도 임시로 지원하지만, 룬(runes)을 사용하는 컴포넌트에서는 사용할 수 없습니다.

```svelte
<script>
	import { beforeUpdate, afterUpdate } from 'svelte';

	beforeUpdate(() => {
		console.log('컴포넌트 곧 업데이트 예정 (구식)');
	});

	afterUpdate(() => {
		console.log('컴포넌트 방금 업데이트됨 (구식)');
	});
</script>
```

`beforeUpdate` 대신 `$effect.pre`를, `afterUpdate` 대신 `$effect`를 사용하세요. 이 룬들은 더 세밀한 제어가 가능하고, 실제로 관심 있는 변경 사항에만 반응합니다. "옛날 방식은 이제 버려, 더 똑똑한 놈들이 나왔다고!"

**채팅창 예제**

새 메시지가 나타날 때 자동으로 맨 아래로 스크롤되는 채팅창을 구현한다고 해봅시다 (단, 이미 맨 아래에 스크롤되어 있을 때만). 이러려면 DOM을 업데이트하기 전에 측정해야 합니다.

Svelte 4에서는 `beforeUpdate`로 이걸 했지만, 이건 결함이 있는 방식입니다. 관련 있든 없든 모든 업데이트 전에 실행되거든요. 아래 예제에서는 `updatingMessages` 같은 확인 로직을 넣어서, 누가 다크 모드를 켜고 끌 때 스크롤 위치가 꼬이지 않도록 해야 합니다.

룬을 사용하면 `$effect.pre`를 쓸 수 있는데, 이건 `$effect`와 비슷하게 동작하지만 DOM이 업데이트되기 전에 실행됩니다. 이펙트 본문 안에서 `messages`를 명시적으로 참조하기만 하면, `messages`가 변경될 때만 실행되고 `theme`이 변경될 때는 실행되지 않습니다.

따라서 `beforeUpdate`와 그만큼 문제 많은 짝꿍 `afterUpdate`는 Svelte 5에서 deprecated 되었습니다. "효율 떨어지는 놈들은 이제 퇴물 취급이다!"

---

**얘네 뭐 하는 애들이냐?**

Svelte 5의 라이프사이클 훅은 컴포넌트가 태어나고 죽는, 딱 그 두 순간에만 집중합니다. `onMount`는 컴포넌트가 화면에 딱! 나타났을 때 "짠! 나 등장!" 하고 알려주는 신호탄이고, `onDestroy`는 컴포넌트가 사라지기 직전에 "나 이제 퇴장한다..." 유언 남기는 격이죠. Svelte 4에 있던 `beforeUpdate`, `afterUpdate`는 "내가 뭘 하든 일단 불러!" 스타일이라 비효율적이었는데, Svelte 5에서는 `$effect.pre`나 `$effect` 같은 룬(runes)으로 "진짜 필요한 놈만 불러!" 식으로 바뀌었습니다. `tick`은 "화면 다 바뀔 때까지 잠깐 기다려!" 하는 용도로 쓰고요.

**왜 쓰는데?**

1.  **`onMount`**:
    *   DOM 요소 직접 조작: 컴포넌트가 화면에 그려진 후에야 가능한 DOM 관련 작업들 (예: 특정 요소에 포커스 주기, 외부 라이브러리 초기화) 할 때 씁니다. "일단 화면에 내보내고 나서 만지작거리자."
    *   데이터 가져오기: 컴포넌트 보일 때 API에서 데이터 땡겨올 때 씁니다.
    *   이벤트 리스너 설정/해제: `window`나 `document`에 이벤트 걸고, 컴포넌트 사라질 때 깔끔하게 치울 때 씁니다. `onMount`에서 함수를 리턴하면 그게 `onDestroy`처럼 작동해서 뒷정리하기 딱 좋죠.
2.  **`onDestroy`**:
    *   메모리 누수 방지: `setInterval`, `setTimeout` 같은 거 걸어놨던 거 해제하거나, 웹소켓 연결 끊는 등 "내가 만든 쓰레기는 내가 치운다!" 정신을 발휘할 때 씁니다.
3.  **`tick`**:
    *   DOM 업데이트 후 작업 보장: 상태 변경 후 DOM이 확실히 업데이트된 다음에 뭔가 하고 싶을 때 (예: 스크롤 위치 조정, 변경된 DOM 크기 측정) 씁니다. "급하게 서두르지 말고, 화면 다 바뀐 거 확인하고 다음 일 해!"
4.  **`$effect.pre` / `$effect` (Svelte 5 스타일 업데이트 감지)**:
    *   `beforeUpdate` / `afterUpdate` 대체: 특정 상태가 변했을 때만, DOM 업데이트 직전이나 직후에 뭔가를 하고 싶을 때 씁니다. 훨씬 정교하고 효율적이죠. "옛날처럼 아무 때나 부르지 말고, 진짜 용건 있을 때만 불러!"

**언제 불려 나오냐?**

*   `onMount`: 컴포넌트가 처음 DOM에 삽입된 직후, 딱 한 번. (서버 사이드 렌더링 시에는 안 불림)
*   `onDestroy`: 컴포넌트가 DOM에서 제거되기 직전, 딱 한 번. (서버 사이드 렌더링 시에도 불림)
*   `tick`: 호출하면, 현재 진행 중인 상태 변경으로 인한 DOM 업데이트가 완료된 후 (또는 다음 마이크로태스크에서) Promise가 resolve 되면서 `.then()` 안의 코드가 실행됩니다.
*   `$effect.pre` / `$effect`: 이펙트 함수 내에서 참조하는 반응형 상태(state)가 변경될 때마다. `$effect.pre`는 DOM 업데이트 전, `$effect`는 DOM 업데이트 후에 실행됩니다.

**쓸 때 꿀팁 및 주의사항:**

*   **`onMount`의 클린업 함수는 동기적으로!**: `onMount`에서 반환하는 정리 함수는 `async` 쓰면 안 됩니다. Promise 반환하면 Svelte가 "이거 언제 끝날지 몰라서 정리 못 해주겠다!" 하고 무시합니다.
*   **서버 사이드 렌더링(SSR) 환경 인지**: `onMount`는 브라우저 전용입니다. SSR 환경에서는 `window`나 `document` 같은 브라우저 객체가 없어서 에러 나기 십상이죠. `onDestroy`는 SSR에서도 돌아가니 참고하세요.
*   **`tick`은 만능 해결사 아님**: `tick`은 DOM 업데이트를 기다려주지만, 모든 비동기 작업을 기다려주진 않습니다. API 호출 같은 건 별도로 `async/await` 처리해야 합니다. "화면 업데이트만 기다려주는 거지, 네 모든 숙제를 대신 해주진 않아!"
*   **Svelte 5에서는 룬(Runes)이 대세**: `beforeUpdate`, `afterUpdate`는 이제 구시대 유물입니다. Svelte 5에서는 `$effect.pre`, `$effect` 같은 룬을 써서 더 깔끔하고 효율적으로 상태 변화에 대응하세요. "새 술은 새 부대에! 업그레이드된 기능 쓰자!"
*   **`onDestroy`에서 비동기 작업은 조심**: `onDestroy` 콜백은 컴포넌트 파괴 직전에 실행되지만, 콜백 안의 비동기 작업이 끝날 때까지 기다려주지 않습니다. 중요한 마무리 작업은 동기적으로 처리하거나, 정말 필요하면 애플리케이션 레벨에서 관리해야 합니다. "떠나는 마당에 너무 오래 붙잡지 마라!"