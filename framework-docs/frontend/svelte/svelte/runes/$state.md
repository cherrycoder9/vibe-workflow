`$state` 룬(rune)은 반응형 상태(reactive state)를 만드는 녀석입니다. 이게 뭐냐고요? 값이 바뀌면 UI가 알아서 착착 업데이트되는 마법 같은 거죠.

```svelte
<script>
	let count = $state(0); // count야, 이제부터 넌 반응형이다!
</script>

<button onclick={() => count++}>
	클릭 횟수: {count}
</button>
```

다른 프레임워크처럼 복잡한 API 쓸 필요 없습니다. `count`는 그냥 숫자예요. 객체나 함수 같은 거 아니고, 다른 변수 다루듯이 업데이트하면 됩니다. "어렵게 생각할 거 없어, 그냥 변수야!"

**깊은 상태 (Deep state)**

`$state`를 배열이나 단순 객체에 쓰면, 그 속까지 전부 반응형으로 만들어주는 "깊은 반응형 프록시(proxy)"가 됩니다. 이 프록시 덕분에 Svelte는 여러분이 객체 속성을 읽거나 쓸 때, 심지어 `array.push(...)` 같은 메서드를 쓸 때도 알아채고, 필요한 부분만 콕 집어 업데이트할 수 있습니다. "네 속마음까지 다 보고 있다!"

클래스 인스턴스는 프록시로 안 만듭니다. 대신 직접 정의한 클래스 안에 반응형 상태 필드를 만들 수 있죠. Svelte는 `Set`이나 `Map` 같은 내장 객체들의 반응형 버전도 `svelte/reactivity`에서 가져다 쓸 수 있게 제공합니다.

Svelte는 배열이나 단순 객체가 아닌 다른 걸 만날 때까지 재귀적으로 상태를 프록시화합니다. 예를 들어 이런 경우...

```javascript
let todos = $state([
	{
		done: false,
		text: '할 일 더 추가하기'
	}
]);
```

...개별 `todo` 항목의 속성을 바꾸면, 그 특정 속성에 의존하는 UI의 모든 부분이 업데이트됩니다.

```javascript
todos[0].done = !todos[0].done; // 첫 번째 할 일의 완료 상태를 뒤집어!
```

배열에 새 객체를 밀어 넣으면, 그 객체도 프록시화됩니다.

```javascript
todos.push({ // 새 할 일 추가!
	done: false,
	text: '점심 먹기'
});
```

프록시의 속성을 업데이트할 때, 원본 객체는 건드리지 않습니다. "원본은 소중하니까."

**주의!** 반응형 값을 구조 분해 할당(destructure)하면, 그 참조는 반응형이 아닙니다. 일반 자바스크립트처럼 구조 분해하는 시점의 값으로 고정돼요.

```javascript
let { done, text } = todos[0]; // 이 순간의 done, text 값을 가져옴

// 이건 `done` 변수 값에 영향 안 줌. todos[0].done만 바뀜.
todos[0].done = !todos[0].done;
```

**클래스 (Classes)**

`$state`는 클래스 필드(public이든 private이든)나 생성자 안에서 속성에 처음 값을 할당할 때도 쓸 수 있습니다.

```javascript
class Todo {
	done = $state(false); // done은 이제 반응형!

	constructor(text) {
		this.text = $state(text); // text도 반응형!
	}

	reset() {
		this.text = '';
		this.done = false;
	}
}
```

컴파일러는 `done`과 `text`를 클래스 프로토타입의 `get`/`set` 메서드로 바꿔서 비공개 필드를 참조하게 만듭니다. 그래서 이 속성들은 열거할 수 없게(not enumerable) 됩니다.

자바스크립트에서 메서드를 호출할 때 `this`의 값이 중요합니다. 아래 코드는 작동 안 해요. `reset` 메서드 안의 `this`가 `Todo` 인스턴스가 아니라 `<button>`을 가리키기 때문이죠. "야, 너 누구냐?"

```svelte
<button onclick={todo.reset}> <!-- 이거 안됨 -->
	초기화
</button>
```

인라인 함수를 쓰거나...

```svelte
<button onclick={() => todo.reset()}> <!-- 이렇게 하거나 -->
	초기화
</button>
```

...클래스 정의에서 화살표 함수를 쓰면 됩니다.

```javascript
class Todo {
	done = $state(false);

	constructor(text) {
		this.text = $state(text);
	}

	reset = () => { // 화살표 함수로 this 고정!
		this.text = '';
		this.done = false;
	}
}
```

**`$state.raw`**

객체나 배열을 깊게 반응형으로 만들고 싶지 않을 때 `$state.raw`를 씁니다. "넌 그냥 그대로 있어."

`$state.raw`로 선언된 상태는 직접 수정할 수 없고, 오직 재할당만 가능합니다. 즉, 객체의 속성을 바꾸거나 `push` 같은 배열 메서드를 쓰는 대신, 업데이트하고 싶으면 객체나 배열 전체를 새것으로 갈아 끼워야 합니다.

```javascript
let person = $state.raw({
	name: '헤라클레이토스',
	age: 49
});

// 이건 효과 없음. 꿈쩍도 안 함.
person.age += 1;

// 이건 작동함. 새 사람으로 갈아 끼웠으니까.
person = {
	name: '헤라클레이토스',
	age: 50
};
```

어차피 수정할 계획이 없는 큰 배열이나 객체에 이걸 쓰면, 반응형으로 만드는 비용을 줄여서 성능을 향상시킬 수 있습니다. 참고로, `raw` 상태는 반응형 상태를 포함할 수 있습니다 (예: 반응형 객체로 이루어진 `raw` 배열). "겉바속촉 같은 건가?"

**`$state.snapshot`**

깊은 반응형 `$state` 프록시의 정적 스냅샷을 찍고 싶으면 `$state.snapshot`을 씁니다. "찰칵! 지금 이 순간을 저장."

```svelte
<script>
	let counter = $state({ count: 0 });

	function onclick() {
		// Proxy { ... } 대신 { count: ... }가 찍힘
		console.log($state.snapshot(counter));
	}
</script>
```

`structuredClone`처럼 프록시를 예상하지 않는 외부 라이브러리나 API에 상태를 전달할 때 유용합니다.

**함수에 상태 전달하기 (Passing state into functions)**

자바스크립트는 값에 의한 전달(pass-by-value) 언어입니다. 함수를 호출할 때, 인자는 변수 자체가 아니라 그 값입니다. 즉:

```javascript
// index.js
function add(a: number, b: number) {
	return a + b;
}

let a = 1;
let b = 2;
let total = add(a, b);
console.log(total); // 3

a = 3; // a, b 바꿔도
b = 4;
console.log(total); // total은 여전히 3! "난 처음 값만 기억해."
```

만약 `add` 함수가 `a`와 `b`의 현재 값을 알고 싶고, 현재 `total` 값을 반환하고 싶다면, 대신 함수를 사용해야 합니다:

```javascript
// index.js
function add(getA: () => number, getB: () => number) {
	return () => getA() + getB(); // 함수를 반환해서 호출 시점에 값 계산
}

let a = 1;
let b = 2;
let total = add(() => a, () => b); // a, b를 가져오는 함수를 넘김
console.log(total()); // 3

a = 3;
b = 4;
console.log(total()); // 7. "이제야 제대로 나오네!"
```

Svelte의 상태도 다르지 않습니다. `$state` 룬으로 선언된 것을 참조할 때...

```javascript
let a = $state(1);
let b = $state(2);
```

...여러분은 그것의 현재 값에 접근하는 겁니다.

'함수'라는 건 범위가 넓습니다. 프록시의 속성이나 `get`/`set` 속성도 포함돼요.

```javascript
// index.js
function add(input: { a: number, b: number }) {
	return {
		get value() { // getter로 현재 값 반영
			return input.a + input.b;
		}
	};
}

let input = $state({ a: 1, b: 2 });
let total = add(input);
console.log(total.value); // 3

input.a = 3;
input.b = 4;
console.log(total.value); // 7
```

...하지만 이런 코드를 짜고 있다면, 그냥 클래스를 쓰는 걸 고려해보세요. "굳이 이렇게까지...?"

**모듈 간 상태 전달 (Passing state across modules)**

`.svelte.js`나 `.svelte.ts` 파일에서 상태를 선언할 수 있지만, 직접 재할당되지 않는 경우에만 그 상태를 `export` 할 수 있습니다. 즉, 이렇게는 안 됩니다:

```javascript
// state.svelte.js
export let count = $state(0); // 이런 식으로 export하면...

export function increment() {
	count += 1; // 여기서 count를 직접 바꾸면 문제가 생김
}
```

왜냐하면 `count`에 대한 모든 참조는 Svelte 컴파일러에 의해 변환되기 때문입니다. 위 코드는 대략 아래와 비슷하게 바뀝니다:

```javascript
// state.svelte.js (컴파일러가 변환한 모습, 대략)
export let count = $.state(0); // 내부적으로 $.state() 같은 걸로 감싸짐

export function increment() {
	$.set(count, $.get(count) + 1); // 값 가져오고 설정하는 것도 내부 함수 호출
}
```

Svelte 플레이그라운드의 'JS Output' 탭을 클릭하면 Svelte가 생성하는 코드를 볼 수 있습니다.

컴파일러는 한 번에 한 파일만 처리하기 때문에, 다른 파일에서 `count`를 가져오면 Svelte는 각 참조를 `$.get`과 `$.set`으로 감싸야 한다는 것을 모릅니다.

```javascript
// 다른 파일
import { count } from './state.svelte.js';

console.log(typeof count); // 'number'가 아니라 'object'가 찍힘. "어라, 네 정체가 뭐냐?"
```

이러면 모듈 간에 상태를 공유하는 두 가지 방법이 남습니다. 재할당하지 않거나...

```javascript
// state.svelte.js
// 이건 허용됨 - `counter` 자체가 아니라 `counter.count`를
// 업데이트하기 때문에 Svelte가 `$.state`로 감싸지 않음
export const counter = $state({ // 객체 자체를 export
	count: 0
});

export function increment() {
	counter.count += 1; // 객체 내부 속성 변경은 OK
}
```

...직접 `export` 하지 않는 겁니다:

```javascript
// state.svelte.js
let count = $state(0); // 모듈 내부에서만 사용

export function getCount() { // getter 함수로 값 제공
	return count;
}

export function increment() { // 상태 변경 함수 제공
	count += 1;
}
```

---

**얘 뭐 하는 애냐? (기능 및 목적)**
`$state`는 Svelte 5 (룬 도입 이후)에서 "이 값 바뀌면 화면도 같이 바뀌게 해주세요!"라고 선언하는 마법 주문입니다. 변수 앞에 `$state()`를 붙여주면, 그 변수가 바뀔 때마다 Svelte가 알아서 관련된 UI 부분을 업데이트해줍니다. 한마디로, 데이터와 화면을 실시간으로 동기화시켜주는 핵심 장치죠. "데이터 바뀌면 화면도 새로고침? 그건 옛날 얘기!"

**왜 쓰는데?**
1.  **반응성 확보**: 변수 값 변경만으로 UI가 자동으로 업데이트되니 코드가 간결해지고 직관적이 됩니다. "내가 값만 바꾸면 나머진 Svelte가 알아서!"
2.  **쉬운 사용법**: `useState` 같은 훅(hook)이나 복잡한 상태 관리 라이브러리 없이, 그냥 일반 변수처럼 쓰고 할당하면 끝. "배울 게 확 줄었네?"
3.  **깊은 반응성 (옵션)**: 객체나 배열 내부의 값 변경까지 감지해서 세밀한 업데이트가 가능합니다. 물론 `$state.raw`로 얕은 반응성만 선택할 수도 있고요. "속까지 다 보거나, 겉만 보거나. 선택은 자유!"
4.  **클래스와의 통합**: 클래스 필드에도 `$state`를 써서 객체 지향 프로그래밍과 반응형 상태를 자연스럽게 결합할 수 있습니다.

**언제 불려 나오냐?**
Svelte 컴포넌트의 `<script>` 태그 안이나 `.svelte.js`/`.svelte.ts` 파일에서 UI와 연동되어 값이 변할 수 있는 모든 변수를 선언할 때 씁니다. 예를 들어, 사용자가 버튼을 클릭해서 숫자를 증가시키거나, 입력 필드의 내용을 저장하거나, API에서 받아온 데이터를 화면에 표시할 때 등등 "이 값 바뀌면 화면에 티 나야 한다!" 싶으면 무조건 `$state`를 붙입니다.

**쓸 때 꿀팁 및 주의사항:**
*   **단순함이 최고**: `$state(0)`처럼 쓰면 `count`는 그냥 숫자입니다. `.value` 같은 거 붙일 필요 없어요. "복잡한 건 죄악이다!"
*   **객체/배열은 프록시**: `$state({ a: 1 })`이나 `$state()`로 만든 건 프록시 객체입니다. 내부 값을 바꿔도 반응성이 유지되지만, 원본 객체는 건드리지 않아요.
*   **구조 분해 할당의 함정**: `let { prop } = $state({ prop: 1 });` 이렇게 하면 `prop`은 반응성을 잃습니다. 그냥 `state.prop`으로 쓰세요. "편하자고 하다가 큰코다친다!"
*   **`$state.raw`는 성능 개선용**: 진짜 큰 데이터인데 내부 값 변경 감지가 필요 없다면 `$state.raw`로 프록시 비용을 아끼세요. 단, 이땐 통째로 갈아 끼워야 업데이트됩니다.
*   **`$state.snapshot`은 외부 연동용**: 프록시가 아닌 순수 객체가 필요할 때 (예: `JSON.stringify` 전에) 사용합니다.
*   **모듈 간 공유 시 주의**: `$state`로 만든 변수를 직접 `export let count = $state(0);` 하고 다른 파일에서 `count++` 하려고 하면 안 됩니다. 객체로 감싸서 속성을 바꾸거나, `getter`/`setter` 함수를 `export` 하세요. "컴파일러는 만능이 아니다!"
*   **클래스 메서드 `this`**: 클래스 메서드를 이벤트 핸들러로 직접 넘길 때 `this`가 깨질 수 있습니다. 화살표 함수로 메서드를 정의하거나, 인라인 함수 `() => instance.method()`를 쓰세요. "자바스크립트 `this`는 언제나 골칫거리."