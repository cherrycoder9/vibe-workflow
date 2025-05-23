`<svelte:document>`

`<svelte:window>`랑 비슷하게, 이 엘리먼트는 `document` 객체에서 발생하는 이벤트를 듣거나 액션을 적용할 수 있게 해줍니다. 예를 들어 `visibilitychange` 같이 `window`에서는 안 터지는 이벤트들 말이죠. `document`에 액션을 먹일 때도 씁니다.

`<svelte:window>`처럼, 이 엘리먼트는 컴포넌트 최상단에만 놔야 하고, 절대 블록이나 다른 엘리먼트 안에 쑤셔 넣으면 안 됩니다.

```svelte
<svelte:document onvisibilitychange={handleVisibilityChange} use:someAction />
```

다음 속성들에도 값을 바인딩할 수 있습니다:

*   `activeElement` (현재 포커스된 요소)
*   `fullscreenElement` (전체 화면 모드인 요소)
*   `pointerLockElement` (마우스 포인터가 잠긴 요소)
*   `visibilityState` (페이지 표시 상태)

전부 다 읽기 전용입니다. 값을 직접 바꿀 생각은 마세요.

---

**얘 뭐 하는 애냐?**

`<svelte:document>`는 Svelte 컴포넌트 안에서 웹 브라우저의 `document` 객체에 직접 접근해서 이벤트 리스너를 걸거나, 특정 액션(Svelte의 `use:action` 지시어)을 적용하고, `document`의 이런저런 상태 값을 실시간으로 감시할 수 있게 해주는 특수 태그입니다. 웹페이지 전체, 즉 '문서' 레벨에서 벌어지는 일들을 Svelte스럽게 다루고 싶을 때 쓰는 놈이죠. "페이지 전체를 관장하는 숨은 실세" 정도로 이해하면 됩니다.

**왜 쓰는데?**

1.  **`window` 객체로는 부족할 때:** `window` 객체에서 안 터지는 `document` 고유의 이벤트들(예: `visibilitychange`, `fullscreenchange`, `selectionchange` 등)을 처리해야 할 때 씁니다. "창문(window)만 보지 말고 문서(document) 전체를 좀 보라고!"
2.  **문서 전체에 액션 걸기:** Svelte의 `use:action` 기능을 `document` 전체에 적용하고 싶을 때 씁니다. 예를 들어, 페이지 전체의 드래그 앤 드롭 동작을 커스텀한다거나 할 때 유용하죠.
3.  **문서 상태 실시간 감지:** 현재 페이지가 사용자에게 보이는지(`visibilityState`), 어떤 요소가 포커스되어 있는지(`activeElement`), 전체 화면 모드인지(`fullscreenElement`) 등을 Svelte 컴포넌트 안에서 쉽게 알고 싶을 때 씁니다. 이 값들이 바뀌면 알아서 화면도 업데이트되니 편하죠.

**언제 불려 나오냐?**

*   Svelte 컴포넌트가 화면에 딱 그려질 때(`mount`될 때), `<svelte:document>`에 설정해둔 이벤트 리스너나 액션이 `document`에 실제로 적용됩니다.
*   `document`에서 해당 이벤트(예: `visibilitychange`)가 발생하면, 지정해둔 핸들러 함수가 "나 불렀냐?" 하면서 실행됩니다.
*   바인딩해둔 속성들(`activeElement` 등)은 `document`의 실제 상태가 바뀔 때마다 Svelte의 반응성 시스템 덕분에 알아서 값이 갱신됩니다.

**쓸 때 꿀팁 및 주의사항:**

*   **위치 절대 사수!:** 이놈은 무조건 컴포넌트 코드의 맨 바깥, 최상단에 있어야 합니다. `{#if ...}` 블록이나 다른 HTML 태그 안에 넣으면 Svelte가 "이거 문법 오류인데?" 하면서 바로 쌍욕 박습니다.
*   **`svelte:window`랑 헷갈리지 마세요:** 창(window) 관련 이벤트는 `svelte:window`로, 문서(document) 관련 이벤트는 `svelte:document`로! 각자 역할이 다릅니다. "김 부장 일은 김 부장한테, 박 차장 일은 박 차장한테 시키라고!"
*   **서버 사이드 렌더링(SSR) 시에는 없는 놈 취급:** `document` 객체는 브라우저에만 있는 존재입니다. 서버에서 페이지 미리 그릴 때는 `document`가 없어서 에러 파티가 열릴 수 있습니다. `onMount` 같은 걸 써서 브라우저 환경에서만 실행되게 하거나, `if (browser)` 같은 조건문으로 감싸줘야 합니다. "서버에는 '문서'가 없다고, 이 양반아!"
*   **읽기 전용 속성은 눈으로만:** `bind:visibilityState`처럼 값을 가져올 순 있지만, 그걸로 `document.visibilityState = 'hidden'` 이런 식으로 값을 강제로 바꾸려 들면 안 됩니다. "보기만 하세요. 만지면 고장 나요."
*   **남용은 금물:** 문서 전체에 영향을 주는 기능인 만큼, 너무 많은 이벤트 리스너를 걸거나 복잡한 액션을 남용하면 페이지 성능에 안 좋을 수 있습니다. 꼭 필요할 때만, 신중하게 쓰세요. "과유불급! 뭐든 적당히 해야지."