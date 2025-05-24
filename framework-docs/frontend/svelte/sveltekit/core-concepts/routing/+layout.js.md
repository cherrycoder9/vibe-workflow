`+page.svelte`가 `+page.js`에서 데이터 받아먹는 것처럼, 니 `+layout.svelte` 컴포넌트도 `+layout.js` 안에 있는 `load` 함수를 통해서 데이터를 공급받을 수 있어.

```javascript
// src/routes/settings/+layout.js

import type { LayoutLoad } from './$types';

export const load: LayoutLoad = () => {
	return {
		sections: [
			{ slug: 'profile', title: 'Profile' },
			{ slug: 'notifications', title: 'Notifications' }
		]
	};
};
```

만약 `+layout.js`가 페이지 옵션들(`prerender`, `ssr`, `csr`)을 export 한다면, 그건 그 아래 딸린 페이지들의 기본값으로 사용될 거야.

레이아웃의 `load` 함수가 반환한 데이터는 그 밑에 있는 모든 자식 페이지들에서도 써먹을 수 있어:

```svelte
// src/routes/settings/profile/+page.svelte

<script lang="ts">
	import type { PageProps } from './$types';

	let { data }: PageProps = $props();

	console.log(data.sections); // [{ slug: 'profile', title: 'Profile' }, ...]
</script>
```

페이지 사이를 왔다 갔다 할 때 레이아웃 데이터는 안 바뀌는 경우가 많은데, SvelteKit은 똑똑해서 필요할 때만 `load` 함수를 다시 실행시켜.

---

**얘 뭐 하는 애냐?**
`+page.js`가 특정 페이지만의 밥이라면, `+layout.js`는 그 페이지를 감싸는 레이아웃(틀)이랑 그 레이아웃을 공유하는 모든 자식 페이지들의 공용 반찬 같은 데이터를 준비하는 녀석이야. "얘들아, 공통 메뉴 나왔다! 각자 페이지에서는 메인 디쉬만 챙겨!"

**왜 쓰는데?**
1.  **데이터 중복 제거 (DRY - Don't Repeat Yourself)**: 여러 페이지에서 똑같이 필요한 데이터(예: 사용자 정보, 사이트 메뉴)를 레이아웃 `load`에서 한 번만 가져오면 하위 페이지들은 날로 먹을 수 있음. API 호출 줄이고 코드도 깔끔해짐. "한 번만 만들어서 돌려쓰자, 지구도 아끼고 개발자도 아끼고."
2.  **일관된 데이터 제공**: 특정 섹션의 모든 페이지가 동일한 기본 데이터를 공유하니까 데이터 불일치 문제 줄어듦.
3.  **페이지 옵션 기본값 설정**: 레이아웃 수준에서 `prerender`, `ssr`, `csr` 같은 옵션을 정해두면, 그 아래 페이지들은 특별히 지정 안 해도 그 설정을 따라감. "우리 구역 룰은 이거다."

**언제 불려 나오냐?**
해당 레이아웃을 사용하는 페이지에 접근할 때, 그리고 그 레이아웃이 화면에 그려지기 전에 호출돼. 자식 페이지로 이동할 때도, 해당 레이아웃 데이터가 필요하면 (SvelteKit이 똑똑하게 판단해서) 다시 불릴 수 있어.

**쓸 때 꿀팁 및 주의사항:**
*   **데이터 상속 & 덮어쓰기**: 레이아웃 `load` 데이터는 모든 자식 페이지, 자식 레이아웃의 `load` 함수, 그리고 최종적으로 페이지 컴포넌트(`+page.svelte`)에서 `data` prop으로 접근 가능해. 만약 부모 레이아웃이랑 자식 페이지 `load`에서 같은 이름의 키로 데이터를 반환하면, 더 안쪽(자식) 것이 우선권을 가져서 덮어쓴다. "내 데이터가 더 중요하거든?"
*   **`await parent()`로 부모 데이터 활용**: 자식 레이아웃이나 페이지의 `load` 함수 안에서 `const parentData = await parent()` 이렇게 호출하면 바로 위 부모 레이아웃의 `load` 함수가 반환한 데이터에 접근할 수 있어. 이걸로 부모 데이터 기반으로 추가 작업하기 편함.
*   **레이아웃 `load`는 신중하게**: 여기가 무거워지면 이 레이아웃 쓰는 모든 페이지 로딩이 느려터지는 대참사가 발생. 진짜 공통으로, 자주 필요한 데이터만 넣자. "모두의 발목을 잡을 셈이냐!"
*   **데이터 변경 감지**: SvelteKit이 알아서 데이터 변경 시 `load` 함수를 재실행한다지만, 너무 복잡한 의존성이 있거나 하면 가끔 개발자 멱살 잡을 때도 있으니 테스트는 필수.
*   **서버 전용 `+layout.server.js`**: `+page.js`처럼, 레이아웃 데이터도 서버에서만 실행해야 하는 로직(DB 직접 접근, 비밀 키 사용 등)이 있다면 `+layout.server.js` 파일을 사용하면 돼. "이건 너네(클라이언트)한테 보여줄 수 없는 고급 정보다."