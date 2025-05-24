**Manual setup (수동 설정)**

**`<title>`과 `<meta>` 태그**
모든 페이지에는 `<svelte:head>` 안에 잘 작성된 고유한 `<title>` 태그와 `<meta name="description">` 태그가 있어야 해. 어떻게 하면 검색엔진이 이해하기 쉽게 제목이랑 설명을 잘 쓸 수 있는지, 그리고 다른 SEO 팁들은 구글 라이트하우스 SEO 감사 문서에 잘 나와 있으니 참고하라고. "검색엔진 님, 제 페이지 좀 예쁘게 봐주세요~" 하는 거지.

흔한 패턴 중 하나는 페이지 로드 함수에서 SEO 관련 데이터를 반환하고, 이걸 루트 레이아웃의 `<svelte:head>` 안에서 `$page.data` 형태로 가져다 쓰는 거야.

**사이트맵 (Sitemaps)**
사이트맵은 검색엔진이 네 사이트의 페이지 우선순위를 정하는 데 도움을 줘. 특히 컨텐츠 양이 어마어마할 때 유용하지. 엔드포인트를 사용해서 동적으로 사이트맵을 만들 수 있어.

```typescript
// src/routes/sitemap.xml/+server.ts 예시
export async function GET() {
	return new Response(
		`
		<?xml version="1.0" encoding="UTF-8" ?>
		<urlset
			xmlns="https://www.sitemaps.org/schemas/sitemap/0.9"
			xmlns:xhtml="https://www.w3.org/1999/xhtml"
			xmlns:mobile="https://www.google.com/schemas/sitemap-mobile/1.0"
			xmlns:news="https://www.google.com/schemas/sitemap-news/0.9"
			xmlns:image="https://www.google.com/schemas/sitemap-image/1.1"
			xmlns:video="https://www.google.com/schemas/sitemap-video/1.1"
		>
			<!-- 여기에 <url> 요소들이 들어감 -->
		</urlset>`.trim(),
		{
			headers: {
				'Content-Type': 'application/xml'
			}
		}
	);
}
```

**AMP (Accelerated Mobile Pages)**
요즘 웹 개발하다 보면 좀 짜증나지만 가끔 AMP 버전의 사이트를 만들어야 할 때가 있어. 스벨트킷에서는 `inlineStyleThreshold` 옵션을 설정하고...

```javascript
// svelte.config.js 예시
/** @type {import('@sveltejs/kit').Config} */
const config = {
	kit: {
		// <link rel="stylesheet">는 허용 안 되니까 모든 스타일을 인라인으로 박아버려
		inlineStyleThreshold: Infinity
	}
};

export default config;
```

...루트 `+layout.js`나 `+layout.server.js`에서 클라이언트 사이드 렌더링(CSR)을 비활성화하고...

```javascript
// src/routes/+layout.server.ts 예시
export const csr = false;
```

...`app.html`에 `amp` 속성을 추가하고...

```html
<html amp>
...
</html>
```

...마지막으로 `@sveltejs/amp`에서 가져온 `transform` 함수랑 `transformPageChunk`를 써서 HTML을 변환해주면 돼.

```typescript
// src/hooks.server.ts 예시
import * as amp from '@sveltejs/amp';
import type { Handle } from '@sveltejs/kit';

export const handle: Handle = async ({ event, resolve }) => {
	let buffer = '';
	return await resolve(event, {
		transformPageChunk: ({ html, done }) => {
			buffer += html;
			if (done) return amp.transform(buffer);
		}
	});
};
```

AMP 페이지로 변환하면서 안 쓰는 CSS까지 몽땅 포함되는 걸 막으려면 `dropcss`를 쓸 수 있어.

```typescript
// src/hooks.server.ts (dropcss 사용 예시)
import * as amp from '@sveltejs/amp';
import dropcss from 'dropcss';
import type { Handle } from '@sveltejs/kit';

export const handle: Handle = async ({ event, resolve }) => {
	let buffer = '';

	return await resolve(event, {
		transformPageChunk: ({ html, done }) => {
			buffer += html;

			if (done) {
				let css = '';
				const markup = amp
					.transform(buffer)
					.replace('⚡', 'amp') // dropcss가 이 문자 처리를 못 함
					.replace(/<style amp-custom([^>]*?)>([^]+?)<\/style>/, (match, attributes, contents) => {
						css = contents;
						return `<style amp-custom${attributes}></style>`;
					});

				css = dropcss({ css, html: markup }).css;
				return markup.replace('</style>', `${css}</style>`);
			}
		}
	});
};
```

페이지를 미리 렌더링(prerendering)하는 경우라면, `handle` 훅에서 `amphtml-validator`를 써서 변환된 HTML을 검증하는 게 좋아. 근데 이거 엄청 느리니까 주의하고.

---

**얘 뭐 하는 애냐?**
"Manual setup"은 스벨트킷이 자동으로 다 못 해주는, 개발자가 직접 손봐야 하는 설정들을 모아놓은 거야. 주로 검색엔진이 네 사이트를 더 잘 이해하고, 모바일에서 더 빠르게 뜨도록 (AMP) 도와주는 작업들이지. "자동차가 아무리 좋아도 운전은 내가 해야지" 뭐 이런 느낌.

**왜 쓰는데?**
1.  **SEO 극대화 (`<title>`, `<meta>`, 사이트맵):** 검색 결과 상단에 뜨고 싶으면 검색엔진한테 잘 보여야 하잖아? 제목, 설명 똑바로 달고, 사이트맵으로 "우리 집 구조는 이렇습니다" 하고 알려주는 거야. "검색엔진 님, 우리 가게 간판 잘 보이게 달았어요! 메뉴판도 여기!"
2.  **모바일 사용자 경험 개선 (AMP):** 특히 인터넷 느린 환경의 모바일 사용자들한테 번개처럼 빠른 페이지를 보여주고 싶을 때 AMP를 써. 구글이 좋아하기도 하고. "느려터진 로딩은 이제 그만! 3초 안에 안 뜨면 손절각."
3.  **사이트 관리 효율 증대 (사이트맵):** 사이트맵은 검색엔진뿐만 아니라 개발자 자신에게도 사이트 구조를 파악하는 데 도움이 돼.

**언제 불려 나오냐?**
*   **`<title>`/`<meta>`:** 모든 페이지 만들 때마다 신경 써야 해. 이건 그냥 기본 소양.
*   **사이트맵:** 사이트 규모가 좀 커지거나, 컨텐츠가 자주 바뀔 때 만들어두면 좋아. 보통 `sitemap.xml`이라는 주소로 제공하지.
*   **AMP:** 모바일 트래픽이 아주 중요하거나, 구글 뉴스 같은 특정 플랫폼에 컨텐츠를 노출시키고 싶을 때 고려해. 근데 이거 설정하는 거 좀 귀찮으니까 진짜 필요할 때만.

**쓸 때 꿀팁 및 주의사항:**
*   **`<title>`/`<meta>`는 정성스럽게:** 대충 키워드만 때려 박는다고 능사가 아니야. 사용자가 클릭하고 싶게, 페이지 내용을 정확히 요약해야 해. "낚시성 제목 달면 바로 블랙리스트행."
*   **사이트맵은 최신 상태로 유지:** 새 페이지 생기거나 없어지면 사이트맵도 업데이트해줘야 검색엔진이 헷갈리지 않아. 동적으로 생성하는 게 속 편한 이유지.
*   **AMP는 양날의 검:** 빠르긴 한데, 제약사항이 많아서 일반 웹페이지처럼 화려하게 만들긴 어려워. CSS도 인라인으로 넣어야 하고, 자바스크립트도 제한적이야. "속도를 위해 많은 것을 포기해야 할 수도 있다." 꼭 필요한지, 유지보수 비용은 감당 가능한지 잘 따져봐야 해.
*   **`dropcss`는 신중하게:** 안 쓰는 CSS 쳐내는 건 좋은데, 혹시 필요한 CSS까지 날려버리면 화면 깨질 수 있으니 테스트 철저히.
*   **AMP 검증은 필수, 근데 느림:** `amphtml-validator`로 AMP 페이지가 규격에 맞는지 꼭 확인해야 해. 근데 빌드 타임에만 돌리는 게 정신 건강에 이로워. "느린 검증기는 개발자의 인내심을 시험한다."