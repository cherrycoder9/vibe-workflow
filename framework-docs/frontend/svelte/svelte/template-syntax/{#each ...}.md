`{#each ...}` 블록은 값을 반복해서 화면에 뿌릴 때 쓰는 녀석입니다. 배열, 유사 배열 객체 (그러니까 `length` 속성 가진 놈들), 아니면 `Map`이나 `Set` 같은 이터러블 객체, 즉 `Array.from`으로 바꿀 수 있는 건 뭐든지 다 돌릴 수 있습니다.

```svelte
<h1>장바구니</h1>
<ul>
	{#each items as item}
		<li>{item.name} x {item.qty}</li>
	{/each}
</ul>
```

`{#each}` 블록은 `array.map(...)` 콜백의 두 번째 인자처럼 인덱스도 지정할 수 있습니다:

```svelte
{#each items as item, i}
	<li>{i + 1}: {item.name} x {item.qty}</li>
{/each}
```

**키(Key)를 사용하는 `{#each}` 블록**

```svelte
{#each expression as name (key)}...{/each}
{#each expression as name, index (key)}...{/each}
```

만약 각 리스트 아이템을 고유하게 식별하는 `key` 표현식을 제공하면, Svelte는 데이터가 바뀔 때 리스트 끝에 아이템을 추가하거나 제거하고 중간 상태를 업데이트하는 대신, 아이템을 삽입, 이동, 삭제하는 방식으로 똑똑하게 리스트를 업데이트합니다. "똑똑하게 찾아서 필요한 것만 바꾼다, 이거지."

`key`는 어떤 객체든 될 수 있지만, 문자열이나 숫자를 쓰는 게 좋습니다. 왜냐하면 객체 자체가 바뀌어도 고유성을 유지할 수 있거든요.

```svelte
{#each items as item (item.id)}
	<li>{item.name} x {item.qty}</li>
{/each}

<!-- 인덱스 값 추가 버전 -->
{#each items as item, i (item.id)}
	<li>{i + 1}: {item.name} x {item.qty}</li>
{/each}
```

`{#each}` 블록 안에서는 구조 분해 할당이나 나머지 패턴(rest patterns)도 자유롭게 쓸 수 있습니다. "코드를 더 깔끔하게 만들 수 있는 꿀팁!"

```svelte
{#each items as { id, name, qty }, i (id)}
	<li>{i + 1}: {name} x {qty}</li>
{/each}

{#each objects as { id, ...rest }}
	<li><span>{id}</span><MyComponent {...rest} /></li>
{/each}

{#each items as [id, ...rest]}
	<li><span>{id}</span><MyComponent values={rest} /></li>
{/each}
```

**아이템 이름 없는 `{#each}` 블록**

```svelte
{#each expression}...{/each}
{#each expression, index}...{/each}
```

그냥 뭔가를 n번 렌더링하고 싶을 때는 `as` 부분을 생략할 수 있습니다 (데모 참고): "반복 횟수만 중요할 때!"

```svelte
<div class="chess-board">
	{#each { length: 8 } as _, rank}  <!-- Svelte 5부터는 as _, rank 처럼 명시적 이름 필요 -->
		{#each { length: 8 } as _, file} <!-- Svelte 5부터는 as _, file 처럼 명시적 이름 필요 -->
			<div class:black={(rank + file) % 2 === 1}></div>
		{/each}
	{/each}
</div>
```
*(주의: Svelte 5부터는 `{#each { length: 8 }}` 와 같이 `as name` 없이 사용하는 구문이 변경되어, `{#each { length: 8 } as _}` 또는 `{#each { length: 8 } as item}` 처럼 명시적인 변수 이름(사용하지 않더라도 `_` 와 같이)을 지정해야 합니다. 위 예제는 Svelte 5 이전 스타일일 수 있으니, 최신 Svelte 버전에 맞춰 사용하세요.)*

**`{:else}` 블록**

```svelte
{#each expression as name}...{:else}...{/each}
```

`{#each}` 블록은 리스트가 비어있을 때 렌더링되는 `{:else}` 절을 가질 수도 있습니다. "데이터 없으면 썰렁하니까 뭐라도 보여주자."

```svelte
{#each todos as todo}
	<p>{todo.text}</p>
{:else}
	<p>오늘 할 일 없음! 놀자!</p>
{/each}
```

---

**얘 뭐 하는 애냐? (기능 및 목적)**
`{#each ...}` 블록은 Svelte 템플릿 안에서 자바스크립트의 `for...of` 루프나 `Array.prototype.map()` 메소드처럼 데이터 묶음을 순회하면서 HTML 요소를 반복적으로 만들어내는 녀석입니다. 배열이나 객체 리스트 같은 데이터를 받아서 각 항목마다 지정된 HTML 구조를 찍어내는 "붕어빵 틀"이라고 생각하면 쉽습니다. 동적인 리스트, 표, 반복되는 UI 요소 만들 때 필수죠.

**왜 쓰는데?**
1.  **데이터 기반 UI 렌더링**: 자바스크립트 배열이나 객체 데이터를 HTML로 쉽게 변환. "데이터만 주면 알아서 그려줄게."
2.  **코드 간결성**: 복잡한 DOM 조작 없이 선언적으로 리스트 UI 구현. "바닐라 JS로 `createElement`랑 `appendChild` 반복하던 시절은 갔다."
3.  **반응성**: 원본 데이터가 바뀌면 Svelte가 알아서 화면을 업데이트. `key`를 쓰면 더 효율적으로 변경 사항만 쏙쏙 반영합니다. "데이터 바뀌면 화면도 알아서 착착!"

**언제 불려 나오냐?**
*   할 일 목록, 상품 리스트, 댓글 목록 등 아이템 목록을 화면에 표시할 때.
*   데이터 개수만큼 특정 UI 요소를 반복해서 보여주고 싶을 때 (예: 별점 표시).
*   체스판처럼 정해진 횟수만큼 행과 열을 반복해서 구조를 만들 때.

**쓸 때 꿀팁 및 주의사항:**
*   **`key`는 생명줄 (특히 리스트 아이템이 추가/삭제/순서 변경될 때)**: `(item.id)`처럼 각 아이템을 고유하게 식별할 수 있는 `key`를 꼭 쓰세요. 안 그러면 Svelte가 아이템들을 제대로 구분 못 해서 리스트가 변경될 때 엉뚱한 애니메이션이 나오거나 상태가 꼬일 수 있습니다. "key 없으면 DOM 재활용하다가 다 망가진다!" `key`로는 문자열이나 숫자가 좋습니다. 객체 자체를 `key`로 쓰면 객체 참조가 바뀔 때마다 다른 아이템으로 인식해서 비효율적일 수 있습니다.
*   **인덱스는 `key` 대용으로 부적합**: `{#each items as item, i (i)}`처럼 인덱스를 `key`로 쓰는 건 리스트 아이템이 중간에 추가/삭제되거나 순서가 바뀌면 문제가 생깁니다. 아이템의 고유 ID를 사용하세요. "인덱스는 그냥 순서일 뿐, 신분증이 아니야."
*   **구조 분해 할당으로 가독성 UP**: `{#each items as { id, name, qty }}`처럼 쓰면 `item.name`, `item.qty` 대신 바로 `name`, `qty`로 접근할 수 있어서 코드가 깔끔해집니다.
*   **`{:else}` 블록으로 사용자 경험 개선**: 데이터가 없을 때 "데이터 없음" 메시지라도 보여주는 게 사용자에게 친절합니다. "빈 화면은 너무 삭막하잖아."
*   **횟수 반복 시 주의 (Svelte 5+):** `{#each { length: 5 }}` 같은 구문은 Svelte 5부터 `{#each { length: 5 } as _}` 또는 `{#each Array(5) as _}` 처럼 명시적으로 `as name` (사용하지 않더라도 `_` 같은 플레이스홀더)을 써줘야 합니다. 옛날 코드 그대로 쓰면 에러 날 수 있으니 버전 확인 필수! "업데이트 내용 확인 안 하면 나중에 피곤해진다."