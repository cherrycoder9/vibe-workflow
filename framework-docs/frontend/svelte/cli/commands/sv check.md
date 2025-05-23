`sv check`는 프로젝트에서 오류나 경고를 찾아내는 탐정 같은 놈입니다. 예를 들면 이런 것들을 잡아내죠:

*   안 쓰는 CSS (옷장에 처박아둔 헌 옷 같은 존재)
*   웹 접근성 관련 힌트 (모두를 위한 배려, 놓치면 안 되죠)
*   자바스크립트/타입스크립트 컴파일러 오류 (코드의 오타나 문법 오류)

이거 돌리려면 Node 16 이상 버전이 필요합니다.

**설치**

프로젝트에 `svelte-check` 패키지를 먼저 깔아야 합니다:

```bash
npm i -D svelte-check
```

**사용법**

```bash
npx sv check
```

**옵션**

*   `--workspace <경로>`
    작업 공간 경로를 지정합니다. `node_modules`나 `--ignore`에 명시된 폴더 빼고 하위 디렉토리까지 전부 검사합니다. "내 나와바리는 여기까지다!"
*   `--output <형식>`
    오류와 경고를 어떻게 보여줄지 정합니다. 기계가 읽기 좋은 형식도 있어요.
    *   `human`: 사람이 보기 편한 일반적인 형태.
    *   `human-verbose`: 좀 더 자세한 사람 친화적 형태.
    *   `machine`: 기계가 파싱하기 좋은 형태. CI/CD 파이프라인에서 쓰기 좋죠.
    *   `machine-verbose`: 더 자세한 기계 친화적 형태.
*   `--watch`
    프로세스를 계속 살려두고 파일 변경 사항을 감시합니다. "눈 부릅뜨고 지켜본다!"
*   `--preserveWatchOutput`
    `--watch` 모드에서 화면이 지워지는 걸 막습니다. 이전 로그도 보고 싶을 때.
*   `--tsconfig <경로>`
    `tsconfig.json`이나 `jsconfig.json` 파일 경로를 직접 지정합니다. 이 옵션을 쓰면 해당 설정 파일의 `files`, `include`, `exclude` 패턴에 맞는 파일만 진단하고, 타입스크립트/자바스크립트 파일 오류도 보고합니다. 지정 안 하면 프로젝트 디렉토리부터 위로 올라가면서 설정 파일을 알아서 찾습니다.
*   `--no-tsconfig`
    현재 디렉토리와 그 하위의 `.svelte` 파일만 검사하고 `.js` / `.ts` 파일은 무시 (타입 검사 안 함)하고 싶을 때 씁니다. "Svelte 파일만 볼 거야, 나머진 신경 꺼!"
*   `--ignore <경로들>`
    작업 공간 루트 기준으로 무시할 파일/폴더를 지정합니다. 경로는 쉼표로 구분하고 따옴표로 감싸야 합니다. 예: `npx sv check --ignore "dist,build"`
    이 옵션은 `--no-tsconfig`랑 같이 쓸 때만 효과가 있습니다. `--tsconfig`랑 같이 쓰면, 진단 대상 파일은 `tsconfig.json`이 결정하고, 이 옵션은 감시 대상 파일에만 영향을 줍니다.
*   `--fail-on-warnings`
    이 옵션을 주면 경고만 있어도 `sv check`가 오류 코드를 내뱉으며 종료됩니다. "경고도 용납 못 해!"
*   `--compiler-warnings <경고들>`
    컴파일러 경고 코드와 처리 방식을 `코드:동작` 쌍으로 묶어 쉼표로 구분한 문자열을 전달합니다. `동작`은 `ignore` (무시) 또는 `error` (오류로 처리) 중 하나입니다. 예: `npx sv check --compiler-warnings "css_unused_selector:ignore,a11y_missing_attribute:error"`
*   `--diagnostic-sources <소스들>`
    어떤 소스에 대해 진단을 실행할지 쉼표로 구분된 문자열로 지정합니다. 기본적으로는 전부 활성화됩니다:
    *   `js` (타입스크립트 포함)
    *   `svelte`
    *   `css`
    예: `npx sv check --diagnostic-sources "js,svelte"`
*   `--threshold <수준>`
    표시할 진단 메시지 수준을 필터링합니다:
    *   `warning` (기본값) — 오류와 경고 모두 표시.
    *   `error` — 오류만 표시. "경고 따윈 시끄럽다!"

**문제 해결**
전처리기 설정이나 다른 문제 해결 정보는 language-tools 문서를 참고하세요.

**기계가 읽을 수 있는 출력 (Machine-readable output)**
`--output`을 `machine`이나 `machine-verbose`로 설정하면 CI 파이프라인이나 코드 품질 검사 도구처럼 기계가 읽기 쉽게 출력 형식을 바꿉니다.

각 줄은 새 레코드를 의미하고, 공백 하나로 구분된 열들로 구성됩니다. 모든 줄의 첫 번째 열은 모니터링에 쓸 수 있는 밀리초 단위 타임스탬프입니다. 두 번째 열은 "행 타입"인데, 이에 따라 뒤따르는 열의 개수와 타입이 달라질 수 있습니다.

첫 번째 줄은 `START` 타입이고 작업 공간 폴더(따옴표로 감싸임)를 포함합니다. 예:
`1590680325583 START "/home/user/language-tools/packages/language-server/test/plugins/typescript/testfiles"`

이후 `ERROR` 또는 `WARNING` 레코드가 여러 개 나올 수 있습니다. 구조는 동일하며 `output` 인자에 따라 다릅니다.

*   `machine` 형식: 파일명, 시작 줄/열 번호, 오류 메시지를 알려줍니다. 파일명은 작업 공간 디렉토리 기준 상대 경로입니다. 파일명과 메시지는 따옴표로 감싸입니다. 예:
    `1590680326283 ERROR "codeactions.svelte" 1:16 "Cannot find module 'blubb' or its corresponding type declarations."`
    `1590680326778 WARNING "imported-file.svelte" 0:37 "Component has unused export property 'prop'. If it is for external reference only, please consider using \`export const prop\`"`
*   `machine-verbose` 형식: 파일명, 시작 줄/열 번호, 끝 줄/열 번호, 오류 메시지, 진단 코드, 사람이 읽기 좋은 코드 설명, 사람이 읽기 좋은 진단 소스(예: svelte/typescript)를 알려줍니다. 파일명은 작업 공간 디렉토리 기준 상대 경로입니다. 각 진단은 로그의 타임스탬프가 앞에 붙은 ndjson 한 줄로 표현됩니다. 예:
    `1590680326283 {"type":"ERROR","fn":"codeaction.svelte","start":{"line":1,"character":16},"end":{"line":1,"character":23},"message":"Cannot find module 'blubb' or its corresponding type declarations.","code":2307,"source":"js"}`
    `1590680326778 {"type":"WARNING","filename":"imported-file.svelte","start":{"line":0,"character":37},"end":{"line":0,"character":51},"message":"Component has unused export property 'prop'. If it is for external reference only, please consider using \`export const prop\`","code":"unused-export-let","source":"svelte"}`

출력은 검사 중 발견된 총 파일 수, 오류 수, 경고 수를 요약하는 `COMPLETED` 메시지로 끝납니다. 예:
`1590680326807 COMPLETED 20 FILES 21 ERRORS 1 WARNINGS 3 FILES_WITH_PROBLEMS`

애플리케이션 실행 중 오류가 발생하면 `FAILURE` 레코드로 표시됩니다. 예:
`1590680328921 FAILURE "Connection closed"`

**만든 사람들**
`svelte-check`의 기초를 닦은 Vue의 VTI에 감사를.

**FAQ**
**Q: 왜 특정 파일만 검사하는 옵션(예: 스테이징된 파일만)이 없나요?**
A: `svelte-check`는 검사가 유효하려면 프로젝트 전체를 '봐야' 합니다. 예를 들어 컴포넌트 prop 이름을 바꿨는데 그 prop을 사용하는 곳들을 업데이트하지 않았다고 가정해봅시다. 그 사용처들은 전부 오류인데, 변경된 파일만 검사하면 이런 오류들을 놓치게 됩니다. "숲을 봐야 나무도 제대로 보이지!"

---

**얘 뭐 하는 애냐? (기능 및 목적)**
`sv check`는 SvelteKit 프로젝트의 코드 품질 관리반장입니다. 안 쓰는 CSS 찾아내서 "이거 왜 안 써? 버려!" 하거나, 접근성 문제 있으면 "이러면 안 되지!" 하고 지적하고, 자바스크립트/타입스크립트 코드에 문법 오류나 타입 안 맞는 거 있으면 "야, 이거 틀렸잖아!" 하고 잡아냅니다. 한마디로 "네 코드, 문제없는지 내가 싹 다 훑어주마!" 하는 역할이죠.

**왜 쓰는데?**
1.  **버그 사전 차단**: 컴파일 오류, 타입 오류 등을 미리 발견해서 런타임에 터질 폭탄을 줄여줍니다. "미래의 나에게 욕먹기 싫으면 지금 고쳐라."
2.  **코드 품질 향상**: 안 쓰는 코드, 접근성 미흡한 부분 등을 알려줘서 더 깔끔하고 견고한 코드를 만들도록 유도합니다. "보기 좋은 코드가 디버깅하기도 좋다."
3.  **협업 효율 증대**: 팀 프로젝트에서 일관된 코드 스타일과 품질을 유지하는 데 도움을 줍니다. CI/CD 파이프라인에 연동해서 커밋 전에 자동으로 검사 돌리면 금상첨화. "내 코드는 내가 지킨다! (팀 코드도 같이)"

**언제 불려 나오냐?**
코드 작성 중이나 커밋하기 전에, 또는 CI/CD 과정에서 프로젝트 전체 코드의 건강 상태를 점검하고 싶을 때 터미널에 `npx sv check`를 입력해서 실행합니다.

**쓸 때 꿀팁 및 주의사항:**
*   **`svelte-check`는 개발 의존성으로**: `npm i -D svelte-check`로 설치하는 거 잊지 마세요. 빌드 결과물에는 필요 없는 녀석입니다.
*   **`--watch` 모드는 개발 중 찰떡궁합**: 코드 수정할 때마다 실시간으로 피드백 받으니 편합니다. "저장하는 순간, 검사 시작!"
*   **`tsconfig.json` 활용**: 타입스크립트 쓴다면 `tsconfig.json` 설정을 잘 활용해야 정확한 검사가 가능합니다. `--no-tsconfig`는 정말 Svelte 파일만 볼 때나 쓰는 옵션.
*   **`--fail-on-warnings`는 엄격 모드**: 경고도 오류처럼 처리해서 CI에서 빌드를 실패시키고 싶을 때 유용합니다. "작은 실수도 용납하지 않겠다!"는 의지의 표현.
*   **기계 출력은 자동화의 친구**: `--output machine` 또는 `machine-verbose`는 스크립트로 결과 파싱해서 리포트 만들거나 다른 도구랑 연동할 때 씁니다. "사람 눈보다 정확한 기계의 눈."
*   **전체 프로젝트 검사가 기본**: 특정 파일만 콕 집어 검사하는 기능이 없는 건 이유가 있습니다. 코드 변경은 연쇄 반응을 일으킬 수 있어서 전체를 봐야 정확한 진단이 가능하기 때문이죠. "부분만 보면 코끼리 다리 만지는 격."