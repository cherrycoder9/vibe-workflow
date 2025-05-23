`event.locals`

요청에 커스텀 데이터를 추가해서 `+server.js` 파일의 핸들러나 서버 `load` 함수로 넘기고 싶으면, 아래처럼 `event.locals` 객체를 채우면 됩니다.

`src/hooks.server.js` (또는 `.ts`)
```typescript
import type { Handle } from '@sveltejs/kit';

export const handle: Handle = async ({ event, resolve }) => {
	// event.cookies 에서 세션 ID 가져와서 사용자 정보 때려박기
	event.locals.user = await getUserInformation(event.cookies.get('sessionid'));

	const response = await resolve(event);

	// 응답 헤더 수정이 항상 안전빵은 아님.
	// 응답 객체 중에는 헤더가 안 바뀌는 놈들도 있음 (예: 엔드포인트에서 Response.redirect() 때린 경우).
	// 안 바뀌는 헤더 건드리면 TypeError 뜨니까 조심.
	// 그럴 땐 응답 객체를 복제하거나, 헤더 안 바뀌는 응답 객체는 만들지 마셈.
	response.headers.set('x-custom-header', '감자'); // 커스텀 헤더도 슬쩍

	return response;
};
```

여러 `handle` 함수를 정의하고 `sequence` 헬퍼 함수로 순서대로 실행시킬 수도 있습니다.

---

**얘 뭐 하는 애냐?**
`event.locals`는 SvelteKit에서 들어오는 모든 서버 요청마다 딸려 있는 일종의 "비밀 주머니"입니다. `hooks.server.js`의 `handle` 함수에서 이 주머니에 원하는 데이터를 넣어두면, 그 요청이 처리되는 동안 서버 사이드의 다른 곳(주로 `load` 함수나 API 엔드포인트)에서 꺼내 쓸 수 있습니다. "요청 처리 파이프라인에 몰래 정보 쪽지 끼워넣기" 스킬이죠.

**왜 쓰는데?**
1.  **사용자 인증 정보 공유:** 로그인한 사용자 정보를 매번 DB에서 조회하는 대신, `handle`에서 한 번 조회해서 `event.locals.user`에 넣어두면 다른 `load` 함수에서 바로 써먹을 수 있습니다. "손님, 신분증 이쪽 주머니에 넣어두세요. 계속 보여주셔야 합니다."
2.  **요청별 설정값 전달:** 특정 요청에만 적용될 테마, 언어 설정, 접근 권한 같은 걸 `locals`에 담아두면 편리합니다.
3.  **공용 데이터/서비스 접근:** DB 커넥션 풀이나 자주 쓰는 서비스 객체를 `locals`에 넣어두고 재활용할 수도 있습니다. (근데 이건 보통 앱 초기화 시점에 전역적으로 관리하는 게 더 낫긴 합니다.)

**언제 불려 나오냐?**
클라이언트가 서버에 뭔가 요청할 때마다(`GET`, `POST` 등등), SvelteKit은 `src/hooks.server.js`의 `handle` 함수를 실행합니다. 바로 이 `handle` 함수 안에서 `event.locals`에 값을 할당하는 겁니다. 이렇게 저장된 값은 해당 요청이 끝나기 전까지, 즉 서버 `load` 함수나 `+server.js`의 API 핸들러가 실행될 때 `event.locals.어쩌구` 형태로 참조할 수 있습니다.

**쓸 때 꿀팁 및 주의사항:**
*   **타입 정의는 국룰:** TypeScript 쓴다면 `src/app.d.ts` 파일에 `App.Locals` 인터페이스를 확장해서 `locals`에 뭘 넣을지 미리 정의해두세요. 자동완성도 되고, 실수로 엉뚱한 거 넣는 참사를 막아줍니다. "타입스크립트 안 쓰면? 그건 코딩이 아니라 야생에서 맨손 격투하는 거랑 똑같아."
*   **민감 정보는 가공해서:** 세션 ID 자체를 `locals`에 넣고 여기저기 돌려쓰기보다는, 그 ID로 조회한 사용자 객체처럼 한 번 가공된 정보를 넣는 게 좀 더 안전합니다. `locals`는 서버에서만 돌아가지만, 그래도 보안은 아무리 강조해도 지나치지 않아요.
*   **덮어쓰기 조심:** 여러 `handle` 함수를 `sequence`로 엮어 쓸 때, 앞에 있는 `handle` 함수가 `locals`에 설정한 값을 뒤에 있는 놈이 덮어쓸 수 있습니다. 순서 잘 생각하고 짜세요.
*   **직렬화 안 됨:** `event.locals`는 서버에서만 유효하고, 클라이언트로 자동으로 전달되지 않습니다. 클라이언트에 데이터 보내려면 `load` 함수에서 리턴값으로 넘겨야 합니다. "이 주머니는 서버 밖으로 못 나간다!"

---

`resolve` 함수의 두 번째 인자

`resolve` 함수는 두 번째 선택적 파라미터를 받는데, 이걸로 응답이 어떻게 렌더링될지 더 세밀하게 제어할 수 있습니다. 이 파라미터는 객체 형태고, 다음 필드들을 가질 수 있습니다:

*   `transformPageChunk(opts: { html: string, done: boolean }): MaybePromise<string | undefined>`: HTML 조각에 커스텀 변환을 적용합니다. `done`이 `true`면 마지막 조각이라는 뜻입니다. 조각들이 항상 완전한 HTML 형태라는 보장은 없지만(예: 여는 태그만 있고 닫는 태그는 없을 수도 있음), `%sveltekit.head%`나 레이아웃/페이지 컴포넌트 같은 의미 있는 경계에서 항상 잘립니다.
*   `filterSerializedResponseHeaders(name: string, value: string): boolean`: `load` 함수가 `fetch`로 리소스를 로드할 때, 어떤 헤더를 직렬화된 응답에 포함할지 결정합니다. 기본적으로는 아무것도 포함 안 합니다.
*   `preload(input: { type: 'js' | 'css' | 'font' | 'asset', path: string }): boolean`: 어떤 파일을 `<head>` 태그에 추가해서 미리 로드할지 결정합니다. 이 메서드는 빌드 시점에 코드 조각을 만들면서 발견된 각 파일에 대해 호출됩니다. 예를 들어 `+page.svelte`에 `import './styles.css'`가 있으면, 해당 페이지 방문 시 이 CSS 파일의 최종 경로와 함께 `preload`가 호출됩니다. 개발 모드에서는 빌드 시점 분석에 의존하기 때문에 `preload`가 호출되지 않습니다. 미리 로드하면 에셋을 더 빨리 다운로드해서 성능을 높일 수 있지만, 불필요한 걸 너무 많이 받으면 오히려 해가 될 수도 있습니다. 기본적으로 `js`와 `css` 파일은 미리 로드됩니다. `asset` 파일은 현재 전혀 미리 로드되지 않지만, 피드백을 보고 나중에 추가될 수도 있습니다.

`src/hooks.server.js` (또는 `.ts`)
```typescript
import type { Handle } from '@sveltejs/kit';

export const handle: Handle = async ({ event, resolve }) => {
	const response = await resolve(event, {
		// HTML 응답 나갈 때 'old'를 'new'로 싹 바꿔버리기
		transformPageChunk: ({ html }) => html.replace('old', 'new'),
		// 'x-'로 시작하는 헤더만 fetch 응답에 포함시키기
		filterSerializedResponseHeaders: (name) => name.startsWith('x-'),
		// JS 파일이거나 경로에 '/important/'가 포함된 파일만 미리 로드하기
		preload: ({ type, path }) => type === 'js' || path.includes('/important/')
	});

	return response;
};
```

참고로 `resolve(...)`는 절대 에러를 던지지 않고, 항상 적절한 상태 코드를 가진 `Promise<Response>`를 반환합니다. 만약 `handle` 실행 중 다른 곳에서 에러가 터지면, SvelteKit은 그 에러를 치명적인 것으로 간주하고, `Accept` 헤더에 따라 에러의 JSON 표현이나 (커스텀 가능한 `src/error.html`을 통한) 폴백 에러 페이지로 응답합니다. 에러 처리에 대한 자세한 내용은 여기서 읽어볼 수 있습니다.

---

**얘 뭐 하는 애냐?**
`resolve` 함수 자체는 SvelteKit이 요청을 받아 실제 페이지 HTML이나 API 응답을 만들어내는 핵심 로직입니다. 여기에 두 번째 인자로 객체를 넘기면, SvelteKit이 응답을 생성하는 과정에 살짝 끼어들어서 최종 결과물을 입맛대로 바꾸거나 최적화할 수 있는 "고급 컨트롤 패널"을 여는 셈입니다.

**왜 쓰는데?**
1.  **`transformPageChunk` (HTML 변조 마법):** 서버에서 생성된 HTML이 브라우저로 가기 직전에 내용을 살짝 바꾸고 싶을 때 씁니다. 예를 들어, 특정 문구를 동적으로 삽입하거나, A/B 테스트를 위해 다른 내용을 보여주거나, 특정 조건에 따라 스크립트를 추가/제거할 때 유용합니다. "HTML아, 너 잠깐 일로 와봐. 성형 좀 하자."
2.  **`filterSerializedResponseHeaders` (헤더 검문소):** `load` 함수 안에서 `fetch`로 다른 API를 호출했을 때, 그 응답 헤더 중에서 특정 헤더만 골라서 클라이언트(브라우저)로 전달하고 싶을 때 씁니다. 보안상 민감하거나 불필요한 헤더는 여기서 걸러낼 수 있죠. "이 헤더는 통과! 저 헤더는 압수!"
3.  **`preload` (자원 선점 전략):** 특정 페이지를 열 때 꼭 필요한 JS, CSS, 폰트 파일들을 브라우저가 좀 더 일찍 다운로드하도록 `<link rel="preload">` 태그를 `<head>`에 자동으로 넣어줍니다. 이렇게 하면 실제 해당 자원이 필요할 때 이미 다운로드되어 있어서 페이지 로딩 속도가 빨라질 수 있습니다. "야, 이 파일들 VIP니까 미리미리 대기시켜놔!"

**언제 불려 나오냐?**
`src/hooks.server.js`의 `handle` 함수 안에서 `await resolve(event, { ...옵션들... })`을 호출하면, SvelteKit 내부적으로 응답을 만드는 과정에서 각 옵션에 해당하는 로직이 실행됩니다.
*   `transformPageChunk`: HTML이 조각(chunk) 단위로 생성될 때마다 호출됩니다.
*   `filterSerializedResponseHeaders`: `load` 함수에서 `fetch`를 사용하고 그 응답이 클라이언트로 전달될 때 호출됩니다.
*   `preload`: 페이지를 빌드할 때 분석된 에셋 목록을 기반으로, 해당 페이지 요청 시 호출됩니다. (개발 모드 제외)

**쓸 때 꿀팁 및 주의사항:**
*   **`transformPageChunk`는 조각난 HTML 대상:** 전체 HTML 문서가 통째로 오는 게 아니라, 말 그대로 '조각'입니다. 정규식 같은 걸로 뭔가 바꿀 때 태그가 중간에 잘려있을 수 있으니, "어? 왜 제대로 안 바뀌지?" 하기 전에 이 점을 명심하세요. 웬만하면 간단한 문자열 치환 정도에 쓰는 게 속 편합니다.
*   **`filterSerializedResponseHeaders`는 화이트리스트 방식:** 기본적으로 아무 헤더도 안 넘깁니다. 필요한 헤더만 `true`를 반환해서 명시적으로 허용해야 합니다. "아무나 통과시키면 보안 뚫리는 건 시간문제."
*   **`preload`는 양날의 검:** 잘 쓰면 성능 개선에 큰 도움이 되지만, 너무 많은 파일을 `preload`하거나 당장 필요 없는 것까지 걸어두면 오히려 초기 로딩에 불필요한 네트워크 요청만 늘려서 성능을 해칠 수 있습니다. "뷔페 가서 욕심부리다 배 터지는 수가 있다." 진짜 핵심적인 자원만 신중하게 선택하세요.
*   **`preload`는 빌드 타임 분석 기반:** 개발 모드에서는 `preload`가 호출되지 않으니, "왜 개발 서버에선 `preload` 안 먹히지?" 하고 삽질하지 마세요. 빌드된 결과물에서 확인해야 합니다.
*   **`resolve`는 절대 울지 않아:** `resolve` 함수 자체는 에러를 뱉지 않습니다. 내부적으로 문제가 생겨도 어떻게든 `Response` 객체를 만들어서 돌려주니, `resolve`를 `try...catch`로 감쌀 필요는 거의 없습니다. 다른 곳에서 에러 나면 SvelteKit이 알아서 에러 페이지 보여줍니다.