`animate` 디렉티브는 키(key)가 있는 `each` 블록 안의 내용물 순서가 바뀔 때 애니메이션을 터뜨립니다. 요소가 추가되거나 삭제될 때는 작동 안 하고, 오직 `each` 블록 안 기존 데이터 아이템의 순번(index)이 바뀔 때만 움직입니다. `animate` 디렉티브는 반드시 키가 있는 `each` 블록 바로 밑에 있는 자식 요소에 붙여야 합니다.

애니메이션은 스벨트(Svelte)에 내장된 애니메이션 함수나 직접 만든 커스텀 함수를 쓸 수 있습니다.

```html
<!-- `list` 배열의 순서가 바뀌면 애니메이션이 실행됩니다 -->
{#each list as item, index (item)}
	<li animate:flip>{item}</li>
{/each}
```

**애니메이션 파라미터**
액션(actions)이나 트랜지션(transitions)처럼 애니메이션에도 파라미터를 넘길 수 있습니다.

(이중 중괄호 `{{ }}`는 특별한 문법이 아니고, 그냥 표현식 태그 안에 객체 리터럴을 쓴 겁니다.)

```html
{#each list as item, index (item)}
	<li animate:flip={{ delay: 500 }}>{item}</li>
{/each}
```

**커스텀 애니메이션 함수**
```typescript
animation = (node: HTMLElement, { from: DOMRect, to: DOMRect } , params: any) => {
	delay?: number, // 지연 시간 (ms)
	duration?: number, // 지속 시간 (ms)
	easing?: (t: number) => number, // 이징 함수 (0~1 사이 값으로 변화 속도 조절)
	css?: (t: number, u: number) => string, // CSS 문자열 반환 함수
	tick?: (t: number, u: number) => void // 매 프레임마다 실행될 함수
}
```
애니메이션에는 노드(node, 해당 HTML 요소), 애니메이션 객체({ from, to }), 그리고 파라미터(params)를 인자로 받는 커스텀 함수를 쓸 수 있습니다. 애니메이션 파라미터는 `from`과 `to` 속성을 가진 객체인데, 각각 요소의 시작 위치와 끝 위치에서의 기하학적 정보(크기와 위치)를 담은 `DOMRect` 객체입니다. `from`은 시작 위치의 `DOMRect`이고, `to`는 리스트 순서가 바뀌고 DOM이 업데이트된 후의 최종 위치 `DOMRect`입니다.

만약 반환된 객체에 `css` 메서드가 있다면, 스벨트는 해당 요소에서 재생될 웹 애니메이션을 만듭니다.

`css` 함수에 전달되는 `t` 인자는 이징 함수가 적용된 후 0에서 1로 변하는 값입니다. `u` 인자는 `1 - t`와 같습니다.

이 함수는 애니메이션 시작 전에 `t`와 `u` 값을 바꿔가며 여러 번 호출됩니다.

**앱 예시 (css 사용)**
```html
<script lang="ts">
	import { cubicOut } from 'svelte/easing'; // 스벨트 이징 함수 중 하나

	// 'whizz'라는 이름의 커스텀 애니메이션 함수
	function whizz(node: HTMLElement, { from, to }: { from: DOMRect; to: DOMRect }, params: any) {
		const dx = from.left - to.left; // x축 이동 거리
		const dy = from.top - to.top;   // y축 이동 거리

		const d = Math.sqrt(dx * dx + dy * dy); // 총 이동 거리 (피타고라스 정리)

		return {
			delay: 0, // 지연 없음
			duration: Math.sqrt(d) * 120, // 거리에 비례하는 지속 시간
			easing: cubicOut, // 튕기듯 나가는 이징 효과
			// t: 애니메이션 진행도 (0->1), u: 역진행도 (1->0)
			// u * dx, u * dy: 시작점 기준으로 얼마나 이동했는지
			// t * 360: 진행도에 따라 360도 회전
			css: (t, u) => `transform: translate(${u * dx}px, ${u * dy}px) rotate(${t * 360}deg);`
		};
	}
</script>

{#each list as item, index (item)}
	<div animate:whizz>{item}</div>
{/each}
```

커스텀 애니메이션 함수는 `tick` 함수를 반환할 수도 있는데, 이 함수는 애니메이션 도중 같은 `t`와 `u` 인자를 받으며 호출됩니다.

만약 `tick` 대신 `css`를 쓸 수 있다면 그렇게 하세요. 웹 애니메이션은 메인 스레드와 별개로 돌아갈 수 있어서, 느린 기기에서도 버벅임(jank)을 막아줍니다.

**앱 예시 (tick 사용)**
```html
<script lang="ts">
	import { cubicOut } from 'svelte/easing';

	function whizz(node: HTMLElement, { from, to }: { from: DOMRect; to: DOMRect }, params: any) {
		const dx = from.left - to.left;
		const dy = from.top - to.top;

		const d = Math.sqrt(dx * dx + dy * dy);

		return {
			delay: 0,
			duration: Math.sqrt(d) * 120,
			easing: cubicOut,
			// 애니메이션 진행도(t)가 0.5를 넘으면 글자색 Pink, 아니면 Blue
			tick: (t, u) => Object.assign(node.style, { color: t > 0.5 ? 'Pink' : 'Blue' })
		};
	}
</script>

{#each list as item, index (item)}
	<div animate:whizz>{item}</div>
{/each}
```

---

**얘 뭐 하는 애냐? (기능 및 목적)**
`animate`는 `{#each ...}` 블록 안에서 아이템들 순서가 뒤죽박죽될 때 "어이쿠, 내가 원래 여기 있었는데 저기로 가네?" 하면서 스르륵 움직이는 효과를 주는 녀석입니다. 목적은 단 하나, UI 변경 사항을 사용자 눈에 부드럽게 보여줘서 "이 앱, 좀 세련됐는데?" 소리 듣게 하는 거죠. 갑자기 화면 요소 위치가 뿅 하고 바뀌면 사용자는 멀미합니다.

**왜 쓰는데?**
1.  **UX 향상**: 리스트 정렬이나 필터링으로 아이템 순서가 바뀔 때, 뭐가 어떻게 변했는지 시각적으로 알려줘서 사용자가 "내 아이템 어디 갔어!" 하고 당황하는 걸 막아줍니다.
2.  **생동감 부여**: 정적인 화면보다 동적인 화면이 훨씬 보기 좋죠. 앱이 살아 움직이는 듯한 느낌을 줍니다. "이 앱, 펄떡펄떡 살아있네!"
3.  **개발 편의성**: 복잡한 위치 계산이나 CSS 트랜지션 직접 안 짜도 됩니다. 스벨트가 `from` (이전 위치), `to` (새 위치) 정보를 다 알려주니, 개발자는 "어떻게 움직일까?"만 고민하면 됩니다. "애니메이션은 스벨트에게 맡기고 칼퇴하자."

**언제 불려 나오냐?**
키가 있는 (`(item.id)` 같은 거) `{#each ...}` 블록 바로 아래 있는 HTML 요소에 `animate:함수이름` 형태로 딱 붙여놓으면 됩니다. 그리고 그 `{#each ...}` 블록이 바라보는 배열 데이터의 순서가 실제로 변경될 때 (예: `list.sort()` 호출, 또는 완전히 새로운 순서의 배열로 교체) 스벨트가 "어, 순서 바뀌었네? 애니메이션 틀어!" 하면서 해당 요소의 `animate`에 지정된 함수를 호출합니다.

**쓸 때 꿀팁 및 주의사항:**
*   **키(key)는 생명줄**: `{#each list as item (item.id)}`처럼 각 아이템을 고유하게 식별할 수 있는 키가 **반드시** 있어야 `animate`가 제대로 작동합니다. 키 없으면 스벨트는 "어떤 놈이 어떤 놈으로 변한 거야?" 하고 헷갈려서 애니메이션이고 뭐고 없습니다. "주민번호 없으면 동명이인 구분 못 하는 거랑 똑같아!"
*   **추가/삭제는 `transition` 담당**: `animate`는 오로지 '순서 변경'에만 관여합니다. 아이템이 목록에 새로 추가되거나 아예 사라질 때의 애니메이션은 `in:`, `out:`, `transition:` 디렉티브의 영역입니다. "선 넘지 마라, 각자 할 일 하자."
*   **직계 자식에게만 허락된 영광**: `{#each ...} <div><p animate:flip>...</p></div> {/each}` 이런 식으로 `each` 블록 한 단계 더 안쪽에 있는 요소에 `animate` 붙이면 작동안합니다. `animate`는 `each` 바로 밑에 있는 첫 번째 자식 요소, 즉 `div`에 붙여야 합니다. "내 바로 밑 딱까리만 챙긴다."
*   **`from` / `to`는 보물지도**: 커스텀 애니메이션 만들 때 `from` (이전 위치/크기)과 `to` (새 위치/크기) `DOMRect` 객체는 애니메이션 로직의 핵심입니다. 이걸로 이동 거리, 방향, 크기 변화 등 모든 걸 계산할 수 있습니다. "이 좌표만 있으면 어디든 갈 수 있어!"
*   **성능은 `css` > `tick`**: 커스텀 애니메이션 함수가 `css` 함수를 반환하면 스벨트가 브라우저의 네이티브 웹 애니메이션 기능을 사용합니다. 이건 메인 스레드 부담을 덜 줘서 애니메이션이 훨씬 부드럽고, 특히 저사양 기기에서 버벅임(jank)이 줄어듭니다. 반면 `tick` 함수는 매 프레임마다 자바스크립트로 스타일을 직접 바꾸는 거라, 복잡하면 성능에 부담될 수 있습니다. "웬만하면 CSS한테 맡겨, 걔가 하드웨어 가속도 쓰고 더 잘해."
*   **Svelte 내장 `flip` 활용**: `import { flip } from 'svelte/animate';` 해서 `animate:flip` 쓰면 간단한 위치 변경 애니메이션은 그냥 거저먹는 수준입니다. FLIP (First, Last, Invert, Play) 애니메이션 기법을 쉽게 구현해줍니다. "복잡하게 생각 말고 일단 flip부터 써봐."