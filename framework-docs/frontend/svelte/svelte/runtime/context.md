Context API

Context는 부모 컴포넌트가 가진 값을 자식 컴포넌트에게 props로 줄줄이 넘겨주지 않고도 (이른바 'props 드릴링' 없이) 접근할 수 있게 해줍니다. 부모 컴포넌트는 `setContext(key, value)`로 컨텍스트를 설정하고...

**Parent.svelte**
```svelte
<script lang="ts">
	import { setContext } from 'svelte';

	setContext('my-context', 'Parent.svelte에서 보낸 안녕!');
</script>
```

...자식 컴포넌트는 `getContext`로 그 값을 가져옵니다:

**Child.svelte**
```svelte
<script lang="ts">
	import { getContext } from 'svelte';

	const message = getContext('my-context');
</script>

<h1>{message}, Child.svelte 안에서 받음</h1>
```

이건 특히 `Parent.svelte`가 `Child.svelte`를 직접 알지 못하고, 대신 `children` 스니펫의 일부로 렌더링할 때 아주 유용합니다 (데모 참고):

```svelte
<Parent>
	<Child />
</Parent>
```

위 예제의 `key`('my-context')와 컨텍스트 값 자체는 어떤 자바스크립트 값이든 될 수 있습니다.

`setContext`와 `getContext` 외에도, Svelte는 `hasContext`와 `getAllContexts` 함수도 제공합니다.

**상태(state)와 함께 컨텍스트 사용하기**

반응형 상태를 컨텍스트에 저장할 수 있습니다 (데모 참고)...

```svelte
<script>
	import { setContext } from 'svelte';
	import Child from './Child.svelte';

	let counter = $state({ // Svelte 5의 $state 사용 예시
		count: 0
	});

	setContext('counter', counter);
</script>

<button onclick={() => counter.count += 1}>
	증가
</button>

<Child />
<Child />
<Child />
```

...하지만 `counter`를 업데이트하는 대신 재할당하면 '연결이 끊어진다'는 점에 유의하세요. 즉, 이렇게 하지 말고...

```svelte
<button onclick={() => counter = { count: 0 }}> <!-- 이러면 안됨! -->
	초기화
</button>
```

...이렇게 해야 합니다:

```svelte
<button onclick={() => counter.count = 0}> <!-- 이렇게 해야 됨! -->
	초기화
</button>
```

잘못 사용하면 Svelte가 경고를 날려줄 겁니다.

**타입-안전한 컨텍스트**

타입 안전성을 유지하면서 `setContext`와 `getContext` 호출을 랩핑하는 헬퍼 함수를 만드는 건 유용한 패턴입니다:

**context.ts**
```typescript
import { getContext, setContext } from 'svelte';
import type { User } from './types'; // 예시 타입

const key = Symbol(); // 고유한 심볼 키 사용 권장

export function setUserContext(user: User) {
	setContext(key, user);
}

export function getUserContext() {
	return getContext(key) as User; // 타입 단언
}
```

**전역 상태 대체하기**

여러 컴포넌트에서 공유되는 상태가 있을 때, 별도 모듈에 넣고 필요한 곳마다 가져다 쓰고 싶을 수 있습니다:

**state.svelte.js (또는 .js/.ts)**
```javascript
export const myGlobalState = $state({ // Svelte 5의 $state 사용 예시
	user: {
		// ...
	}
	// ...
});
```

대부분의 경우 이건 괜찮지만, 서버 사이드 렌더링(SSR) 중에 상태를 변경하면 (권장되진 않지만 얼마든지 가능!) 위험이 따릅니다...

**App.svelte**
```svelte
<script lang="ts">
	import { myGlobalState } from './state.svelte.js';

	let { data } = $props(); // Svelte 5의 $props 사용 예시

	if (data.user) {
		myGlobalState.user = data.user; // SSR 시 문제 발생 가능 지점
	}
</script>
```

...그러면 그 데이터가 다음 사용자에게 노출될 수 있습니다. Context는 요청 간에 공유되지 않기 때문에 이 문제를 해결합니다.

---

**얘 뭐 하는 애냐?**

Svelte의 Context API는 컴포넌트 트리 안에서 데이터를 쉽게 공유하게 해주는 메커니즘입니다. 부모가 어떤 값을 "이거 우리 집안사람들만 써!" 하고 설정해두면, 그 아래 있는 자식이나 손자 컴포넌트들이 중간 다리 역할 하는 컴포넌트들한테 일일이 "이거 좀 전달해주세요" 부탁할 필요 없이 바로 꺼내 쓸 수 있게 해줍니다. 한마디로 "우리집 비밀금고" 같은 거죠.

**왜 쓰는데?**

1.  **Props 드릴링 지옥 탈출:** 이게 핵심. 데이터 하나 전달하자고 중간에 있는 수많은 컴포넌트들에게 props를 계속 넘기고 또 넘기는, 소위 'props 드릴링'이라는 개노가다를 안 해도 됩니다. "택배 왔습니다! 옆집에 전달 좀... 또 옆집에 전달 좀..." 이런 거 이제 그만!
2.  **느슨한 결합, 유연한 구조:** 부모랑 자식이 직접 얼굴 맞대고 있지 않아도, 예를 들어 `<slot>`을 통해 내용물이 동적으로 채워지는 구조에서도 데이터 공유가 깔끔해집니다. "얼굴 한번 못 봤지만, 우린 통하는 게 있다니까?"
3.  **SSR 환경에서의 안전한 상태 공유:** 일반적인 자바스크립트 모듈 전역 변수는 서버 사이드 렌더링할 때 여러 사용자 요청 간에 데이터가 꼬일 수 있는 위험이 있습니다. Context는 각 요청별로 격리되어서 이런 문제를 막아줍니다. "내 정보는 소중하니까, 너만 봐."

**언제 불려 나오냐?**

*   `setContext(key, value)`: 주로 부모 컴포넌트의 `<script>` 블록 안에서, 컴포넌트가 처음 만들어질 때 호출됩니다. "금고에 보관할 물건이랑 비밀번호 설정!"
*   `getContext(key)`: 자식 컴포넌트의 `<script>` 블록 안에서, 역시 컴포넌트 초기화 시점에 호출해서 부모가 설정한 값을 가져옵니다. "비밀번호 대고 물건 찾아가자!"
*   `hasContext(key)`: `getContext` 하기 전에 해당 `key`로 설정된 컨텍스트가 있는지 확인사살하고 싶을 때 씁니다.
*   `getAllContexts()`: 현재 컴포넌트에서 접근 가능한 모든 컨텍스트를 객체 형태로 싹 다 긁어모을 때 씁니다. 디버깅할 때나 가끔 유용.

**쓸 때 꿀팁 및 주의사항:**

*   **Key는 유니크하게:** `key` 값은 문자열도 되지만, 다른 라이브러리나 코드랑 겹치지 않도록 `Symbol()`이나 고유한 객체를 쓰는 게 안전빵입니다. "이름표는 아무나 못 알아보게 특수 제작으로!"
*   **`setContext`는 부모에서, `getContext`는 자식에서! 그리고 `setContext`는 초기화 때만!**: 이 순서 어기면 에러 나거나 값 못 받습니다. 특히 `setContext`는 컴포넌트 만들어질 때 딱 한 번만 의미가 있고, 나중에 호출해도 이미 만들어진 자식들한테는 영향 못 줍니다.
*   **반응성 객체 업데이트 시 주의:** 컨텍스트에 객체(`$state` 포함)를 넣고 그 객체의 *내부 속성*을 바꾸면(`counter.count++`) 자식들도 잘 알아채고 반응합니다. 하지만 객체 *자체를 통째로 새 걸로 바꿔치기*하면(`counter = { count: 0 }`) 자식들과의 연결고리가 끊어져서 업데이트가 안 될 수 있습니다. "그릇은 놔두고 내용물만 바꿔!" Svelte 5의 `$state`는 이 부분을 개선했지만, 여전히 내부 속성 변경이 권장됩니다.
*   **타입스크립트 쓸 땐 헬퍼 함수가 국룰:** `key`와 주고받을 값에 타입을 명확히 지정하는 헬퍼 함수(예: `setUserContext`, `getUserContext`)를 만들면 코드도 깔끔해지고 타입 에러도 줄일 수 있습니다. "타입... 그거슨 개발자의 평화를 지켜주는 마법."
*   **Svelte Store랑 뭐가 달라?:** 간단한 전역 상태는 그냥 Store 써도 됩니다. Context는 주로 특정 컴포넌트 하위 트리 전체에 값을 공급하거나, SSR 시 요청별 데이터 격리가 중요할 때 더 빛을 발합니다. "망치로 모든 못을 박을 순 없지."
*   **SSR 시 데이터 오염 방지:** 이게 Context의 중요한 장점 중 하나. 일반 JS 모듈 전역 변수는 서버에서 여러 요청 처리하다 보면 데이터 섞일 수 있는데, Context는 요청마다 독립적이라 안전합니다. "내 정보가 옆집 철수한테 가면 안 되잖아?"