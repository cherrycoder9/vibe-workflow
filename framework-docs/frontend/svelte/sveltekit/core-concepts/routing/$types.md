위 예제들 쭉 보다 보면 `$types.d.ts` 파일에서 뭔가 계속 가져오는 걸 봤을 거야. 이건 SvelteKit이 네가 타입스크립트(혹은 JSDoc 주석 쓰는 자바스크립트)를 쓸 경우, 루트 파일들 다룰 때 타입 안전성을 확보해주려고 숨겨진 디렉토리에 자동으로 만들어주는 파일이야. "너의 코드는 소중하니까, 타입 에러는 내가 막아줄게!" 뭐 이런 거지.

예를 들어, `let { data } = $props()` 코드에 `PageProps` (혹은 `+layout.svelte` 파일이라면 `LayoutProps`)로 타입을 딱 명시해주면, 타입스크립트는 `data`의 타입이 `load` 함수에서 반환된 바로 그놈이라는 걸 알게 돼:

```typescript
// src/routes/blog/[slug]/+page.svelte

<script lang="ts">
	import type { PageProps } from './$types';

	let { data }: PageProps = $props();
</script>
```

SvelteKit 2.16.0 버전에 추가된 `PageProps`랑 `LayoutProps` 타입은 `data` 프롭을 `PageData`나 `LayoutData`로, 그리고 페이지용 `form`이나 레이아웃용 `children` 같은 다른 프롭들도 한 방에 타입 지정해주는 단축키 같은 거야. 이전 버전에서는 이런 프롭들 타입을 일일이 수동으로 적어줘야 했지. 예를 들어 페이지라면:

```typescript
// +page.svelte

import type { PageData, ActionData } from './$types';

let { data, form }: { data: PageData, form: ActionData } = $props();
```

레이아웃이라면:

```typescript
// +layout.svelte

import type { LayoutData } from './$types';
import type { Snippet } from 'svelte'; // Snippet 타입은 svelte에서 가져와야 함

let { data, children }: { data: LayoutData, children: Snippet } = $props();
```

마찬가지로, `load` 함수에 `PageLoad`, `PageServerLoad`, `LayoutLoad`, `LayoutServerLoad` (각각 `+page.js`, `+page.server.js`, `+layout.js`, `+layout.server.js`용) 같은 걸로 타입을 지정해주면, `params`나 반환값 타입이 정확하게 체크돼.

만약 VS Code나 LSP(Language Server Protocol)랑 타입스크립트 플러그인 지원하는 IDE 쓰고 있다면, 이런 타입들 아예 생략해도 돼! 스벨트 IDE 도구가 알아서 정확한 타입을 끼워 넣어주니까, 직접 안 써도 타입 체크 다 된다고. `svelte-check`라는 커맨드 라인 도구에서도 똑같이 작동해.

`$types` 생략하는 거에 대한 자세한 내용은 관련 블로그 글에서 더 읽어볼 수 있어.

---

**얘 뭐 하는 애냐?**
`$types`는 SvelteKit 프로젝트에서 타입스크립트 쓸 때, 코드 자동완성이나 타입 오류 검사 같은 개발 편의 기능을 제공하는 마법의 파일이야. SvelteKit이 네 코드 구조를 파악해서 "이 자리엔 이런 타입의 데이터가 와야 해!" 하고 알려주는 거지. 일종의 개발자용 네비게이션 겸 비서랄까.

**왜 쓰는데?**
1.  **타입 안전성 확보**: "어? 여기 숫자 들어가야 하는데 왜 문자열이 왔지?" 같은 어이없는 실수를 컴파일 전에 미리 잡아줘서 버그를 줄여줘. "런타임 에러? 그게 뭐죠? 먹는 건가요?"
2.  **개발 생산성 향상**: 코드 자동완성 기능으로 오타도 줄이고, 어떤 데이터가 넘어오는지 일일이 기억할 필요 없이 바로바로 확인 가능해. "개발 시간 단축! 칼퇴 보장!"
3.  **협업 용이성 증대**: 여러 명이 같이 작업할 때, 데이터 구조에 대한 오해를 줄여줘서 소통 비용을 아낄 수 있어. "이거 타입 뭐였더라... 아, `$types` 보면 되지!"

**언제 불려 나오냐? (언제 생성/사용되냐?)**
SvelteKit 프로젝트에서 타입스크립트 설정하고(`tsconfig.json` 같은 거), `.svelte` 파일이나 `+page.js`, `+layout.js` 같은 SvelteKit 고유 파일들을 만들기 시작하면, SvelteKit이 내부적으로 `.svelte-kit/types` 같은 숨겨진 폴더에 `$types.d.ts` 파일을 자동으로 생성하고 업데이트해. 개발자는 그냥 `import type { ... } from './$types'` 형태로 가져다 쓰기만 하면 돼.

**쓸 때 꿀팁 및 주의사항:**
*   **자동 생성 파일이니 직접 수정은 금물**: `$types.d.ts`는 SvelteKit이 알아서 관리하는 파일이야. 네가 직접 건드리면 다음에 SvelteKit이 업데이트할 때 네 수정사항 다 날아간다. "만지지 마세요. 자동으로 생성됩니다."
*   **IDE 설정 확인**: VS Code 같은 IDE에서 타입스크립트 지원 기능이 제대로 켜져 있어야 `$types`의 축복을 온전히 누릴 수 있어. Svelte 확장 프로그램 설치는 기본 중의 기본.
*   **`./$types` 경로의 의미**: 이 상대 경로는 SvelteKit이 알아서 실제 숨겨진 경로로 연결해주는 마법의 경로야. 그냥 약속이니까 "왜 하필 `./$types`임?" 하고 너무 깊게 파고들진 말자.
*   **타입 생략 기능 활용**: 최신 Svelte IDE 도구를 쓰면 `PageProps` 같은 거 굳이 `import` 안 해도 알아서 타입 추론해준다니, 얼마나 편해? 하지만 팀 컨벤션이나 가독성을 위해 명시적으로 적는 걸 선호할 수도 있으니 상황 봐서 결정하자. "써도 그만, 안 써도 그만이지만 알면 더 편하다."
*   **`$types`가 안 만들어지거나 업데이트가 안 될 때**: 가끔 SvelteKit 개발 서버를 껐다 켜거나, `.svelte-kit` 폴더를 지우고 다시 빌드/실행하면 해결될 때가 있어. "컴퓨터가 이상하면 일단 껐다 켜라."는 국룰은 여기서도 통한다.