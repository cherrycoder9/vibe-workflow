SvelteKit 어댑터는 당신의 SvelteKit 사이트를 다양한 플랫폼에 배포할 수 있게 해주는 마법 도구입니다. 이 부가 기능(`add-on`)을 사용하면 SvelteKit 공식 어댑터들을 설정할 수 있고, 커뮤니티에서 만든 어댑터들도 많으니 골라 쓰는 재미가 있죠.

**사용법**

```bash
npx sv add sveltekit-adapter
```

**뭘 얻을 수 있나?**

*   선택한 SvelteKit 어댑터가 설치되고, `svelte.config.js` 파일에 설정까지 알아서 해줍니다. "손 하나 까딱 안 해도 배포 준비 끝!"

**옵션**

*   `adapter`
    어떤 SvelteKit 어댑터를 쓸지 정합니다:
    *   `auto` — `@sveltejs/adapter-auto`는 적절한 어댑터를 자동으로 골라주지만, 세부 설정은 좀 부족합니다. "알아서 잘 딱 깔끔하게 센스있게! (근데 디테일은 묻지 마)"
    *   `node` — `@sveltejs/adapter-node`는 독립 실행형 Node.js 서버를 생성합니다. "내 서버는 내가 직접 돌린다!"
    *   `static` — `@sveltejs/adapter-static`은 SvelteKit을 정적 사이트 생성기(SSG)로 쓸 수 있게 해줍니다. "HTML, CSS, JS 파일만 딱! 서버는 거들 뿐."
    *   `vercel` — `@sveltejs/adapter-vercel`은 Vercel에 배포할 수 있게 해줍니다.
    *   `cloudflare` — `@sveltejs/adapter-cloudflare`는 Cloudflare에 배포할 수 있게 해줍니다.
    *   `netlify` — `@sveltejs/adapter-netlify`는 Netlify에 배포할 수 있게 해줍니다.

**특정 어댑터 바로 설치 예시**

```bash
npx sv add sveltekit-adapter=adapter:node
```
이 명령어는 `@sveltejs/adapter-node`를 바로 설치하고 설정합니다.

---

**얘 뭐 하는 애냐? (기능 및 목적)**
SvelteKit 어댑터는 한마디로 "변환 플러그"입니다. 우리가 SvelteKit으로 열심히 만든 웹앱을 Vercel, Netlify, Cloudflare 같은 서버리스 플랫폼이나 개인 Node.js 서버, 혹은 그냥 정적 파일 묶음으로 "번역" 또는 "패키징" 해주는 역할을 합니다. 개발할 때는 `npm run dev`로 잘 돌아가던 것도, 실제 세상(프로덕션 환경)에 내놓으려면 그 환경에 맞는 옷으로 갈아입혀야 하거든요. 어댑터가 바로 그 옷을 입혀주는 스타일리스트인 셈입니다. "개발은 자유롭게, 배포는 플랫폼 맞춤으로!"

**왜 쓰는데?**
1.  **원 소스 멀티 유즈**: SvelteKit 코드는 하나인데, 어댑터만 갈아 끼우면 다양한 플랫폼에 배포 가능. "이 코드 한 벌로 돌려입기 쌉가능."
2.  **플랫폼 최적화**: 각 플랫폼(Vercel, Netlify 등)의 특징에 맞게 빌드 결과물을 최적화해줍니다. 예를 들어 서버리스 함수 형태로 만들어주거나, 엣지 네트워크에 맞게 조정해주는 식이죠. "그 집 음식은 그 집 그릇에 담아야 제맛."
3.  **빌드 과정 자동화**: `svelte.config.js`에 어댑터 설정해두고 `npm run build` 명령어만 치면, 해당 플랫폼에 맞는 빌드 결과물이 뿅 하고 나옵니다. 복잡한 배포 스크립트 짤 필요 없어요. "버튼 하나로 배포 준비 완료!"

**언제 불려 나오냐?**
SvelteKit으로 만든 프로젝트를 로컬 개발 환경이 아닌, 실제 서비스할 서버나 플랫폼에 올리려고 할 때 `npx sv add sveltekit-adapter` 명령어로 원하는 어댑터를 설치하고 설정합니다. 보통 프로젝트 개발 마무리 단계나 배포 환경을 결정했을 때 사용하죠.

**쓸 때 꿀팁 및 주의사항:**
*   **`adapter-auto`는 만능이 아니다**: `@sveltejs/adapter-auto`는 Vercel, Netlify, Cloudflare Pages, GitHub Pages 같은 유명 플랫폼을 감지해서 자동으로 설정해주니 편하긴 합니다. 하지만 지원 안 하는 플랫폼이거나, 특정 설정을 만져야 한다면 해당 플랫폼 전용 어댑터(예: `adapter-node`, `adapter-static`)를 써야 합니다. "자동 모드는 편하지만, 수동 모드가 더 강력하다."
*   **`adapter-static`의 함정**: 정적 사이트로 만들면 서버 사이드 렌더링(SSR)의 이점을 못 누릴 수 있습니다. 모든 페이지를 미리 HTML로 구워버리니까요. 동적인 데이터가 거의 없고, SEO가 중요하며, CDN으로 빠르게 뿌리고 싶을 때 좋습니다. API 라우트 같은 서버 기능도 당연히 못 씁니다. "모든 걸 다 가질 순 없어."
*   **`adapter-node`는 자유, 그러나 책임도 따른다**: Node.js 서버로 직접 돌리려면 서버 관리, 보안, 스케일링 등을 직접 챙겨야 합니다. Docker랑 같이 쓰면 좀 편해지긴 합니다. "자유에는 대가가 따른다."
*   **플랫폼별 환경 변수**: Vercel, Netlify 등에 배포할 때는 해당 플랫폼에서 제공하는 환경 변수 설정 기능을 잘 활용해야 API 키 같은 민감 정보를 안전하게 관리할 수 있습니다. 어댑터 자체의 옵션으로 해결 안 되는 경우가 많습니다.
*   **`svelte.config.js` 확인은 필수**: 어댑터를 추가/변경하면 `svelte.config.js` 파일에 `import adapter from '@sveltejs/adapter-xxx';` 와 `kit: { adapter: adapter() }` 부분이 잘 바뀌었는지 확인하세요. 가끔 꼬일 때도 있습니다. "설정 파일은 내 친구."
*   **커뮤니티 어댑터도 많다**: 공식 어댑터 외에도 AWS Amplify, Firebase, Deno Deploy 등 다양한 플랫폼을 위한 커뮤니티 어댑터들이 있으니, 필요한 게 있다면 찾아보는 것도 좋습니다. (예: `svelte-adapter-firebase`) "없으면 만들면 된다... 가 아니라 찾아보면 이미 있을지도?"