`toSSG`
`toSSG`는 정적 사이트(Static Site Generation, SSG)를 만드는 핵심 함수야. Hono 애플리케이션이랑 파일 시스템 모듈을 인자로 받아서 돌아가지. 기본 원리는 이래:

**입력 (Input)**
`toSSG`에 넘겨주는 인자들은 `ToSSGInterface`에 정의되어 있어.

```typescript
export interface ToSSGInterface {
  (
    app: Hono, // 네가 만든 Hono 앱 인스턴스
    fsModule: FileSystemModule, // 파일 쓰고 디렉토리 만드는 기능이 있는 놈
    options?: ToSSGOptions // 추가 옵션 (선택 사항)
  ): Promise<ToSSGResult> // 결과물 (성공 여부, 파일 목록 등)
}
```

*   `app`: 라우트가 등록된 `new Hono()` 인스턴스를 지정해. "어떤 내용을 정적 파일로 구울 건지 레시피를 줘!" 하는 거지.
*   `fs`: `node:fs/promises` 같은 파일 시스템 모듈 객체를 넘겨줘. 대충 이런 모양이야:

    ```typescript
    export interface FileSystemModule {
      writeFile(path: string, data: string | Uint8Array): Promise<void> // 파일 쓰는 기능
      mkdir(
        path: string,
        options: { recursive: boolean } // 하위 디렉토리까지 한 번에 만드는 옵션
      ): Promise<void | string> // 디렉토리 만드는 기능
    }
    ```

**Deno랑 Bun용 어댑터 사용하기**
Deno나 Bun 환경에서 SSG 쓰고 싶으면, 각 파일 시스템에 맞는 `toSSG` 함수가 이미 준비되어 있어. 따로 `fsModule` 안 넘겨도 돼서 편해.

*   **Deno용**:

    ```typescript
    import { toSSG } from 'hono/deno'

    toSSG(app) // 두 번째 인자는 ToSSGOptions 타입의 옵션 객체 (선택)
    ```
*   **Bun용**:

    ```typescript
    import { toSSG } from 'hono/bun'

    toSSG(app) // 얘도 두 번째 인자는 ToSSGOptions 타입의 옵션 객체 (선택)
    ```

**옵션 (Options)**
옵션은 `ToSSGOptions` 인터페이스에 정의되어 있어.

```typescript
export interface ToSSGOptions {
  dir?: string // 정적 파일 저장할 디렉토리 경로
  concurrency?: number // 동시에 몇 개나 파일 만들 건지 (병렬 처리 개수)
  beforeRequestHook?: BeforeRequestHook // 요청 보내기 전에 실행할 훅
  afterResponseHook?: AfterResponseHook // 응답 받고 나서 실행할 훅
  afterGenerateHook?: AfterGenerateHook // 파일 생성 완료 후 실행할 훅
  extensionMap?: Record<string, string> // Content-Type별 파일 확장자 매핑
}
```

*   `dir`: 정적 파일들을 어디에 저장할지 경로를 지정해. 기본값은 `./static`이야. "구운 빵 어디다 둘래?"
*   `concurrency`: 동시에 몇 개의 파일을 생성할지 정하는 숫자야. 기본값은 2. "빵 굽는 오븐 몇 개 돌릴래?" (너무 높이면 컴퓨터 힘들어한다)
*   `extensionMap`: `Content-Type`을 키로, 파일 확장자를 값으로 가지는 맵이야. 예를 들어 `Content-Type: application/json`이면 `.json` 파일로, `text/html`이면 `.html` 파일로 저장하도록 확장자를 결정할 때 써.
*   각종 `Hook`들은 나중에 설명한다는데, 대충 특정 시점에 끼어들어서 추가 작업을 할 수 있게 해주는 갈고리 같은 거야.

**출력 (Output)**
`toSSG` 함수는 작업 결과를 아래 `ToSSGResult` 타입으로 반환해.

```typescript
export interface ToSSGResult {
  success: boolean // 성공했냐? (true/false)
  files: string[] // 생성된 파일들 경로 목록
  error?: Error // 혹시 에러 났으면 여기에 에러 객체가 담김
}
```

---

**얘 뭐 하는 애냐?**
`toSSG`는 네가 만든 Hono 앱의 각 페이지(라우트)들을 미리 다 방문해서 그 결과물(주로 HTML)을 실제 파일로 저장해주는 녀석이야. 이렇게 만들어진 파일들을 웹서버에 그냥 올려두면, 사용자가 접속할 때마다 서버가 동적으로 페이지를 만드는 게 아니라 이미 만들어진 파일을 바로바로 보여주니까 속도가 엄청 빨라지지. "공장에서 빵 미리 다 구워놓고 손님 오면 바로 내주는 시스템"이라고 보면 돼. 블로그, 문서 사이트, 포트폴리오처럼 내용이 자주 바뀌지 않는 사이트에 딱이지.

**왜 쓰는데?**
1.  **성능 끝판왕**: 이미 완성된 HTML 파일을 제공하니까 로딩 속도가 미친 듯이 빨라. 서버에서 뭐 계산하고 데이터베이스 뒤지고 할 필요가 없거든. "0.1초 컷 로딩 가즈아!"
2.  **서버 부담 감소 & 비용 절감**: 정적 파일만 서빙하면 되니까 서버 자원을 거의 안 써. 트래픽 몰려도 끄떡없고, 서버 비용도 확 줄일 수 있어. "서버야, 이제 좀 쉬어도 돼."
3.  **보안 강화**: 서버 사이드 로직이 거의 없으니 해킹당할 구멍도 줄어들어. 데이터베이스 털릴 걱정? 그게 뭐죠?
4.  **배포 용이성**: 생성된 정적 파일들을 Netlify, Vercel, GitHub Pages 같은 곳에 그냥 턱 올려놓기만 하면 배포 끝. "드래그 앤 드롭으로 배포 완료!"

**언제 불려 나오냐?**
주로 빌드(build) 과정에서 한 번 실행돼. 네가 Hono 앱 코드를 다 짜고, "자, 이제 이걸 정적 사이트로 만들자!" 할 때 `node build-script.js` 같은 명령어로 `toSSG` 함수를 호출하는 스크립트를 실행하는 거지. 그러면 `dir` 옵션으로 지정한 폴더에 HTML, CSS, JS, 이미지 파일들이 쫙 생성될 거야.

**쓸 때 꿀팁 및 주의사항:**
*   **모든 페이지가 정적일 필요는 없어**: `toSSG`는 앱에 등록된 모든 GET 요청 라우트를 순회하면서 파일을 만들려고 해. 만약 특정 페이지만 정적으로 만들고 싶다면, `options`의 `filter` (문서엔 없지만 다른 SSG 툴엔 흔함, Hono에선 라우트 등록 시 조절하거나 훅을 이용해야 할 수도) 같은 걸 쓰거나, 아예 SSG용 Hono 앱 인스턴스를 따로 만들어서 필요한 라우트만 등록하는 게 좋아. "뷔페 음식 다 담을 필요 있나? 맛있는 것만 골라 담자."
*   **동적 데이터 처리 전략**: 사용자마다 다른 내용을 보여줘야 하거나, 실시간 데이터가 필요한 부분은 SSG랑 안 맞아. 이런 부분은 클라이언트 사이드 자바스크립트로 API 호출해서 채워 넣거나 (CSR 방식 혼용), 아니면 그 페이지만 SSR로 돌리는 하이브리드 방식을 고민해야 해. "정적 사이트라고 해서 모든 게 멈춰 있을 필요는 없다."
*   **빌드 시간**: 페이지가 수천, 수만 개 되면 `toSSG` 실행 시간이 꽤 오래 걸릴 수 있어. `concurrency` 옵션 조절하고, 불필요한 작업은 줄여서 빌드 시간을 최적화해야 해. "빵 수만 개 굽는데 하루 종일 걸리면 안 되잖아?"
*   **파일 시스템 모듈 선택**: Node.js 환경이면 `fs/promises`를 직접 구현해서 넘겨주거나, Hono가 제공하는 Node.js용 어댑터(`hono/node-server` 쪽 유틸리티를 찾아봐야 할 수도 있어. 문서에는 Deno/Bun용만 명시됨)를 써야 해. Deno나 Bun은 내장 `toSSG` 쓰면 알아서 해주니까 편하고.
*   **`extensionMap` 활용**: API 엔드포인트가 `/api/users`인데 `application/json`을 반환한다면, `extensionMap: { 'application/json': '.json' }` 이렇게 설정해서 `/api/users.json` 파일로 저장되게 할 수 있어. 그래야 클라이언트가 데이터 가져가기 편하지.
*   **훅(Hook)을 잘 쓰면 고급짐**: `beforeRequestHook`으로 각 페이지 렌더링 전에 특정 데이터를 주입하거나, `afterResponseHook`으로 생성된 HTML을 한 번 더 가공하거나, `afterGenerateHook`으로 사이트맵 생성 같은 후처리 작업을 자동화할 수 있어. "자동화는 개발자의 미덕."