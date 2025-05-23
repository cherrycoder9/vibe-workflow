Svelte에서 HTML 요소에 클래스를 박는 방법은 두 가지입니다: `class` 속성 쓰는 거랑 `class:` 디렉티브 쓰는 거.

**속성 (Attributes)**

원시값(숫자, 문자열, 불리언 등)은 다른 속성처럼 취급됩니다:

```svelte
<div class={large ? 'large' : 'small'}>...</div>
```

역사적인 이유로, `false`나 `NaN` 같은 falsy 값들은 문자열로 변환돼서 들어갑니다 (`class="false"` 이런 식으로). 하지만 `class={undefined}`나 `class={null}`은 아예 `class` 속성 자체를 빼버립니다. 미래 버전의 Svelte에서는 모든 falsy 값이 `class` 속성을 빼버리도록 바뀔 예정입니다. "옛날엔 그랬지~ 지금은 좀 다르지만, 앞으론 더 깔끔해질 거야!"

**객체와 배열 (Objects and arrays)**

Svelte 5.16 버전부터는 `class` 속성에 객체나 배열을 넘길 수 있고, 이건 `clsx` 라이브러리를 써서 문자열로 변환됩니다.

값이 객체면, truthy한 키들이 클래스 이름으로 추가됩니다:

```svelte
<script>
	let { cool } = $props();
</script>

<!-- `cool`이 truthy면 `class="cool"`이 되고,
	 아니면 `class="lame"`이 됨 -->
<div class={{ cool, lame: !cool }}>...</div>
```

값이 배열이면, truthy한 값들이 합쳐집니다:

```svelte
<!-- `faded`랑 `large` 둘 다 truthy면,
	 `class="saturate-0 opacity-50 scale-200"`이 됨 -->
<div class={[faded && 'saturate-0 opacity-50', large && 'scale-200']}>...</div>
```

배열이든 객체든, 조건 하나로 여러 클래스를 한 방에 넣을 수 있다는 점이 개꿀입니다. 특히 Tailwind CSS 같은 거 쓸 때 아주 유용하죠. "조건 하나에 클래스 뭉탱이! Tailwind 쓰는 사람들은 이거 못 참지."

배열 안에는 또 다른 배열이나 객체가 들어갈 수 있고, `clsx`가 알아서 평탄화(flatten)해줍니다. 로컬 클래스랑 프롭스로 받은 클래스를 합칠 때 유용합니다. 예를 들어:

**Button.svelte**

```svelte
<script lang="ts">
	let props = $props();
</script>

<button {...props} class={['cool-button', props.class]}>
	{@render props.children?.()}
</button>
```

이 컴포넌트를 쓰는 쪽에서도 똑같이 객체, 배열, 문자열을 섞어서 유연하게 쓸 수 있습니다:

**App.svelte**

```svelte
<script lang="ts">
	import Button from './Button.svelte';
	let useTailwind = $state(false);
</script>

<Button
	onclick={() => useTailwind = true}
	class={{ 'bg-blue-700 sm:w-1/2': useTailwind }}
>
	테일윈드의 불가피함을 받아들이세요
</Button>
```

Svelte는 `ClassValue`라는 타입도 제공하는데, 이건 HTML 요소의 `class` 속성이 받을 수 있는 값의 타입입니다. 컴포넌트 프롭스에서 타입 안전하게 클래스 이름을 쓰고 싶을 때 유용합니다:

```svelte
<script lang="ts">
	import type { ClassValue } from 'svelte/elements';

	const props: { class: ClassValue } = $props();
</script>

<div class={['original', props.class]}>...</div>
```

**`class:` 디렉티브 (The `class:` directive)**

Svelte 5.16 이전에는 `class:` 디렉티브가 조건부로 클래스를 설정하는 가장 편한 방법이었습니다.

```svelte
<!-- 아래 두 개는 똑같음 -->
<div class={{ cool, lame: !cool }}>...</div>
<div class:cool={cool} class:lame={!cool}>...</div>
```

다른 디렉티브처럼, 클래스 이름이랑 변수 이름이 같으면 줄여 쓸 수 있습니다:

```svelte
<div class:cool class:lame={!cool}>...</div>
```

옛날 버전 Svelte 쓰는 거 아니면, `class:` 디렉티브는 웬만하면 피하세요. 그냥 `class` 속성 쓰는 게 더 강력하고 조합하기도 좋습니다. "구관이 명관이라지만, 이건 신관이 더 낫다!"

---

**얘 뭐 하는 애냐? (기능 및 목적)**
HTML 요소에 CSS 클래스를 동적으로 추가하거나 빼는 기능을 합니다. 자바스크립트 변수 값에 따라 특정 클래스를 넣었다 뺐다 하면서 스타일을 바꾸는 거죠. "조건 따라 옷 갈아입히기"라고 생각하면 쉽습니다.

**왜 쓰는데?**
1.  **동적 스타일링**: 사용자 인터랙션(클릭, 호버 등)이나 앱 상태 변화에 따라 요소의 모양을 실시간으로 바꾸고 싶을 때 씁니다. "버튼 눌렀더니 색깔이 뿅!"
2.  **코드 가독성**: 복잡한 삼항 연산자나 if문을 HTML 템플릿 안에서 덕지덕지 쓰는 것보다 훨씬 깔끔하게 클래스를 관리할 수 있습니다. 특히 객체나 배열 문법은 여러 클래스를 조건부로 관리할 때 빛을 발합니다.
3.  **재사용성**: 컴포넌트 프롭스로 클래스 값을 넘겨서 부모 컴포넌트가 자식 컴포넌트의 스타일을 제어할 수 있게 합니다. `Button` 예제처럼요. "이 버튼은 파란색, 저 버튼은 빨간색, 다 니가 정해!"

**언제 불려 나오냐?**
Svelte 컴포넌트의 `<template>` (또는 그냥 `<script>` 바깥 HTML 부분) 안에서 HTML 태그에 클래스를 지정할 때 사용됩니다. `<script>` 안의 로직에 따라 클래스가 결정되죠.

**쓸 때 꿀팁 및 주의사항:**
*   **Svelte 5.16+ 이면 `class` 속성 쓰세요**: `class:` 디렉티브는 이제 옛날 방식입니다. `class={객체}`나 `class={배열}` 형태가 훨씬 유연하고 강력합니다. Tailwind CSS 같은 유틸리티 우선 CSS 프레임워크랑 궁합이 특히 좋습니다. "새 술은 새 부대에!"
*   **Falsy 값 처리 주의 (옛날 방식)**: `class="false"` 같은 문자열이 박히는 건 Svelte의 오랜 동작 방식이지만, 곧 바뀔 예정입니다. `undefined`나 `null`을 쓰면 속성 자체가 빠지니, 의도적으로 클래스를 빼고 싶으면 이쪽을 활용하세요.
*   **`clsx`의 마법**: 객체나 배열을 넘기면 내부적으로 `clsx`라는 유틸리티가 문자열로 예쁘게 합쳐줍니다. 중첩된 배열이나 객체도 알아서 처리해주니 복잡한 조건도 간결하게 표현 가능합니다. "복잡한 건 `clsx`에게 맡겨!"
*   **`ClassValue` 타입 활용**: 타입스크립트 쓴다면 `import type { ClassValue } from 'svelte/elements';` 해서 프롭스 타입으로 지정하세요. 안전하고 명확해집니다. "타입... 그거슨 빛..."
*   **Tailwind CSS와의 시너지**: `<div class={{ 'font-bold text-xl': isImportant, 'bg-blue-500': isActive }}>` 이런 식으로 Tailwind 클래스들을 조건부로 묶어서 관리하기 아주 편합니다. "Tailwind + Svelte 객체/배열 class = 환상의 짝꿍"
*   **`class:` 디렉티브는 과거의 유물 (하지만 알아는 두자)**: 혹시 구버전 프로젝트를 만지거나 옛날 코드 볼 일 있으면 `class:이름={조건}` 형태를 이해는 해야 합니다. `class:이름`은 `class:이름={이름}`의 축약형이고요.