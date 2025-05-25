# Loading Data After Actions

액션이 실행된 후에는 (리다이렉트나 예상치 못한 에러가 발생하지 않는 한) 페이지가 다시 렌더링돼. 이때 액션의 반환값은 `form`이라는 prop으로 페이지에서 사용할 수 있게 되고. 이게 뭘 의미하냐면, 페이지의 `load` 함수들은 액션이 완료된 *후에* 실행된다는 거야. "액션 끝! 이제 `load` 출동!"

여기서 중요한 점 하나. `handle` 함수는 액션이 호출되기 *전에* 실행되고, `load` 함수들이 실행되기 전에는 다시 실행되지 않아. 이 말인즉슨, 만약 예를 들어 `handle` 함수에서 쿠키를 기반으로 `event.locals`에 값을 채워 넣는다고 할 때, 액션 내에서 해당 쿠키를 설정하거나 삭제한다면 `event.locals`도 직접 업데이트해줘야 한다는 거지. 안 그러면 `load` 함수는 옛날 `locals` 값을 쓰게 될 테니까. "내 `locals`는 내가 챙겨야 한다."

**src/hooks.server.js 예시:**
```javascript
// @ts-check
import type { Handle } from '@sveltejs/kit';

// 모든 요청 처리 전에 실행되는 'handle' 함수
export const handle: Handle = async ({ event, resolve }) => {
	// 쿠키에서 세션 ID를 가져와 사용자 정보를 event.locals에 저장
	event.locals.user = await getUser(event.cookies.get('sessionid')); // getUser는 사용자 정의 함수
	// 요청 처리 계속 진행
	return resolve(event);
};

// (실제로는 db에서 가져오거나 하겠지만, 여기선 간단히 함수로 가정)
async function getUser(sessionid: string | undefined) {
	if (sessionid === 'some-valid-session-id') { // 실제 세션 검증 로직 필요
		return { name: 'SvelteKit User', email: 'user@example.com' };
	}
	return null;
}
```

**src/routes/account/+page.server.js 예시:**
```javascript
import type { PageServerLoad, Actions } from './$types';

// 페이지 데이터 로드 함수
export const load: PageServerLoad = (event) => {
	// handle 함수에서 설정된 event.locals.user 값을 페이지 데이터로 반환
	return {
		user: event.locals.user
	};
};

// 폼 액션 정의
export const actions = {
	// 'logout' 액션
	logout: async (event) => {
		// 'sessionid' 쿠키 삭제
		event.cookies.delete('sessionid', { path: '/' });
		// event.locals.user 값도 직접 null로 업데이트! (이게 중요)
		event.locals.user = null;
	}
} satisfies Actions;
```

---

**얘 뭐 하는 애냐?**
SvelteKit에서 폼 액션(form action)이 실행된 후 데이터가 어떻게 로드되고 페이지가 어떻게 업데이트되는지, 그리고 `hooks.server.js`의 `handle` 함수와 `event.locals`를 사용할 때 주의할 점을 설명하는 거야. "액션 후 데이터 흐름과 `handle` 함수의 타이밍 문제, 그거 어떻게 돌아가는 거냐?" 에 대한 해답이지.

**왜 쓰는데? (왜 이런 흐름이고, 왜 알려주는데?)**
1.  **예측 가능한 데이터 흐름**: 액션 실행 → (필요시 `event.locals` 수동 업데이트) → `load` 함수 재실행 → 페이지 재렌더링. 이 순서를 알아야 데이터 변경 후 화면 업데이트를 제대로 제어할 수 있어.
2.  **`event.locals`의 일관성 유지**: `handle`은 요청 초기에 딱 한 번 돌고, 여기서 세팅한 `event.locals`는 액션이 뭔가 바꿔도 자동으로 갱신되지 않아. 그래서 액션에서 쿠키 같은 걸 건드려서 `locals` 내용이 바뀌어야 한다면, 개발자가 직접 `event.locals`를 업데이트해줘야 그 뒤에 실행될 `load` 함수가 최신 정보를 쓸 수 있어. "액션이 `locals` 건드렸으면, 너도 `locals` 업데이트해라. 안 그러면 `load` 함수는 옛날 값 쓴다."
3.  **서버 상태와 클라이언트 뷰 동기화**: 로그인/로그아웃 같은 액션으로 서버 쪽 상태가 바뀌면, 바로 이어지는 `load` 함수가 이 최신 상태를 반영한 데이터를 가져와서 페이지를 정확하게 보여주도록 하는 게 목적이야.

**언제 불려 나오냐? (언제 이 문제가 발생/고려되냐?)**
*   폼 액션이 성공적으로 실행되고 페이지가 다른 곳으로 리다이렉트되지 않을 때.
*   `hooks.server.js`의 `handle` 함수에서 `event.locals`에 사용자 정보 같은 것을 쿠키 기반으로 저장하고, 이 정보가 특정 액션(예: 로그인/로그아웃)에 의해 변경될 가능성이 있을 때.
*   액션 실행 후 다시 실행되는 `load` 함수에서 `event.locals`의 최신 값을 참조해야 할 때.

**쓸 때 꿀팁 및 주의사항:**
*   **`event.locals`는 수동 업데이트가 핵심**: 액션에서 쿠키를 바꾸거나 해서 `event.locals`에 담긴 정보가 더 이상 유효하지 않게 되면, 반드시! 액션 코드 안에서 `event.locals.user = null;`처럼 직접 업데이트해줘야 해. 안 그러면 `load` 함수는 유통기한 지난 데이터를 쓰게 돼.
*   **`load`는 액션 후 최신 상태 기준**: `load` 함수는 액션이 끝난 직후의 "가장 최신" 상태를 기준으로 데이터를 가져와야 해. `event.locals`를 액션에서 잘 업데이트해줘야 이게 가능해.
*   **리다이렉트하면 `load`는 스킵**: 액션에서 `redirect()`를 써서 다른 페이지로 보내버리면, 현재 페이지의 `load` 함수는 실행되지 않아. 당연하지, 다른 데로 가버리니까.
*   **`form` prop은 임시 데이터용**: 액션의 반환값은 `form` prop으로 페이지에 전달되는데, 이건 보통 "로그인 성공!" 같은 일시적인 메시지나 간단한 결과 표시에 쓰고, 페이지의 핵심 데이터는 여전히 `load` 함수를 통해 다시 로드하는 게 일반적이야.
*   **`handle`은 문지기, 한 번만**: `handle`은 모든 서버 요청의 첫 관문이고 액션보다 먼저 실행돼. 하지만 액션이 끝나고 `load`가 실행되기 *전에* `handle`이 또 실행되진 않는다는 걸 명심해야 해. "문지기는 입장할 때만 검사한다."