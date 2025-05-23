`<svelte:window>`
`<svelte:window on:event={handler} />`
`<svelte:window bind:prop={value} />`

`<svelte:window>` 태그는 `window` 객체에 이벤트 리스너를 달 때 쓰는 놈입니다. 컴포넌트 없어질 때 리스너 제거하는 거 신경 안 써도 되고, 서버 사이드 렌더링 할 때 `window` 객체 있는지 없는지 확인할 필요도 없게 해주죠. 개꿀.

이 태그는 컴포넌트 최상단에만 올 수 있습니다. 블록이나 다른 태그 안에 쑤셔 넣으면 안 돼요. "나대지 말고 꼭대기에만 있어라."

```svelte
<script>
	function handleKeydown(event) {
		alert(`${event.key} 키 눌렀네`);
	}
</script>

<svelte:window on:keydown={handleKeydown} />
```

아래 속성들에도 값을 바인딩할 수 있습니다:

*   `innerWidth`
*   `innerHeight`
*   `outerWidth`
*   `outerHeight`
*   `scrollX`
*   `scrollY`
*   `online` — `window.navigator.onLine`의 별명. "인터넷 되냐 안 되냐"
*   `devicePixelRatio`

`scrollX`랑 `scrollY` 빼고 나머지는 읽기 전용입니다. "눈으로만 보세요."

```svelte
<svelte:window bind:scrollY={y} />
```

참고로, 접근성 문제 때문에 페이지가 초기값으로 스크롤되지는 않습니다. `scrollX`나 `scrollY`에 바인딩된 변수가 그 이후에 변경될 때만 스크롤이 일어납니다. 컴포넌트 렌더링될 때 꼭 스크롤해야 할 합당한 이유가 있다면, `$effect` 안에서 `scrollTo()`를 호출하세요. "함부로 스크롤 땡기지 마라, 사용자 놀란다."

---

**얘 뭐 하는 애냐? (기능 및 목적)**
`<svelte:window>`는 Svelte 컴포넌트 안에서 브라우저 창(`window` 객체)의 이벤트(마우스 클릭, 키보드 입력, 스크롤 등)를 감지하거나, 창의 크기나 스크롤 위치 같은 속성값을 가져오거나 변경할 때 쓰는 특수 태그입니다. 복잡한 `addEventListener`, `removeEventListener` 관리를 Svelte가 알아서 해주니까 개발자는 핵심 로직에만 집중할 수 있죠. "창문 단속은 얘가 전문가."

**왜 쓰는데?**
1.  **간편한 이벤트 처리**: `window.addEventListener('scroll', ...)` 쓰고, 컴포넌트 파괴될 때 `window.removeEventListener('scroll', ...)` 해주는 귀찮은 일을 대신 해줍니다. "붙였다 뗐다, 이제 그만."
2.  **서버 사이드 렌더링(SSR) 안전**: 서버에는 `window` 객체가 없어서 그냥 `window` 쓰면 에러납니다. `<svelte:window>`는 Svelte가 알아서 처리해주니 SSR 환경에서도 안심. "서버에서는 얌전히 있는다."
3.  **반응적인 창 속성 바인딩**: 창 크기(`innerWidth`, `innerHeight`)나 스크롤 위치(`scrollX`, `scrollY`)가 바뀔 때마다 Svelte 변수에 자동으로 업데이트하거나, 반대로 Svelte 변수 값으로 스크롤 위치를 변경할 수 있습니다. (단, `scrollX`, `scrollY` 외에는 읽기 전용) "창문 상태 실시간 중계."

**언제 불려 나오냐?**
*   사용자가 창 크기를 조절할 때 뭔가 하고 싶을 때 (예: 반응형 레이아웃 변경)
*   키보드 단축키를 전역적으로 감지하고 싶을 때
*   페이지 스크롤 위치에 따라 특정 애니메이션이나 동작을 실행하고 싶을 때
*   인터넷 연결 상태(`online`)를 확인하고 싶을 때

컴포넌트의 `<script>` 태그 바깥, 최상위 레벨에 선언해서 사용합니다.

**쓸 때 꿀팁 및 주의사항:**
*   **위치 고정! 최상단!**: `<svelte:window>`는 컴포넌트 코드의 제일 바깥쪽에만 써야 합니다. `{#if ...}` 블록 안이나 다른 HTML 태그 안에 넣으면 에러납니다. "나대는 애들은 꼭대기로 보내버려."
*   **이벤트 핸들러는 깔끔하게**: `on:eventname={handlerFunction}` 형태로 이벤트 핸들러를 연결합니다.
*   **양방향 바인딩은 `bind:`**: `bind:scrollX={variable}`처럼 `bind:` 디렉티브를 쓰면 `window`의 속성값과 Svelte 변수를 양방향으로 묶을 수 있습니다. (물론 읽기 전용 속성은 단방향)
*   **초기 스크롤은 수동으로**: `bind:scrollX`나 `bind:scrollY`를 써도 컴포넌트가 처음 딱! 나타날 때 그 값으로 스크롤되지 않습니다. 사용자가 갑자기 엉뚱한 곳으로 끌려가는 걸 막기 위해서죠. 꼭 처음부터 특정 위치로 스크롤해야 한다면 `$effect` 안에서 `window.scrollTo()`를 직접 호출해야 합니다. "사용자 배려는 기본이지."
*   **남용은 금물**: 모든 컴포넌트가 다 `<svelte:window>`를 덕지덕지 붙이고 있으면 뭐가 어디서 돌아가는지 헷갈리고 성능에도 좋을 리 없습니다. 정말 전역적인 창 이벤트나 속성이 필요할 때만 쓰세요. "아무 데나 창문 내면 집 무너진다."