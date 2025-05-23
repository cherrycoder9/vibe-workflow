스토어(Store)

스토어는 간단한 '스토어 계약(store contract)'을 통해 값에 반응적으로 접근할 수 있게 해주는 객체입니다. `svelte/store` 모듈에는 이 계약을 충족하는 최소한의 스토어 구현체들이 들어있습니다.

스토어를 참조하고 있다면, 컴포넌트 내부에서 스토어 이름 앞에 `$` 문자를 붙여 그 값에 접근할 수 있습니다. 이렇게 하면 Svelte는 `$`가 붙은 변수를 선언하고, 컴포넌트가 초기화될 때 스토어를 구독(subscribe)했다가 적절한 시점에 구독을 해제(unsubscribe)합니다.

`$`가 붙은 변수에 값을 할당하려면 해당 변수가 쓰기 가능한 스토어(writable store)여야 하며, 이 경우 스토어의 `.set` 메소드가 호출됩니다.

주의할 점은, 스토어는 반드시 컴포넌트 최상단 레벨에서 선언해야 합니다. `if` 블록이나 함수 내부 같은 곳에서는 안 됩니다.

스토어 값을 나타내지 않는 일반 지역 변수에는 `$` 접두사를 붙이면 안 됩니다.

```svelte
<script>
	import { writable } from 'svelte/store';

	const count = writable(0); // count 스토어 만들고 초기값 0으로 세팅
	console.log($count); // 0 출력 (자동 구독 개꿀)

	count.set(1); // count 스토어 값을 1로 변경
	console.log($count); // 1 출력

	$count = 2; // $ 접두사로 직접 값 할당 (내부적으로 set 호출)
	console.log($count); // 2 출력
</script>
```

**언제 스토어를 써야 할까?**

Svelte 5 이전에는 여러 컴포넌트 간에 반응적인 상태를 공유하거나 로직을 분리할 때 스토어가 거의 유일한 해결책이었습니다. 하지만 룬(Runes)이 등장하면서 이런 용도는 많이 줄었습니다.

*   로직을 분리할 때는 룬의 범용적인 반응성을 활용하는 것이 좋습니다. 룬은 컴포넌트 최상단이 아닌 곳에서도 쓸 수 있고, 심지어 `.svelte.js`나 `.svelte.ts` 확장자를 가진 자바스크립트 또는 타입스크립트 파일에 넣을 수도 있습니다.
*   상태를 공유할 때는 필요한 값들을 담은 `$state` 객체를 만들고 그 상태를 조작하면 됩니다.

    `state.svelte.js` (또는 `.svelte.ts`)
    ```javascript
    export const userState = $state({
    	name: '홍길동',
    	/* ... */
    });
    ```

    `App.svelte`
    ```svelte
    <script lang="ts">
    	import { userState } from './state.svelte.js';
    </script>

    <p>사용자 이름: {userState.name}</p>
    <button on:click={() => {
    	userState.name = '전우치';
    }}>
    	이름 변경
    </button>
    ```

그래도 스토어는 복잡한 비동기 데이터 스트림을 다루거나 값 업데이트 또는 변경 감지를 좀 더 수동으로 제어해야 할 때 여전히 좋은 해결책입니다. RxJS에 익숙하고 그 지식을 재활용하고 싶다면, `$` 문법이 유용할 겁니다.

**`svelte/store`**

`svelte/store` 모듈은 스토어 계약을 만족하는 최소한의 스토어 구현체를 제공합니다. 외부에서 값을 업데이트할 수 있는 스토어, 내부에서만 업데이트 가능한 스토어, 그리고 여러 스토어를 조합하거나 파생시키는 스토어를 만드는 메소드들을 제공합니다.

**`writable`**

컴포넌트 '외부'에서 값을 설정할 수 있는 스토어를 만드는 함수입니다. `set`과 `update` 메소드가 추가된 객체로 생성됩니다.

*   `set`: 인자로 받은 값으로 스토어 값을 설정합니다. 기존 값과 다를 경우에만 업데이트됩니다.
*   `update`: 콜백 함수를 인자로 받습니다. 이 콜백은 기존 스토어 값을 인자로 받아 새 값을 반환하여 스토어에 설정합니다.

```javascript
import { writable } from 'svelte/store';

const count = writable(0);

count.subscribe((value) => {
	console.log(value); // 값이 바뀔 때마다 출력
}); // '0' 출력

count.set(1); // '1' 출력

count.update((n) => n + 1); // '2' 출력
```

만약 두 번째 인자로 함수를 전달하면, 구독자 수가 0에서 1로 될 때 (1에서 2로 될 때는 아님) 그 함수가 호출됩니다. 이 함수는 스토어 값을 변경하는 `set` 함수와, 기존 값으로부터 새 값을 계산하는 콜백을 받는 `update` 함수를 인자로 받습니다. 그리고 구독자 수가 1에서 0으로 줄어들 때 호출될 `stop` 함수를 반환해야 합니다.

```javascript
import { writable } from 'svelte/store';

const count = writable(0, () => {
	console.log('구독자 생김!'); // 첫 구독 시 실행
	return () => console.log('구독자 없음...'); // 마지막 구독 해제 시 실행
});

count.set(1); // 아직 구독자가 없어서 아무 일도 안 일어남

const unsubscribe = count.subscribe((value) => {
	console.log(value);
}); // '구독자 생김!' 출력, 그 다음 '1' 출력

unsubscribe(); // '구독자 없음...' 출력
```

`writable` 스토어의 값은 페이지 새로고침 등으로 인해 소멸되면 사라집니다. 하지만 `localStorage` 등에 값을 동기화하는 로직을 직접 작성할 수 있습니다. "데이터 날아가면 내 책임 아니라고~"

**`readable`**

'외부'에서 값을 설정할 수 없는 스토어를 만듭니다. 첫 번째 인자는 스토어의 초기값이고, 두 번째 인자는 `writable`의 두 번째 인자와 동일하게 작동합니다. (구독자 수 변화에 따른 콜백)

```javascript
import { readable } from 'svelte/store';

// 1초마다 현재 시간으로 업데이트되는 스토어
const time = readable(new Date(), (set) => {
	set(new Date()); // 초기값 즉시 설정

	const interval = setInterval(() => {
		set(new Date());
	}, 1000);

	return () => clearInterval(interval); // 구독자 없어지면 인터벌 정리
});

// 1초마다 'tick'과 'tock'을 번갈아 출력하는 스토어
const ticktock = readable('tick', (set, update) => {
	const interval = setInterval(() => {
		update((sound) => (sound === 'tick' ? 'tock' : 'tick'));
	}, 1000);

	return () => clearInterval(interval);
});
```

**`derived`**

하나 이상의 다른 스토어로부터 파생된 스토어를 만듭니다. 콜백 함수는 첫 구독자가 생길 때 처음 실행되고, 그 이후에는 의존하는 스토어들의 값이 변경될 때마다 실행됩니다.

가장 간단한 형태는 단일 스토어를 받고, 콜백이 파생된 값을 반환하는 것입니다.

```javascript
import { derived } from 'svelte/store';

const doubled = derived(a, ($a) => $a * 2); // a 스토어 값의 2배인 스토어
```

콜백은 두 번째 인자로 `set` 함수를, 선택적으로 세 번째 인자로 `update` 함수를 받아 비동기적으로 값을 설정할 수 있습니다. 이 경우, `derived` 함수의 세 번째 인자로 파생 스토어의 초기값을 전달할 수도 있습니다. 초기값을 지정하지 않으면 `undefined`가 됩니다.

```javascript
import { derived } from 'svelte/store';

// a 스토어 값이 변경되면 1초 후에 그 값으로 설정되는 스토어 (초기값은 2000)
const delayed = derived(
	a,
	($a, set) => {
		setTimeout(() => set($a), 1000);
	},
	2000
);

// a 스토어 값이 변경되면 즉시 그 값으로, 1초 후에는 1 증가된 값으로 설정
const delayedIncrement = derived(a, ($a, set, update) => {
	set($a); // 즉시 반영
	setTimeout(() => update((x) => x + 1), 1000); // 1초 후 +1
	// $a가 값을 생성할 때마다, 이 스토어는 두 개의 값을 생성: 즉시 $a, 1초 후 $a + 1
});
```

콜백에서 함수를 반환하면, 그 함수는 콜백이 다시 실행되거나 마지막 구독자가 구독을 해제할 때 호출됩니다. (클린업 함수 역할)

```javascript
import { derived } from 'svelte/store';

// frequency 스토어 값에 따라 주기적으로 현재 시간을 업데이트하는 스토어
const tick = derived(
	frequency, // frequency 스토어에 의존
	($frequency, set) => {
		const interval = setInterval(() => {
			set(Date.now());
		}, 1000 / $frequency);

		return () => { // 클린업 함수
			clearInterval(interval);
		};
	},
	2000 // 초기값
);
```

두 경우 모두, 첫 번째 인자로 단일 스토어 대신 스토어 배열을 전달할 수 있습니다.

```javascript
import { derived } from 'svelte/store';

// a와 b 스토어 값의 합계를 갖는 스토어
const summed = derived([a, b], ([$a, $b]) => $a + $b);

// a와 b 스토어 값이 변경되면 1초 후에 그 합계로 설정되는 스토어
const delayed = derived([a, b], ([$a, $b], set) => {
	setTimeout(() => set($a + $b), 1000);
});
```

**`readonly`**

이 간단한 헬퍼 함수는 스토어를 읽기 전용으로 만듭니다. 이 새로운 읽기 전용 스토어를 통해 원본 스토어의 변경 사항을 여전히 구독할 수 있습니다.

```javascript
import { readonly, writable } from 'svelte/store';

const writableStore = writable(1);
const readableStore = readonly(writableStore); // 읽기 전용으로 변환

readableStore.subscribe(console.log);

writableStore.set(2); // 콘솔: 2 (원본 변경 시 읽기 전용도 업데이트)
readableStore.set(2); // 에러! 읽기 전용 스토어에는 set 메소드가 없음
```

**`get`**

일반적으로 스토어의 값은 구독하고 시간의 흐름에 따라 변하는 값을 사용해야 합니다. 가끔 구독하지 않은 스토어의 현재 값을 가져와야 할 때가 있는데, `get`을 사용하면 됩니다.

이것은 구독을 생성하고, 값을 읽은 다음, 구독을 해제하는 방식으로 작동합니다. 따라서 성능에 민감한 코드(hot code path)에서는 권장되지 않습니다. "잠깐 값만 보고 빠지는 거라 자주 쓰면 느려진다!"

```javascript
import { get } from 'svelte/store';

const value = get(store); // store의 현재 값을 한 번만 가져옴
```

**스토어 계약 (Store Contract)**

```javascript
store = {
  subscribe: (subscription: (value: any) => void) => (() => void), // 구독 메소드. 구독 함수를 받고, 구독 해제 함수를 반환.
  set?: (value: any) => void // (선택 사항) 쓰기 가능한 스토어의 경우 값을 설정하는 메소드.
}
```

`svelte/store`에 의존하지 않고도 스토어 계약을 구현하여 자신만의 스토어를 만들 수 있습니다.

1.  스토어는 `.subscribe` 메소드를 가져야 합니다. 이 메소드는 구독 함수를 인자로 받아야 합니다. 이 구독 함수는 `.subscribe` 호출 시 즉시 동기적으로 스토어의 현재 값으로 호출되어야 합니다. 스토어의 값이 변경될 때마다 모든 활성 구독 함수는 나중에 동기적으로 호출되어야 합니다.
2.  `.subscribe` 메소드는 구독 해제 함수를 반환해야 합니다. 구독 해제 함수를 호출하면 해당 구독이 중지되어야 하며, 해당 구독 함수는 스토어에 의해 다시 호출되어서는 안 됩니다.
3.  스토어는 선택적으로 `.set` 메소드를 가질 수 있습니다. 이 메소드는 스토어의 새 값을 인자로 받아야 하며, 모든 활성 구독 함수를 동기적으로 호출해야 합니다. 이러한 스토어를 쓰기 가능한 스토어(writable store)라고 합니다.

RxJS Observable과의 상호 운용성을 위해, `.subscribe` 메소드는 구독 해제 함수를 직접 반환하는 대신 `.unsubscribe` 메소드를 가진 객체를 반환할 수도 있습니다. 그러나 `.subscribe`가 구독을 동기적으로 호출하지 않는 한 (Observable 스펙에서는 필수가 아님), Svelte는 스토어의 값을 호출될 때까지 `undefined`로 간주합니다. "표준 스펙은 지키되, Svelte 눈높이도 맞춰달라 이 말이야."

---

**얘 뭐 하는 애냐?**

Svelte 스토어는 컴포넌트 여기저기서 함께 쓸 수 있는 '값 저장소' 같은 겁니다. 이 저장소의 값이 바뀌면, 그 값을 쓰는 모든 컴포넌트 화면도 알아서 착착 업데이트되는 마법을 부리죠. `$count`처럼 달러($) 표시 하나면 "이 값 바뀌면 나도 알려줘!" 하고 구독 신청 끝. 간단하죠? "데이터 공유? 스토어한테 맡겨!"

**왜 쓰는데?**

1.  **컴포넌트 간 상태 공유:** 부모-자식 관계가 복잡하거나, 아예 상관없는 컴포넌트끼리 데이터를 주고받아야 할 때 씁니다. Props 드릴링(props drilling, 여러 계층으로 props 넘겨주기)으로 고통받기 싫을 때 아주 유용하죠. "야, 저~기 옆집 컴포넌트에도 이 데이터 좀 보내줘!"
2.  **애플리케이션 전역 상태 관리:** 사용자 로그인 정보, 테마 설정, 장바구니 내용처럼 앱 전체에서 필요한 데이터를 한 곳에서 관리하고 싶을 때 씁니다. "우리 앱 대장 데이터는 내가 다 갖고 있다!"
3.  **복잡한 비동기 로직 처리:** 서버에서 데이터를 가져오거나, 웹소켓으로 실시간 데이터를 받을 때, 이 데이터 흐름을 스토어로 만들어서 깔끔하게 처리할 수 있습니다. RxJS 같은 거 써봤으면 더 쉽게 적응 가능.
4.  **Svelte 5 이전의 유산 (하지만 여전히 유효):** Svelte 5에서 룬(Runes)이라는 더 간편한 상태 관리 방식이 나왔지만, 여전히 스토어는 특정 상황(특히 복잡한 비동기나 수동 제어)에서 강력합니다. "구관이 명관일 때도 있는 법."

**언제 불려 나오냐?**

*   `writable()`: 외부에서 값을 마음대로 바꾸고 싶을 때 씁니다. `set()` 메소드로 새 값을 통째로 넣거나, `update()` 메소드로 기존 값 기반으로 새 값을 계산해서 넣습니다.
*   `readable()`: 외부에서는 값을 못 바꾸게 막고, 스토어 내부 로직(예: 타이머, 외부 API 호출)으로만 값이 바뀌게 하고 싶을 때 씁니다. "읽기만 하세요. 수정은 안 됩니다."
*   `derived()`: 이미 있는 스토어 값을 가지고 새로운 값을 계산해서 또 다른 스토어를 만들고 싶을 때 씁니다. 예를 들어, `firstName` 스토어와 `lastName` 스토어를 합쳐서 `fullName` 스토어를 만드는 거죠. "쟤네 둘이 합치면 뭐가 나올까?"
*   컴포넌트에서 `$storeName` 형태로 쓰면, 컴포넌트가 생길 때 자동으로 구독 시작, 없어질 때 자동으로 구독 해제됩니다.

**쓸 때 꿀팁 및 주의사항:**

*   **`$`는 마법의 약자, 하지만 남용은 금물:** 컴포넌트 안에서 `$storeName`은 정말 편하지만, 이게 실제로는 `storeName.subscribe(...)` 코드를 Svelte가 알아서 넣어주는 겁니다. 너무 많이 쓰면 내부적으로 구독/해제 로직이 많아질 수 있다는 점은 알아두세요.
*   **최상단 선언은 국룰:** `const count = writable(0);` 같은 스토어 선언은 `<script>` 태그 바로 아래, 함수나 `if`문 바깥에 해야 합니다. 안 그러면 Svelte 컴파일러가 화냅니다. "스토어는 VIP석에 모셔라!"
*   **`writable`의 두 번째 인자 (start/stop 알림):** 구독자가 처음 생기거나 마지막 구독자가 사라질 때 특정 작업을 해야 한다면 (예: 외부 리소스 연결/해제), `writable`의 두 번째 인자로 함수를 넘겨서 처리할 수 있습니다. 이거 은근 꿀 기능입니다.
*   **`get()`은 최후의 수단:** 가끔 구독 없이 스토어 값을 딱 한 번만 가져오고 싶을 때 `get(storeName)`을 쓰는데, 이건 내부적으로 구독-값확인-해제 과정을 거치므로 성능이 중요한 곳에서는 피하는 게 좋습니다. "꼭 필요할 때만 살짝 보고 튀어!"
*   **룬(Runes)과의 관계:** Svelte 5부터는 `$state`나 `$derived` 같은 룬으로 더 간결하게 상태 관리를 할 수 있는 경우가 많습니다. 하지만 스토어는 여전히 복잡한 시나리오나 RxJS 통합 등에 유용하니, 상황에 맞게 골라 쓰면 됩니다. "새로운 무기가 나왔다고 옛날 칼을 버릴 필요는 없지."
*   **SSR 환경에서는 주의:** 서버 사이드 렌더링 시에는 `document`나 `window`처럼 브라우저 전용 객체가 없듯이, 스토어의 일부 기능(특히 `readable`에서 `setInterval` 같은 브라우저 API 사용 시)이 문제를 일으킬 수 있습니다. `onMount` 등을 활용해 브라우저 환경에서만 실행되도록 처리해야 합니다.