페이지를 화면에 그리기 전에 미리 데이터를 불러와야 할 때가 많지. 이럴 때 쓰는 게 바로 `+page.js` 모듈이야. 이놈은 `load` 함수를 만들어서 내보내는 역할을 해:

```javascript
// src/routes/blog/[slug]/+page.js

import { error } from '@sveltejs/kit';
import type { PageLoad } from './$types';

export const load: PageLoad = ({ params }) => {
	if (params.slug === 'hello-world') {
		return {
			title: 'Hello world!',
			content: 'Welcome to our blog. Lorem ipsum dolor sit amet...'
		};
	}

	error(404, 'Not found'); // 없으면 없다고 404 에러 던져버려!
};
```

이 함수는 `+page.svelte`랑 같이 돌아가. 그러니까 서버에서 첫 페이지 그릴 때(SSR)도 실행되고, 브라우저에서 다른 페이지로 넘어갈 때(클라이언트 사이드 네비게이션)도 실행된다는 말씀. 자세한 API는 `load` 문서를 참고하라고.

`load` 함수 말고도, `+page.js`는 페이지 작동 방식을 설정하는 값들도 내보낼 수 있어:

```javascript
export const prerender = true; // 또는 false, 'auto'
export const ssr = true;       // 또는 false
export const csr = true;       // 또는 false
```

이것들에 대한 자세한 정보는 페이지 옵션에서 찾아볼 수 있다네.

---

**얘 뭐 하는 애냐?**
`+page.svelte` (화면 그리는 셔틀)가 본격적으로 일 시작하기 전에, "야, 데이터 가져와!" 하고 시키면 필요한 재료(데이터)를 미리 공수해오는 녀석이야. 화면 그릴 때 데이터 없으면 앙꼬 없는 찐빵 신세니까.

**왜 쓰는데?**
1.  **데이터 선로딩**: 페이지 뜨기 전에 데이터부터 챙겨서 빈 화면이나 에러 대신 제대로 된 내용 보여주려고. "선빵필승, 데이터도 선로딩 필승!"
2.  **SSR/CSR 양다리**: 서버에서 처음 페이지 만들 때나, 사용자가 웹에서 다른 페이지로 이동할 때나 똑같은 방식으로 데이터 가져오니 개발이 편해짐.
3.  **페이지별 맞춤 설정**: `prerender` (미리 구워놓기), `ssr` (서버가 다 해줌), `csr` (브라우저 니가 해라) 같은 옵션으로 페이지마다 렌더링 전략을 다르게 가져갈 수 있어. 유연성 갑.

**언제 불려 나오냐?**
사용자가 특정 페이지를 보려고 할 때, 그 페이지의 `+page.svelte`가 화면에 그려지기 바로 직전에 호출돼. 서버에서 첫 로딩할 때도, 브라우저에서 링크 타고 이동할 때도 마찬가지.

**쓸 때 꿀팁 및 주의사항:**
*   **`fetch`는 여기서!**: API 서버에서 데이터 땡겨오는 건 `load` 함수 안에서 SvelteKit이 제공하는 `fetch` 쓰는 게 국룰. 알아서 서버/클라 환경 맞춰줌.
*   **`params`는 꿀통**: `load` 함수 인자로 들어오는 `params`를 쓰면 `/blog/[slug]` 같은 주소에서 `slug` 값 같은 동적 파라미터 쉽게 빼먹을 수 있다.
*   **에러 처리는 `error` 헬퍼로**: 데이터 없거나 문제 생기면 `@sveltejs/kit`에서 `error(상태코드, '메시지')` 던져서 사용자한테 "님, 그거 없음ㅋ" 하고 알려주자.
*   **`parent` 데이터 재활용**: 부모 레이아웃(`+layout.js`)에서 이미 가져온 데이터는 `await parent()`로 자식인 `+page.js`에서 날로 먹을 수 있음. 중복 API 호출 막아서 서버 비용 아끼자.
*   **`load` 함수는 가볍게**: 여기서 시간 오래 끌면 페이지 로딩 속도 작살나서 사용자들 다 도망간다. 최대한 가볍고 빠르게 처리해야 함. "느리면 뭐다? 버려진다."
*   **`prerender`, `ssr`, `csr` 옵션**: 얘네 설정에 따라 `load` 함수가 언제, 어떻게 도는지 달라지니까 공식 문서 정독 필수. 특히 `prerender = true`면 빌드할 때 `load` 실행해서 HTML 미리 만들어두니까, 자주 바뀌는 데이터에는 부적합.