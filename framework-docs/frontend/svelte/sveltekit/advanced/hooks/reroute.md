`reroute`

이 함수는 실제 페이지 핸들러가 실행되기 전에 돌아가면서, URL이 어떤 라우트(경로)로 해석될지 바꿔치기할 수 있게 해줍니다. 이 함수가 반환하는 경로명(기본값은 원래 URL의 경로명)을 가지고 어떤 라우트와 파라미터를 쓸지 결정하거든요.

예를 들어, `src/routes/[[lang]]/about/+page.svelte` 같은 페이지가 있다고 칩시다. 이 페이지를 `/en/about` (영어), `/de/ueber-uns` (독일어), `/fr/a-propos` (프랑스어) 같은 다양한 주소로 접근하게 하고 싶을 수 있겠죠. `reroute`를 쓰면 이렇게 구현할 수 있습니다:

`src/hooks.js` (또는 `.ts`)

```typescript
import type { Reroute } from '@sveltejs/kit';

// 번역된 URL 매핑: "사용자가 입력한 주소": "실제 SvelteKit 라우트 경로"
const translated: Record<string, string> = {
	'/en/about': '/en/about',       // 영어는 그대로
	'/de/ueber-uns': '/de/about',   // 독일어 "ueber-uns"를 내부적으로 "about"으로
	'/fr/a-propos': '/fr/about',    // 프랑스어 "a-propos"도 내부적으로 "about"으로
};

export const reroute: Reroute = ({ url }) => {
	// 만약 사용자가 입력한 주소가 우리 매핑 테이블에 있다면,
	if (url.pathname in translated) {
		// 실제 SvelteKit이 쓸 경로로 바꿔치기해서 알려줘!
		return translated[url.pathname];
	}
	// 없으면? 그냥 원래 URL 경로명 그대로 써.
};
```

이렇게 하면 `lang` 파라미터 (예: `en`, `de`, `fr`)는 바꿔치기된 경로명에서 알아서 잘 추출됩니다.

`reroute`를 써도 브라우저 주소창에 보이는 URL이나 `event.url` 값은 안 바뀝니다. 겉으로는 원래 주소 그대로인데, 내부적으로만 다른 길로 안내하는 거죠.

SvelteKit 2.18 버전부터 `reroute` 훅은 비동기 함수가 될 수 있어서, 예를 들어 백엔드에서 데이터를 가져와서 어디로 보낼지 결정하는 것도 가능해졌습니다. 다만, 이거 잘못 쓰면 페이지 이동이 느려지니까 신중하게, 그리고 빠르게 처리되도록 만들어야 합니다. 데이터 가져올 일 있으면 `reroute` 함수 인자로 넘어오는 `fetch`를 쓰세요. `load` 함수에 제공되는 `fetch`랑 같은 장점이 있는데, 아직 라우트가 결정 안 된 상태라 `handleFetch`에서 `params`나 `id`는 못 쓴다는 점만 다릅니다.

`src/hooks.js` (또는 `.ts`) - 비동기 예시

```typescript
import type { Reroute } from '@sveltejs/kit';

export const reroute: Reroute = async ({ url, fetch }) => {
	// 무한 루프 방지: 우리 앱 내부의 리라우트 API를 호출하는 경우는 제외
	if (url.pathname === '/api/reroute') return;

	// 우리 앱의 /api/reroute 엔드포인트에 물어볼 URL 만들기
	const api = new URL('/api/reroute', url.origin); // url.origin을 써서 전체 주소로 만듦
	api.searchParams.set('pathname', url.pathname); // 현재 경로를 쿼리 파라미터로 전달

	// API 호출해서 결과(JSON) 받아오기
	const result = await fetch(api).then(r => r.json());
	// API가 알려준 새 경로명 반환
	return result.pathname;
};
```

`reroute`는 순수하고 멱등적인 함수로 간주됩니다. 그래서 같은 입력에 대해서는 항상 같은 출력을 내놔야 하고, 부수 효과(side effect)가 없어야 합니다. 이런 가정 하에, SvelteKit은 클라이언트에서 `reroute` 결과를 캐싱해서 고유한 URL당 한 번만 호출되도록 합니다.

---

**얘 뭐 하는 애냐?**

`reroute`는 SvelteKit에서 사용자가 특정 URL로 접속했을 때, "야, 그 주소 사실 이쪽 길이야!" 하고 내부적으로 경로를 살짝 바꿔주는 길 안내원 같은 녀석입니다. 겉으로는 주소창 URL이 그대로인데, 안에서는 다른 라우트 파일을 보여주는 거죠. 일종의 "URL 화장술" 또는 "내부 경로 조작 마법"이라고 보면 됩니다.

**왜 쓰는데?**

1.  **다국어/지역화 URL 처리:** `/ko/소개`, `/en/about`, `/ja/紹介` 처럼 언어별로 다른 URL을 쓰지만, 실제로는 다 같은 `src/routes/[[lang]]/about/+page.svelte` 파일을 보여주고 싶을 때 씁니다. "손님, '소개'나 'about'이나 다 같은 방입니다."
2.  **옛날 URL → 새 URL 구조로 매끄럽게 전환:** 웹사이트 구조 개편으로 URL이 바뀌었지만, 옛날 주소로 들어오는 사용자도 새 콘텐츠를 보게 하고 싶을 때 (브라우저 주소창은 안 바꾸면서). "간판은 옛날 거지만, 안에는 리모델링 싹 했습니다."
3.  **마케팅용 짧은 URL 또는 예쁜 URL:** `/promo/super-deal` 같은 주소를 실제로는 `/products/123?source=super-deal` 같은 복잡한 경로로 연결하고 싶을 때.
4.  **A/B 테스팅 시 URL 분기:** 특정 조건에 따라 같은 URL이라도 다른 버전의 페이지로 내부적으로 연결할 때 (이건 좀 고급 활용).

**언제 불려 나오냐?**

사용자가 SvelteKit 앱의 어떤 URL로 접속 요청을 보내면, SvelteKit 라우터가 "어떤 페이지 파일을 보여줘야 하지?" 하고 고민하기 직전에 이 `reroute` 셔틀이 먼저 나섭니다. `src/hooks.server.js` (또는 `hooks.client.js`, 혹은 둘 다 쓰는 `hooks.js`) 파일 안에 `export const reroute = (...) => { ... }` 형태로 정의해두면 SvelteKit이 알아서 찾아 씁니다.

**쓸 때 꿀팁 및 주의사항:**

*   **주소창은 그대로, 속만 바뀐다:** `reroute`는 브라우저 주소창의 URL을 바꾸지 않습니다. 진짜 URL 리다이렉션(주소창 바뀜)을 원하면 `redirect` 함수를 써야 합니다. "겉바속촉 아니고 겉그속바 (겉은 그대로, 속만 바뀜)."
*   **캐싱 주의:** SvelteKit은 `reroute` 결과를 클라이언트에서 캐싱합니다. 즉, 한번 `/ko/소개` -> `/ko/about`으로 매핑되면, 다음에 또 `/ko/소개`로 접속해도 `reroute` 함수를 다시 실행 안 하고 캐시된 `/ko/about`을 쓴다는 거죠. 만약 동적으로 매번 다른 결과를 내야 한다면 캐싱 동작을 잘 이해하고 설계해야 합니다. "한번 알려준 길은 기억한다, 이거야."
*   **순수성과 멱등성은 국룰:** `reroute` 함수는 같은 입력 URL에 대해 항상 똑같은 출력 경로를 반환해야 하고, 외부 상태를 변경하는 등의 부수 효과가 없어야 합니다. 안 그러면 캐싱 때문에 골치 아파집니다.
*   **비동기 가능, 근데 빠르게!:** `async/await` 써서 API 요청 같은 거 할 수 있는데, 이게 느리면 페이지 로딩 전체가 느려집니다. "손님, 길 안내가 늦어서 죄송합니다... 하면 안 되겠지?" `fetch` 쓸 때는 인자로 넘어오는 SvelteKit 전용 `fetch`를 쓰세요.
*   **무한 루프 자멸각 조심:** `reroute` 설정 잘못해서 A를 B로, B를 다시 A로 돌리면... 네, 상상하는 그거 맞습니다. 브라우저 뻗습니다.
*   **영향 범위 파악:** `reroute`는 `event.url` 객체의 `pathname`만 바꿔서 라우팅 결정에 영향을 줍니다. `event.url`의 다른 부분(예: `searchParams`, `hash`)이나 브라우저 주소창 자체는 안 건드린다는 걸 명심하세요.