`handleError`

얘는 SvelteKit 앱에서 예상치 못한 에러가 터졌을 때 호출되는 함수입니다. 로딩 중이든, 화면 그리는 중이든, 아니면 서버 API 엔드포인트에서든, 뭔가 잘못돼서 에러가 홱 던져지면 이놈이 "어이쿠! 문제 생겼네!" 하면서 등장합니다. 이 함수 덕분에 두 가지를 할 수 있어요:

1.  에러 로그 남기기: 어디서 무슨 에러가 났는지 기록해둘 수 있죠. 나중에 문제 해결할 때 "범인은 이 안에 있어!" 하고 찾아내기 좋습니다.
2.  사용자 친화적인 에러 화면 보여주기: 에러 메시지나 스택 트레이스 같은 민감한 정보는 싹 빼고, 사용자에게 보여줘도 안전한 형태로 에러 정보를 가공해서 보여줄 수 있습니다. 여기서 리턴하는 값은 `$page.error`라는 Svelte 스토어에 저장돼서, 에러 페이지 컴포넌트(`+error.svelte`)에서 써먹을 수 있습니다. 기본값은 `{ message }`인데, "Internal Error" 같은 뜬구름 잡는 메시지일 가능성이 높죠.

여러분이 짠 코드나 라이브러리 코드에서 에러가 나면, 보통 상태 코드는 500 (서버 내부 오류)이고 메시지는 "Internal Error"로 고정됩니다. `error.message`에는 진짜 에러 내용이 담겨있지만, 이걸 그대로 사용자에게 보여주면 "해커님, 여기 취약점 있어요!" 하고 광고하는 꼴이 될 수 있으니 조심해야 합니다. `message` 프로퍼티는 상대적으로 안전하지만, 일반 사용자에겐 "이게 뭔 소리야?" 싶을 겁니다.

`$page.error` 객체에 좀 더 쓸모 있는 정보를 타입 안전하게 추가하고 싶다면, `App.Error` 인터페이스를 선언해서 원하는 모양으로 커스터마이징할 수 있습니다. (단, `message: string`은 기본으로 깔고 가야 합니다. 그래야 최소한의 메시지라도 보여주니까요.) 예를 들어, 사용자가 고객센터에 문의할 때 알려줄 수 있는 추적 ID 같은 걸 추가할 수 있습니다.

**`src/app.d.ts` (타입 정의 파일)**

```typescript
declare global {
	namespace App {
		interface Error {
			message: string; // 필수!
			errorId: string; // 우리가 추가할 추적 ID
		}
	}
}

export {}; // 모듈로 인식시키기 위한 꼼수
```

**`src/hooks.server.ts` (서버 사이드 훅)**

```typescript
import * as Sentry from '@sentry/sveltekit'; // 에러 리포팅 서비스 예시
import type { HandleServerError } from '@sveltejs/kit';

Sentry.init({/* 센트리 설정값... */});

export const handleError: HandleServerError = async ({ error, event, status, message }) => {
	const errorId = crypto.randomUUID(); // 고유한 에러 ID 생성

	// 센트리 같은 서비스에 에러 정보 보내기
	Sentry.captureException(error, {
		extra: { event, errorId, status, messageFromHook: message } // message도 같이 보내주면 좋음
	});

	return {
		message: '앗! 뭔가 문제가 생겼어요. 😥', // 사용자에게 보여줄 메시지
		errorId // 추적 ID도 함께 전달
	};
};
```

**`src/hooks.client.ts` (클라이언트 사이드 훅)**

```typescript
import * as Sentry from '@sentry/sveltekit';
import type { HandleClientError } from '@sveltejs/kit';

Sentry.init({/* 센트리 설정값... */});

export const handleError: HandleClientError = async ({ error, event, status, message }) => {
	const errorId = crypto.randomUUID();

	Sentry.captureException(error, {
		extra: { event, errorId, status, messageFromHook: message }
	});

	return {
		message: '앗! 클라이언트에서 문제가... 😭',
		errorId
	};
};
```

`src/hooks.client.js`에서는 `handleError` 타입이 `HandleClientError`이고, `event` 객체도 `RequestEvent`가 아니라 `NavigationEvent`라는 점이 다릅니다. 하는 일은 비슷해요.

참고로, `@sveltejs/kit`에서 `error` 함수를 임포트해서 의도적으로 던진 에러(예: `throw error(404, 'Not found')`)에 대해서는 이 `handleError` 함수가 호출되지 않습니다. 걔네들은 "예상된" 에러라서 SvelteKit이 알아서 잘 처리해줍니다.

개발 중일 때, Svelte 코드 문법 오류 때문에 에러가 나면, 전달되는 `error` 객체에 `frame`이라는 속성이 추가로 붙어서 어디서 에러가 났는지 코드 조각으로 보여주기도 합니다. 디버깅할 때 꽤 유용하죠.

**가장 중요한 거! `handleError` 함수 안에서는 절대 또 다른 에러가 터지면 안 됩니다.** 안 그러면 "에러 처리하다 에러 나서 망했어요"라는 무한 루프에 빠질 수 있습니다.

---

**얘 뭐 하는 애냐?**

`handleError`는 SvelteKit 애플리케이션의 "비상대책위원장" 같은 놈입니다. 앱 어딘가에서 예상치 못한 에러가 빵 터지면, 이놈이 출동해서 상황을 수습합니다. 에러 내용을 기록(로깅)하고, 사용자에게는 "죄송합니다, 지금 서비스가 원활하지 않습니다. 이 번호로 문의해주세요: XXX-XXX" 같이 정제된 안내 메시지를 보여주는 역할을 하죠. 한마디로, 앱이 에러 때문에 완전히 뻗어버리는 걸 막고, 최소한의 대응이라도 할 수 있게 해주는 안전장치입니다.

**왜 쓰는데?**

1.  **에러 추적 및 분석:** 어디서, 왜, 어떤 에러가 발생했는지 Sentry 같은 외부 서비스에 기록하거나 자체 로그 시스템에 남겨서, 개발자가 나중에 "아, 이때 이런 버그가 있었군!" 하고 파악하고 수정할 수 있게 합니다. "CCTV 설치해서 범인 잡는 격."
2.  **사용자 경험 보호:** 민감한 시스템 내부 정보(에러 메시지, 스택 트레이스 등)가 사용자에게 그대로 노출되는 걸 막아줍니다. 대신 "지금 문제가 생겼으니 잠시 후 다시 시도해주세요" 같은 부드러운 메시지나, 문의할 때 필요한 "에러 ID" 같은 걸 보여줘서 사용자가 덜 당황하게 만듭니다. "고객님, 놀라셨죠? 저희가 해결 중입니다."
3.  **일관된 에러 처리:** 서버 사이드, 클라이언트 사이드 양쪽에서 발생하는 예상치 못한 에러들을 한 곳에서 관리하고 동일한 방식으로 대응할 수 있게 해줍니다. "에러 처리는 이놈에게 맡겨!"

**언제 불려 나오냐?**

*   SvelteKit 앱의 로드 함수(`load`), 폼 액션(`actions`), 서버 API 엔드포인트 핸들러, 또는 렌더링 과정에서 `throw new Error(...)`처럼 예상치 못하게 에러가 던져졌을 때 호출됩니다.
*   `@sveltejs/kit`에서 제공하는 `error()` 헬퍼 함수로 던진 에러(예: `throw error(404, 'Page not found')`)는 "예상된" 에러로 취급되어 `handleError`를 거치지 않고, SvelteKit이 기본으로 제공하는 에러 페이지(`+error.svelte`)로 바로 연결됩니다. "이건 내가 예상했던 시나리오야!"

**쓸 때 꿀팁 및 주의사항:**

*   **서버용, 클라이언트용 따로:** `src/hooks.server.ts` (또는 `.js`) 파일에는 `HandleServerError` 타입으로, `src/hooks.client.ts` (또는 `.js`) 파일에는 `HandleClientError` 타입으로 각각 만들어야 합니다. 하는 일은 비슷하지만, 받는 `event` 객체 타입 등이 살짝 다릅니다. "각개전투!"
*   **`App.Error` 인터페이스 커스텀:** `src/app.d.ts` 파일에 `App.Error` 인터페이스를 정의하면 `$page.error` 객체의 타입을 확장해서 에러 ID, 사용자 친화적 메시지 외에 추가 정보를 담을 수 있습니다. 단, `message: string`은 무조건 포함해야 합니다. "기본 옵션은 필수입니다, 고객님."
*   **민감 정보는 로그로만:** `error.message`나 `error.stack` 같은 상세 정보는 사용자에게 절대 보여주지 말고, Sentry 같은 로깅 서비스에만 보내세요. 사용자에게는 `return { message: '문제가 발생했습니다.', errorId: '...' };` 같이 안전하게 가공된 정보만 넘겨야 합니다. "개인정보 유출은 안돼요!"
*   **`handleError` 자체는 절대 실패 금지:** 이 함수 안에서 또 에러가 나면 앱은 그냥 죽어버립니다. `try...catch`를 쓰거나, 최대한 단순하고 안정적인 코드만 작성해야 합니다. "최후의 보루가 무너지면 끝장이야."
*   **비동기 처리 가능:** `async/await`을 사용할 수 있어서, 에러 정보를 외부 서비스로 보내는 등의 비동기 작업을 처리하고 그 결과를 바탕으로 사용자에게 보여줄 내용을 결정할 수 있습니다.
*   **개발 모드 `error.frame` 활용:** 개발 중 Svelte 컴포넌트 문법 오류 등으로 에러가 나면 `error.frame`에 오류 위치 정보가 담겨 옵니다. 이걸 콘솔에 찍어보면 디버깅에 도움이 됩니다. "범인은 바로 너!" (개발 모드에서만)