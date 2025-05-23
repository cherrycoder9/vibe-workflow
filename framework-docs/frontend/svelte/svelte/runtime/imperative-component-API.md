## 명령형 컴포넌트 API (Imperative component API)

모든 Svelte 애플리케이션은 명령형으로 루트 컴포넌트를 만들면서 시작합니다. 클라이언트에서는 이 컴포넌트가 특정 엘리먼트에 마운트되죠. 서버에서는 대신 렌더링할 수 있는 HTML 문자열을 받고 싶을 거고요. 다음 함수들이 바로 이런 작업을 도와줍니다.

### `mount`

컴포넌트 인스턴스를 만들고 주어진 `target`에 붙여줍니다:

```javascript
import { mount } from 'svelte';
import App from './App.svelte';

const app = mount(App, {
	target: document.querySelector('#app'), // 여기다 박아!
	props: { some: 'property' } // 이런 속성으로!
});
```

한 페이지에 여러 컴포넌트를 마운트할 수도 있고, 애플리케이션 내부에서도 마운트할 수 있습니다. 예를 들어 툴팁 컴포넌트를 만들어서 마우스 올린 엘리먼트에 붙일 때처럼요.

**주의:** Svelte 4에서 `new App(...)`을 호출하는 것과는 다르게, `mount` 중에는 `onMount` 콜백이나 액션 함수 같은 effect들이 실행되지 않습니다. 만약 (테스트 같은 상황에서) 대기 중인 effect들을 강제로 실행해야 한다면, `flushSync()`를 쓰면 됩니다.

---

**얘 뭐 하는 애냐?**

`mount`는 Svelte 컴포넌트를 웹페이지의 특정 위치에 실제로 "심는" 녀석입니다. "야, 이 컴포넌트, 저기 `#app`이라는 땅에다 심어!" 하고 명령하는 거죠. Svelte 앱이 브라우저에서 생명을 얻는 첫 단계라고 보면 됩니다.

**왜 쓰는데?**

1.  **Svelte 앱 시작:** 브라우저에서 Svelte 앱을 띄울 때 가장 먼저 이놈을 불러서 루트 컴포넌트를 DOM에 박아 넣습니다. 이게 없으면 앱 시작도 못 해요.
2.  **동적 컴포넌트 주입:** 페이지 로드 후에도 필요에 따라 특정 컴포넌트(모달, 툴팁, 알림창 등)를 찍어서 원하는 곳에 꽂아 넣을 수 있습니다. "여기 툴팁 하나 추가요!"

**언제 불려 나오냐?**

클라이언트 사이드(브라우저)에서 Svelte 애플리케이션을 초기화할 때, 또는 사용자와의 상호작용 등으로 인해 새로운 Svelte 컴포넌트를 DOM 어딘가에 동적으로 추가해야 할 때 호출됩니다.

**쓸 때 꿀팁 및 주의사항:**

*   **`target`은 필수:** 어디에 심을지 알려줘야 합니다. `document.body`도 되고, 특정 `id`를 가진 엘리먼트도 됩니다. "목표 지정 안 하면 어디다 심으라는 거냐!"
*   **Svelte 5부터 effect 실행 타이밍 변경:** Svelte 4와 달리, `mount` 함수 자체를 실행할 때는 `onMount` 같은 effect들이 바로 실행되지 않습니다. 만약 동기적으로 effect를 실행시켜야 한다면, `mount` 호출 직후에 `flushSync()`를 불러줘야 합니다. "심자마자 바로 물(effect) 주지 않는다고! `flushSync`로 강제 급수 필요!"
*   **여러 개 심기 가능:** 한 페이지에 `mount`로 여러 독립적인 Svelte 컴포넌트들을 심을 수 있습니다. 마이크로 프론트엔드처럼 부분부분 Svelte를 적용할 때 유용하죠.

### `unmount`

이전에 `mount`나 `hydrate`로 생성된 컴포넌트를 제거합니다.

만약 `options.outro`가 `true`이면, 컴포넌트가 DOM에서 제거되기 전에 트랜지션(애니메이션 효과)이 재생됩니다:

```javascript
import { mount, unmount } from 'svelte';
import App from './App.svelte';

const app = mount(App, { target: document.body });

// 나중에...
unmount(app, { outro: true }); // 퇴장 애니메이션 틀고 꺼져!
```

`options.outro`가 `true`이면 트랜지션이 완료된 후 resolve되는 Promise를 반환하고, 그렇지 않으면 즉시 resolve되는 Promise를 반환합니다.

---

**얘 뭐 하는 애냐?**

`unmount`는 `mount`로 열심히 심어놨던 컴포넌트를 "뽑아내는" 녀석입니다. "너 이제 필요 없으니 방 빼!" 하고 DOM에서 퇴출시키는 거죠.

**왜 쓰는데?**

1.  **자원 정리:** 더 이상 화면에 필요 없는 컴포넌트를 제거해서 메모리 누수를 막고 페이지를 가볍게 유지합니다. "안 쓰는 물건은 제때 버려야 집이 깨끗해지지."
2.  **동적 UI 제어:** 특정 조건에 따라 컴포넌트를 화면에서 완전히 치워버릴 때 씁니다.

**언제 불려 나오냐?**

`mount`로 생성했던 컴포넌트 인스턴스가 더 이상 필요 없어졌을 때, 해당 인스턴스를 인자로 넘겨 호출합니다. 예를 들어, 사용자가 모달 창을 닫거나, 특정 뷰에서 벗어날 때죠.

**쓸 때 꿀팁 및 주의사항:**

*   **퇴장 애니메이션은 `outro: true`:** 컴포넌트 사라질 때 스르륵~ 하고 멋지게 사라지게 하려면 이 옵션을 켜세요. Promise를 반환하니까 애니메이션 끝나는 시점도 알 수 있습니다. "갈 때 가더라도 간지나게!"
*   **`mount`된 놈만 뽑을 수 있음:** `mount`나 `hydrate`로 반환된 컴포넌트 인스턴스를 정확히 전달해야 합니다. "엉뚱한 놈 뽑으려 하지 마라."

### `render`

서버에서만, 그리고 `server` 옵션으로 컴파일할 때만 사용할 수 있습니다. 컴포넌트를 받아서 `body`와 `head` 속성을 가진 객체를 반환하며, 이걸로 앱을 서버 사이드 렌더링할 때 HTML을 채울 수 있습니다:

```javascript
import { render } from 'svelte/server'; // svelte/server에서 가져옴!
import App from './App.svelte';

const result = render(App, {
	props: { some: 'property' }
});

result.body; // <body> 태그 어딘가에 들어갈 HTML
result.head; // <head> 태그 어딘가에 들어갈 HTML
```

---

**얘 뭐 하는 애냐?**

`render`는 Svelte 컴포넌트를 서버에서 순수한 HTML "반죽"으로 찍어내는 기계입니다. 브라우저 없이, 오직 Node.js 같은 서버 환경에서만 돌아가는 특별한 녀석이죠. "클라이언트? 그게 뭐임? 난 HTML만 만든다."

**왜 쓰는데?**

1.  **서버 사이드 렌더링 (SSR):** 사용자가 페이지를 요청했을 때, 서버에서 미리 HTML을 다 만들어서 보내줍니다. 이렇게 하면 초기 로딩 속도가 빨라지고, 검색 엔진 최적화(SEO)에도 유리합니다. "손님 오시기 전에 미리 상 차려놓는 센스!"
2.  **정적 사이트 생성 (SSG)의 재료:** SSG 툴에서 각 페이지별 HTML을 만들 때 내부적으로 이놈을 사용합니다.

**언제 불려 나오냐?**

Node.js 같은 서버 환경에서, 웹 서버가 클라이언트로부터 페이지 요청을 받았을 때 호출됩니다. Svelte 컴포넌트와 초기 `props`를 주면, HTML 조각들(`head`, `body`)을 뱉어냅니다.

**쓸 때 꿀팁 및 주의사항:**

*   **출신 성분 확인:** `import { render } from 'svelte/server';` 이렇게 `svelte/server`에서 데려와야 합니다. 그냥 `svelte`에서 찾으면 "그런 애 없는데?" 소리 듣습니다.
*   **컴파일 옵션 필수:** Svelte 컴파일러 설정에 `compilerOptions: { ssr: true }` 또는 `generate: 'ssr'` (Vite 플러그인 기준)이 있어야 제대로 작동합니다. "SSR용으로 빌드 안 하면 얘는 그냥 장식품."
*   **결과물은 분리수거:** `result.head`는 HTML `<head>` 안에, `result.body`는 `<body>` 안에 잘 넣어줘야 합니다. "음식물 쓰레기랑 일반 쓰레기 분리하듯이!"

### `hydrate`

`mount`와 비슷하지만, `target` 내부에 Svelte의 SSR 출력물(render 함수로 생성된 HTML)이 있다면 그걸 재사용해서 상호작용 가능하게 만듭니다:

```javascript
import { hydrate } from 'svelte';
import App from './App.svelte';

const app = hydrate(App, {
	target: document.querySelector('#app'),
	props: { some: 'property' }
});
```

`mount`와 마찬가지로, `hydrate` 중에는 effect가 실행되지 않습니다. 필요하다면 직후에 `flushSync()`를 사용하세요.

---

**얘 뭐 하는 애냐?**

`hydrate`는 서버가 미리 만들어둔 HTML "뼈대"에 Svelte의 "생명력(자바스크립트)"을 불어넣는 작업입니다. `mount`가 맨땅에 건물 올리는 거라면, `hydrate`는 이미 지어진 건물(SSR된 HTML)에 인테리어하고 전기 연결해서 실제 사람이 살 수 있게 만드는 거죠. "마네킹에 영혼을 불어넣는 격!"

**왜 쓰는데?**

1.  **SSR의 완성:** 서버가 보내준 HTML은 그냥 껍데기입니다. `hydrate`를 통해 이 껍데기에 Svelte의 동적인 기능들(이벤트 리스너, 상태 변화 등)을 연결해서 진짜 웹 애플리케이션으로 만듭니다. "보기만 좋던 페이지가 이제 말도 하고 움직이네?"
2.  **빠른 초기 상호작용:** 사용자는 서버에서 온 HTML을 먼저 보고, 백그라운드에서 `hydrate`가 진행되어 앱이 완전해집니다.

**언제 불려 나오냐?**

서버 사이드 렌더링(SSR)으로 생성된 HTML 페이지가 클라이언트(브라우저)에 로드된 후, 해당 HTML에 Svelte의 인터랙션을 추가하기 위해 호출됩니다. 보통 클라이언트 사이드 진입점(`main.js` 같은 파일)에서 실행되죠.

**쓸 때 꿀팁 및 주의사항:**

*   **짝꿍은 `render`:** 서버에서 `render`로 HTML을 만들었다면, 클라이언트에서는 `hydrate`로 물을 줘야 합니다. `mount` 쓰면 서버가 만든 HTML 다 무시하고 새로 그리니까 SSR의 의미가 없어집니다. "세트 메뉴는 같이 시켜야 제맛!"
*   **HTML 구조 일치:** `hydrate` 할 `target` 내부의 HTML 구조는 서버에서 `render`로 만들었던 HTML 구조와 정확히 일치해야 합니다. 안 그러면 Svelte가 "어? 이거 내가 만든 거랑 다른데?" 하면서 경고 띄우거나 화면이 깨질 수 있습니다.
*   **Effect 실행 타이밍 (Svelte 5+):** `mount`와 마찬가지로, `hydrate` 자체 실행 시에는 `onMount` 같은 effect가 바로 안 터집니다. `flushSync()`로 명시적으로 호출해줘야 합니다. "물 준다고 바로 꽃 피는 거 아니라고!"
*   **SSR 안 쓰면 쳐다도 보지 마세요:** SSR 안 하는 순수 클라이언트 사이드 렌더링(CSR) 앱에서는 `hydrate` 쓸 일 없습니다. `mount` 쓰세요.