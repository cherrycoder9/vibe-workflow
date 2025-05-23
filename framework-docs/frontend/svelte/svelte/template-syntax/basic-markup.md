Svelte 컴포넌트 안의 마크업은 HTML에 약간의 마법을 더한 거라고 생각하면 됩니다. HTML++라고나 할까요?

**태그 (Tags)**
소문자로 시작하는 태그, 예를 들어 `<div>` 같은 건 일반적인 HTML 요소를 나타냅니다. 반면, 대문자로 시작하거나 점(.)을 포함하는 태그, 예를 들어 `<Widget>`이나 `<my.stuff>` 같은 건 Svelte 컴포넌트를 의미하죠. "어이, 이건 그냥 HTML 태그가 아니야, 특별한 놈이라고!" 하는 신호인 셈입니다.

```svelte
<script>
	import Widget from './Widget.svelte'; // Widget 컴포넌트 불러오기
</script>

<div>
	<Widget /> {/* Widget 컴포넌트 사용 */}
</div>
```

**요소 속성 (Element attributes)**
기본적으로 HTML 속성과 똑같이 작동합니다.

```svelte
<div class="foo">
	<button disabled>이 버튼은 못 누름</button>
</div>
```

HTML에서처럼 따옴표 없이 값을 쓸 수도 있고요.

```svelte
<input type=checkbox />
```

속성 값에는 자바스크립트 표현식을 넣을 수 있습니다.

```svelte
<a href="page/{p}">page {p}</a> {/* p 변수 값에 따라 링크 주소가 바뀜 */}
```

아니면 아예 자바스크립트 표현식 그 자체를 값으로 쓸 수도 있죠.

```svelte
<button disabled={!clickable}>...</button> {/* clickable이 false면 버튼 비활성화 */}
```

불리언(Boolean) 속성은 값이 참(truthy)이면 요소에 포함되고, 거짓(falsy)이면 제외됩니다. "있거나 없거나, 둘 중 하나!"

다른 모든 속성은 값이 null이나 undefined (nullish)가 아니면 포함됩니다.

```svelte
<input required={false} placeholder="이 입력 필드는 필수 아님" /> {/* required 속성 빠짐 */}
<div title={null}>이 div는 title 속성 없음</div> {/* title 속성 빠짐 */}
```

단일 표현식을 따옴표로 감싸도 값 파싱 방식에는 영향이 없지만, Svelte 6 버전부터는 해당 값을 문자열로 강제 변환할 예정입니다. "지금은 괜찮은데 나중엔 문자열 된다? 조심해!"

```svelte
<button disabled="{number !== 42}">...</button>
```

속성 이름과 값이 같을 때 (`name={name}`), `{name}` 이렇게 줄여 쓸 수 있습니다. 개발자들 귀차니즘은 못 말리죠.

```svelte
<button {disabled}>...</button>
<!-- <button disabled={disabled}> 와 동일 -->
```

**컴포넌트 속성 (Component props)**
관례적으로, 컴포넌트에 전달하는 값은 '속성(attributes)'보다는 '프로퍼티(properties)' 또는 '프롭스(props)'라고 부릅니다. 속성은 DOM의 특징이니까요. "용어 정리는 확실히 하고 가자고."

요소에서처럼, `name={name}`은 `{name}`으로 줄여 쓸 수 있습니다.

```svelte
<Widget foo={bar} answer={42} text="hello" />
```

**전개 속성 (Spread attributes)**
전개 속성을 사용하면 여러 속성이나 프로퍼티를 한 번에 요소나 컴포넌트에 넘길 수 있습니다. "한 방에 싹 다 넘겨!"

하나의 요소나 컴포넌트에 여러 개의 전개 속성을 일반 속성과 섞어 쓸 수 있습니다. 순서가 중요합니다. 만약 `things.a`가 존재하면 `a="b"`보다 우선하고, `c="d"`는 `things.c`보다 우선합니다. "나중에 온 놈이 이기는 거 알지?"

```svelte
<Widget a="b" {...things} c="d" />
```

**이벤트 (Events)**
`on`으로 시작하는 속성을 요소에 추가하면 DOM 이벤트를 감지할 수 있습니다. 예를 들어, `click` 이벤트를 감지하려면 버튼에 `onclick` 속성을 추가하면 됩니다.

```svelte
<button onclick={() => console.log('clicked')}>눌러봐</button>
```

이벤트 속성은 대소문자를 구분합니다. `onclick`은 `click` 이벤트를, `onClick`은 `Click` 이벤트(다른 이벤트임)를 감지합니다. 이렇게 해야 대문자가 포함된 커스텀 이벤트를 감지할 수 있거든요.

이벤트도 그냥 속성이기 때문에, 일반 속성과 같은 규칙이 적용됩니다:
*   단축형 사용 가능: `<button {onclick}>눌러봐</button>`
*   전개 가능: `<button {...thisSpreadContainsEventAttributes}>눌러봐</button>`

타이밍 측면에서, 이벤트 속성은 항상 바인딩(예: `oninput`은 `bind:value` 업데이트 후 실행)에서 발생하는 이벤트 이후에 실행됩니다. 내부적으로 일부 이벤트 핸들러는 `addEventListener`로 직접 연결되고, 다른 것들은 위임(delegation)됩니다.

`ontouchstart`와 `ontouchmove` 이벤트 속성을 사용할 때, 성능 향상을 위해 핸들러는 기본적으로 `passive`로 설정됩니다. 이렇게 하면 브라우저가 이벤트 핸들러가 `event.preventDefault()`를 호출하는지 기다리지 않고 즉시 문서를 스크롤할 수 있어서 반응성이 크게 향상됩니다. "일단 스크롤부터 하고 보자!"

아주 드물게 이러한 이벤트의 기본 동작을 막아야 하는 경우에는 (예: 액션 내부에서) `on:` 지시자를 대신 사용해야 합니다.

**이벤트 위임 (Event delegation)**
메모리 사용량을 줄이고 성능을 높이기 위해, Svelte는 이벤트 위임이라는 기술을 사용합니다. 특정 이벤트들(아래 목록 참조)에 대해서는 애플리케이션 루트에 있는 단일 이벤트 리스너가 해당 이벤트 경로에 있는 모든 핸들러를 실행하는 책임을 집니다. "대표 한 명이 다 처리한다!"

몇 가지 주의할 점이 있습니다:
*   위임된 리스너로 이벤트를 수동으로 발생시킬 때는 `{ bubbles: true }` 옵션을 설정해야 합니다. 그렇지 않으면 애플리케이션 루트까지 도달하지 못합니다.
*   `addEventListener`를 직접 사용할 때는 `stopPropagation` 호출을 피해야 합니다. 그렇지 않으면 이벤트가 애플리케이션 루트에 도달하지 못해 핸들러가 호출되지 않습니다. 마찬가지로, 애플리케이션 루트 내부에 수동으로 추가된 핸들러는 DOM 깊숙한 곳에 선언적으로 추가된 핸들러(예: `onclick={...}`)보다 캡처링 및 버블링 단계 모두에서 먼저 실행됩니다. 이런 이유로 `addEventListener`보다는 `svelte/events`에서 가져온 `on` 함수를 사용하는 것이 좋습니다. 이 함수는 순서가 유지되고 `stopPropagation`이 올바르게 처리되도록 보장합니다.

다음 이벤트 핸들러들이 위임됩니다:
`beforeinput`, `click`, `change`, `dblclick`, `contextmenu`, `focusin`, `focusout`, `input`, `keydown`, `keyup`, `mousedown`, `mousemove`, `mouseout`, `mouseover`, `mouseup`, `pointerdown`, `pointermove`, `pointerout`, `pointerover`, `pointerup`, `touchend`, `touchmove`, `touchstart`

**텍스트 표현식 (Text expressions)**
자바스크립트 표현식을 중괄호 `{}`로 감싸서 텍스트로 포함할 수 있습니다.

```svelte
{expression}
```

null이나 undefined인 표현식은 생략되고, 나머지는 모두 문자열로 변환됩니다.

Svelte 템플릿에 중괄호를 그대로 쓰고 싶으면 HTML 엔티티 문자열을 사용해야 합니다: `{`는 `&lbrace;`, `&lcub;`, 또는 `&#123;`로, `}`는 `&rbrace;`, `&rcub;`, 또는 `&#125;`로 쓰세요.

정규 표현식 리터럴 표기법을 사용한다면 괄호로 감싸야 합니다.

```svelte
<h1>안녕 {name}!</h1>
<p>{a} + {b} = {a + b}.</p>

<div>{(/^[A-Za-z ]+$/).test(value) ? x : y}</div> {/* 정규식은 괄호로! */}
```

표현식은 문자열로 변환되고 XSS 공격을 막기 위해 이스케이프 처리됩니다. 만약 HTML을 직접 렌더링하고 싶다면 `{@html}` 태그를 대신 사용하세요.

```svelte
{@html potentiallyUnsafeHtmlString}
```

XSS 공격을 막으려면 전달된 문자열을 이스케이프 처리하거나, 직접 제어할 수 있는 값으로만 채워야 합니다. "아무거나 막 넣으면 큰일 난다!"

**주석 (Comments)**
컴포넌트 안에 HTML 주석을 사용할 수 있습니다.

```svelte
<!-- 이건 주석입니다! --><h1>Hello world</h1>
```

`svelte-ignore`로 시작하는 주석은 다음 마크업 블록에 대한 경고를 비활성화합니다. 보통 접근성 경고인데, 타당한 이유가 있을 때만 비활성화하세요. "경고 무시? 너 진짜 이유 있는 거지?"

```svelte
<!-- svelte-ignore a11y_autofocus -->
<input bind:value={name} autofocus />
```

`@component`로 시작하는 특별한 주석을 추가하면 다른 파일에서 해당 컴포넌트 이름 위에 마우스를 올렸을 때 이 주석 내용이 표시됩니다. "이 컴포넌트는 말이야..." 하고 설명 달아주는 거죠.

```svelte
<!--
@component
- 마크다운 사용 가능.
- 코드 블록도 사용 가능.
- 사용 예시:
  ```html
  <Main name="Arethra">
  ```
-->
<script>
	let { name } = $props();
</script>

<main>
	<h1>
		안녕, {name}
	</h1>
</main>
```

---

**얘 뭐 하는 애냐? (기능 및 목적)**
Svelte 컴포넌트 안에서 HTML 비슷한 문법으로 화면 구조를 만드는 방법을 설명하는 내용입니다. 그냥 HTML만 쓰는 게 아니라, 자바스크립트 변수나 표현식을 중간중간 끼워 넣어 동적인 화면을 만들고, 다른 Svelte 컴포넌트도 쉽게 가져다 쓸 수 있게 해줍니다. 이벤트 처리나 주석 같은 부가 기능도 있고요. 한마디로 "Svelte로 화면 그리는 기본 도구 사용법"입니다.

**왜 쓰는데?**
1.  **직관적인 마크업**: HTML 아는 사람이면 금방 익숙해질 수 있는 문법. "어? 이거 완전 HTML인데?" 싶을 정도로.
2.  **데이터 바인딩 용이**: 자바스크립트 변수를 `{}`로 감싸기만 하면 화면에 바로 표시. 데이터 바뀌면 화면도 알아서 업데이트. "데이터랑 화면이랑 찰떡궁합!"
3.  **컴포넌트 재사용**: 대문자로 시작하는 태그로 다른 컴포넌트를 쉽게 삽입. "레고 블록처럼 조립!"
4.  **간결한 문법**: 속성 이름과 값이 같으면 `{name}`처럼 줄여 쓰고, 여러 속성을 `{...obj}`로 한 번에 넘기는 등 코드량을 줄여주는 기능 제공. "타이핑 줄여서 칼퇴하자."

**언제 불려 나오냐?**
Svelte 컴포넌트 파일(`.svelte`) 안에서 `<template>` (Svelte 5 이전) 또는 파일 최상단 (Svelte 5 이후)에 HTML과 유사한 형태로 화면 구조를 짤 때 항상 이 규칙들이 적용됩니다. Svelte로 뭔가 화면에 보여주려면 무조건 이 문법을 써야 합니다.

**쓸 때 꿀팁 및 주의사항:**
*   **태그 대소문자 구분**: 소문자는 HTML 태그, 대문자는 Svelte 컴포넌트. 헷갈리면 "이게 왜 안돼?" 시전.
*   **속성 값의 자바스크립트**: `{}` 안에는 거의 모든 자바스크립트 표현식이 들어갈 수 있지만, 너무 복잡하면 가독성 떨어지니 주의. "코드는 나만 보는 게 아니라고."
*   **`{@html}`은 신중하게**: 외부에서 받은 HTML 문자열을 그대로 렌더링하면 XSS 공격에 취약해질 수 있습니다. 정말 믿을 수 있는 내용이거나, 직접 살균 소독(sanitize)한 경우에만 사용. "믿는 도끼에 발등 찍힌다."
*   **이벤트 핸들러 `on:`**: `onclick` 같은 기본 DOM 이벤트 외에 커스텀 이벤트나 좀 더 세밀한 제어가 필요하면 `on:이벤트이름` 형태로 사용.
*   **주석 활용**: 코드 설명이나 임시로 코드 제외할 때 HTML 주석(`<!-- -->`)을 쓰고, 특정 경고를 무시할 땐 `<!-- svelte-ignore 경고종류 -->`를, 컴포넌트 설명을 달고 싶을 땐 `<!-- @component ... -->`를 활용. "주석 잘 다는 것도 실력."
*   **Svelte 버전별 차이**: Svelte 6부터는 따옴표로 감싼 단일 표현식 속성 값이 문자열로 강제 변환될 예정이니, 지금부터라도 불필요한 따옴표는 빼는 습관을 들이는 게 좋습니다. "미리미리 대비하자."