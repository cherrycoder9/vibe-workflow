서비스 워커 안에서는 `$service-worker`라는 모듈을 쓸 수 있는데, 이놈이 모든 정적 파일(static assets), 빌드 결과물, 그리고 미리 렌더링된 페이지들의 경로를 다 알려줘. 앱 버전 문자열도 제공해서 이걸로 고유한 캐시 이름을 만들 수 있고, 배포된 경로의 기본 주소(base path)도 알 수 있지. 만약 네 Vite 설정 파일에 `define` (전역 변수 치환용) 옵션이 있다면, 이건 서버나 클라이언트 빌드뿐만 아니라 서비스 워커에도 똑같이 적용돼. "어디든 공평하게!"

아래 예제 코드는 빌드된 앱이랑 `static` 폴더 안의 모든 파일을 일단 무조건 캐시에 때려 박고, 나머지 요청들은 실제로 요청이 발생할 때마다 캐시하는 방식이야. 이렇게 하면 각 페이지는 한번 방문하고 나면 오프라인에서도 잘 돌아가지. "인터넷 끊겨도 괜찮아, 한번 본 건 기억하거든!"

```typescript
/// <reference types="@sveltejs/kit" />
import { build, files, version } from '$service-worker';

// 이 배포판만의 고유한 캐시 이름을 만들자
const CACHE = `cache-${version}`;

const ASSETS = [
	...build, // 앱 그 자체
	...files  // `static` 폴더 안의 모든 것들
];

self.addEventListener('install', (event) => {
	// 새 캐시를 만들고 모든 파일을 거기에 추가한다
	async function addFilesToCache() {
		const cache = await caches.open(CACHE);
		await cache.addAll(ASSETS);
	}

	event.waitUntil(addFilesToCache());
});

self.addEventListener('activate', (event) => {
	// 디스크에서 이전 캐시 데이터를 삭제한다
	async function deleteOldCaches() {
		for (const key of await caches.keys()) {
			if (key !== CACHE) await caches.delete(key);
		}
	}

	event.waitUntil(deleteOldCaches());
});

self.addEventListener('fetch', (event) => {
	// POST 요청 같은 건 무시한다
	if (event.request.method !== 'GET') return;

	async function respond() {
		const url = new URL(event.request.url);
		const cache = await caches.open(CACHE);

		// `build`나 `files`에 있는 건 항상 캐시에서 꺼내 쓸 수 있다
		if (ASSETS.includes(url.pathname)) {
			const response = await cache.match(url.pathname);

			if (response) {
				return response;
			}
		}

		// 나머지 모든 것들은 일단 네트워크로 시도해보고,
		// 만약 오프라인이면 캐시에서 가져온다
		try {
			const response = await fetch(event.request);

			// 오프라인 상태면, fetch가 에러를 던지는 대신 Response가 아닌 값을 반환할 수 있다
			// 그리고 이 Response가 아닌 값은 respondWith에 넘길 수 없다
			if (!(response instanceof Response)) {
				throw new Error('fetch로부터 유효하지 않은 응답');
			}

			if (response.status === 200) {
				cache.put(event.request, response.clone());
			}

			return response;
		} catch (err) {
			const response = await cache.match(event.request);

			if (response) {
				return response;
			}

			// 캐시에도 없으면, 이 요청에 응답할 방법이 없으니
			// 그냥 에러를 던진다
			throw err;
		}
	}

	event.respondWith(respond());
});
```

캐시할 때는 정신 바짝 차려야 해! 어떤 경우에는 오래된 데이터(stale data)가 오프라인일 때 아예 데이터를 못 쓰는 것보다 더 최악일 수 있거든. 브라우저는 캐시가 너무 꽉 차면 비워버리니까, 비디오 파일처럼 용량 큰 애들 캐시할 때는 특히 조심해야 한다. "욕심부리다 캐시 터진다!"

---

**얘 뭐 하는 애냐?**
서비스 워커는 웹 브라우저 백그라운드에서 돌아가는 스크립트야. 웹페이지랑은 별개로 작동해서, 페이지가 닫혀있어도 푸시 알림을 받거나 데이터를 동기화하는 등의 작업을 할 수 있게 해줘. 핵심 기능은 네트워크 요청을 가로채서 캐시된 응답을 보내거나, 오프라인일 때 미리 저장된 콘텐츠를 보여주는 등 "네트워크 연결에 구애받지 않는 웹 경험"을 만드는 거지. "인터넷 없어도 든든한 내 편!"

**왜 쓰는데?**
1.  **오프라인 지원:** 이게 제일 큰 이유. 한번 방문한 페이지나 앱의 주요 기능들을 인터넷 연결 없이도 쓸 수 있게 해줘. 비행기 안이나 데이터 다 썼을 때 개꿀. "데이터 거지도 앱은 쓴다!"
2.  **성능 향상 (로딩 속도):** 자주 쓰는 파일들(이미지, CSS, JS 등)을 캐시에 저장해뒀다가 필요할 때 바로 꺼내 쓰니까, 서버까지 갔다 올 필요 없이 페이지가 팍팍 떠. "로딩 바는 이제 그만."
3.  **푸시 알림:** 웹사이트가 닫혀 있어도 사용자한테 알림을 보낼 수 있게 해줘. 쇼핑몰 새 상품 알림, 뉴스 속보 같은 거. "자니? 아니, 새 소식 왔어."
4.  **백그라운드 동기화:** 사용자가 뭔가 입력했는데 인터넷이 안 터져서 전송 실패했을 때, 나중에 인터넷 연결되면 자동으로 다시 보내주는 기능 같은 거. "너는 입력만 해, 전송은 내가 알아서 할게."

**언제 불려 나오냐?**
*   **설치 (`install` 이벤트):** 사용자가 서비스 워커를 사용하는 웹사이트에 처음 방문하면, 브라우저가 서비스 워커 스크립트를 다운로드하고 설치하려고 시도해. 이때 `install` 이벤트가 발생하고, 주로 정적 파일들을 캐시에 저장하는 작업을 여기서 해.
*   **활성화 (`activate` 이벤트):** 설치가 성공적으로 끝나고, 기존에 돌아가던 서비스 워커가 없다면 (또는 기존 서비스 워커 제어가 끝났다면) 새 서비스 워커가 활성화돼. `activate` 이벤트에서는 주로 오래된 캐시를 정리하는 작업을 하지.
*   **요청 가로채기 (`fetch` 이벤트):** 서비스 워커가 활성화되면, 그 범위 내의 페이지에서 발생하는 모든 네트워크 요청(이미지, API 호출 등)을 가로챌 수 있어. `fetch` 이벤트 핸들러에서 이 요청을 받아서 캐시에서 응답할지, 네트워크로 보낼지, 아니면 둘 다 시도할지 결정하는 거야.

**쓸 때 꿀팁 및 주의사항:**
*   **캐시 전략이 전부다:** 뭘 얼마나 자주 캐시하고, 언제 업데이트할지 계획 잘 짜야 해. 안 그러면 사용자한테 곰팡내 나는 옛날 데이터 보여주다가 욕먹는다. "신선도가 생명!"
*   **HTTPS는 기본 소양:** 서비스 워커는 보안 때문에 HTTPS 환경에서만 돌아가. HTTP에서는 꿈도 꾸지 마. "안전하지 않으면 안 쓴다."
*   **범위(Scope)를 이해하라:** 서비스 워커는 특정 경로 범위에만 영향을 미쳐. `/app/` 아래에 등록하면 `/app/` 하위 경로에만 적용되는 식. "내 나와바리는 내가 지킨다."
*   **업데이트 프로세스 복잡:** 서비스 워커 파일 자체가 변경되면 브라우저는 새 버전을 감지하고 백그라운드에서 설치해. 근데 기존 서비스 워커가 제어하는 페이지가 하나라도 열려있으면 새 버전은 대기만 타. 모든 관련 탭이 닫히거나 `self.skipWaiting()`과 `clients.claim()`을 써서 강제로 활성화해야 새 버전이 일 시작함. "새 일꾼 왔는데 옛날 일꾼이 안 비켜주네?"
*   **디버깅은 개발자 도구와 함께:** 크롬 개발자 도구의 `Application` 셔 `Service Workers` 탭에서 현재 등록된 서비스 워커 상태 확인, 업데이트, 강제 해제 등을 할 수 있어. `Cache Storage` 탭에서는 캐시 내용도 까볼 수 있고. "문제 생기면 일단 여기부터 뒤져봐."
*   **`POST` 요청 등은 신중하게:** 예제 코드처럼 `GET` 요청만 주로 캐시 대상으로 삼는 게 안전빵이야. `POST` 같은 데이터 변경 요청을 어설프게 캐시하거나 오프라인에서 처리하려고 하면 데이터 꼬이기 십상. "읽기 전용은 괜찮은데, 쓰기는 조심 또 조심!"