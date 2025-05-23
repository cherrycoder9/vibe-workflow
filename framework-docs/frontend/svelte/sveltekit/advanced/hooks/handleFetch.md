`handleFetch`

이 함수는 서버에서 돌아가거나, 페이지 미리 만들 때(`prerendering`), 엔드포인트나 `load` 함수, `action`, `handle`, `handleError`, `reroute` 같은 놈들 안에서 `event.fetch`로 뭔가 가져오려고 할 때, 그 결과를 중간에 가로채서 수정하거나 아예 다른 걸로 바꿔치기할 수 있게 해줍니다. "야, 잠깐! 그거 내가 좀 손 좀 보고 보내자." 뭐 이런 느낌이죠.

예를 들어, 사용자가 페이지 이동할 때 `load` 함수가 `https://api.yourapp.com/` 같은 공개된 주소로 데이터를 요청한다고 칩시다. 근데 서버에서 미리 페이지를 만들 때는 굳이 인터넷망 타고 뺑뺑이 돌 필요 없이, 내부망으로 API 서버(`http://localhost:9999/`)에 바로 다이렉트로 쏘는 게 훨씬 빠르겠죠? 이럴 때 써먹는 겁니다.

`src/hooks.server.js` 에 이렇게 짜면 돼요:
```javascript
import type { HandleFetch } from '@sveltejs/kit';

export const handleFetch: HandleFetch = async ({ request, fetch }) => {
	// 만약 요청하는 주소가 'https://api.yourapp.com/'으로 시작하면,
	if (request.url.startsWith('https://api.yourapp.com/')) {
		// 원래 요청 정보는 그대로 두고, URL만 내부망 주소로 싹 갈아끼운 새 요청을 만듦
		request = new Request(
			request.url.replace('https://api.yourapp.com/', 'http://localhost:9999/'),
			request // 기존 요청의 메서드, 헤더, 본문 등은 그대로 가져감
		);
	}

	// 이렇게 손 본 (또는 안 본) 요청으로 진짜 fetch를 실행하고 그 결과를 돌려줌
	return fetch(request);
};
```

`event.fetch`로 보내는 요청은 기본적으로 브라우저가 쿠키나 인증 헤더 보내는 방식을 따라갑니다.
*   **같은 사이트 요청(same-origin):** `credentials` 옵션을 `"omit"`(안 보냄)으로 일부러 막지 않는 이상, 쿠키랑 인증 헤더는 알아서 같이 딸려갑니다.
*   **다른 사이트 요청(cross-origin):** 요청하는 주소가 내 앱의 하위 도메인(subdomain)이면 쿠키가 같이 갑니다. 예를 들어, 내 앱이 `my-domain.com`이고 API 서버가 `api.my-domain.com`이면 쿠키는 문제없이 전달돼요.

**근데 여기서 골 때리는 예외가 하나 있습니다:** 만약 내 앱(`www.my-domain.com`)이랑 API 서버(`api.my-domain.com`)가 형제뻘 하위 도메인이면, 얘네들의 공통 부모 도메인(`my-domain.com`)에 속한 쿠키는 전달이 안 됩니다. 왜냐? SvelteKit 입장에서는 "이 쿠키가 대체 뉘 집 자식이여?" 하고 헷갈려서 못 보내는 거죠. 이럴 땐 `handleFetch`로 직접 쿠키를 챙겨줘야 합니다:

`src/hooks.server.js` 에 이렇게 추가하면 됩니다:
```javascript
import type { HandleFetch } from '@sveltejs/kit';
export const handleFetch: HandleFetch = async ({ event, request, fetch }) => {
	// API 서버 주소가 'https://api.my-domain.com/'으로 시작하면,
	if (request.url.startsWith('https://api.my-domain.com/')) {
		// 브라우저가 보낸 원래 요청(event.request)에서 쿠키를 꺼내서,
		// API 서버로 가는 요청(request) 헤더에 직접 박아버림
		request.headers.set('cookie', event.request.headers.get('cookie'));
	}

	return fetch(request); // 쿠키 장착 완료! 이제 진짜 요청 보냄
};
```

---

**얘 뭐 하는 애냐?**

`handleFetch`는 SvelteKit 앱이 서버단에서 (또는 빌드 시점에) `event.fetch`를 써서 외부 데이터를 가져오려고 할 때, 그 요청을 중간에 "검문"하는 역할입니다. 요청 URL을 바꾸거나, 헤더를 추가/수정하거나, 심지어 요청 자체를 다른 걸로 바꿔치기할 수도 있죠. 한마디로 `fetch` 요청의 "중간 관리자" 또는 "교통정리 요원" 같은 놈입니다.

**왜 쓰는데?**

1.  **서버 환경 맞춤형 요청 경로 설정:** 클라이언트에서는 외부 API(`https://api.example.com`)를 부르지만, 서버에서는 내부망(`http://localhost:1234`)으로 바로 쏘는 게 효율적일 때 URL을 바꿔줍니다. "서버에서는 이쪽이 더 빠르니까 일루 가!"
2.  **요청 헤더 일괄 처리:** 모든 `event.fetch` 요청에 특정 인증 토큰을 자동으로 붙여주거나, 위 예시처럼 까다로운 형제 도메인 간 쿠키 문제를 해결할 때 씁니다. "이거 헤더에 꼭 챙겨가, 안 그럼 문전박대당한다!"
3.  **요청/응답 조작 및 모킹:** 특정 조건에서는 실제 요청 대신 미리 준비된 가짜 응답(mock)을 반환하게 만들거나, 요청 내용을 살짝 비틀어서 보낼 수도 있습니다. 테스트할 때 아주 유용하죠. "오늘은 저 API 서버 파업이니까, 내가 만든 가짜 데이터라도 먹어라."

**언제 불려 나오냐?**

SvelteKit의 서버 사이드 로직(`load` 함수, `+server.js`의 API 라우트 핸들러, `actions` 처리 함수, 전역 `handle` 및 `handleError`, `reroute` 등) 안에서 `event.fetch(...)`가 호출될 때마다, 실제 네트워크 요청이 나가기 *바로 직전에* 이 `handleFetch` 함수가 실행됩니다. 이 함수는 반드시 `src/hooks.server.js` 파일 안에 `export const handleFetch = ...` 형태로 정의되어야 SvelteKit이 알아먹습니다.

**쓸 때 꿀팁 및 주의사항:**

*   **지정된 위치 엄수:** `src/hooks.server.js`에 안 만들면 SvelteKit은 "그런 거 없는데?" 하고 무시합니다.
*   **원본 `request`는 소중히:** `request` 객체를 수정할 때는 `new Request(newUrl, originalRequest)`처럼 복제해서 쓰는 게 안전빵입니다. 원본을 직접 건드렸다간 나중에 어디서 터질지 몰라요. "원본은 보존하고, 사본으로 작업하자, 응?"
*   **`return fetch(request)`는 생명줄:** `handleFetch` 함수의 마지막에는 반드시 `fetch(request)`를 호출하고 그 결과를 `return` 해줘야 합니다. 안 그러면 요청이 허공으로 사라지는 마술을 보게 될 겁니다.
*   **쿠키, 너란 녀석...:** 같은 출처나 하위 도메인까지는 SvelteKit이 쿠키를 잘 챙겨주지만, 형제 도메인(예: `www.example.com`과 `api.example.com`이 `example.com` 쿠키를 공유해야 할 때)은 `handleFetch`로 직접 챙겨줘야 합니다. "쿠키 때문에 머리 아프면 `handleFetch`부터 뒤져봐."
*   **보안은 아무리 강조해도 지나치지 않음:** `handleFetch`는 서버에서 돌아가니까 API 키 같은 민감 정보를 다루기 상대적으로 안전하지만, 실수로 클라이언트에 노출될 만한 코드를 짜면 바로 보안 사고 터집니다. "서버 사이드라고 방심하면 골로 간다!"
*   **디버깅은 `console.log`와 함께:** 요청이 생각대로 안 바뀌면 `console.log(request.url, request.headers.get('cookie'))` 등으로 중간값을 찍어보면서 뭐가 문제인지 잡아가세요. "안되면... 찍어봐야지 뭐."