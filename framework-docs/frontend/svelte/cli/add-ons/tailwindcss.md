Tailwind CSS는 HTML을 떠나지 않고도 현대적인 웹사이트를 빠르게 구축할 수 있게 해줍니다. "HTML에 클래스만 덕지덕지 붙이면 디자인이 뿅!"

**사용법**

```bash
npx sv add tailwindcss
```

**얻는 것**

*   SvelteKit용 Tailwind 가이드에 따른 Tailwind 설정
*   Tailwind Vite 플러그인
*   업데이트된 `app.css` 및 `+layout.svelte` (SvelteKit용) 또는 `App.svelte` (SvelteKit이 아닌 Vite 앱용)
*   Prettier를 사용 중이라면 Prettier와의 통합

**옵션**

*   `plugins`
    어떤 플러그인을 사용할지 정합니다:
    *   `typography` — `@tailwindcss/typography` (블로그 글처럼 긴 글 예쁘게 만들어주는 마법)
    *   `forms` — `@tailwindcss/forms` (기본 폼 요소들 좀 더 보기 좋게 리셋)

    예시:
    ```bash
    npx sv add tailwindcss="plugins:typography"
    ```

---

**얘 뭐 하는 애냐? (기능 및 목적)**
`sv add tailwindcss`는 이미 만들어진 SvelteKit 프로젝트에 Tailwind CSS를 손쉽게 설치하고 설정해주는 명령어입니다. Tailwind CSS는 미리 정의된 유틸리티 클래스들을 HTML에 직접 적용해서 스타일링하는 방식인데, 이걸 SvelteKit 프로젝트에 맞게 세팅해주는 거죠. "CSS 파일 열기 귀찮지? HTML에서 다 해결해!"가 이 녀석의 모토입니다.

**왜 쓰는데?**
1.  **간편한 설치**: Tailwind CSS 설치하고, `tailwind.config.js` 만들고, `postcss.config.js` 설정하고, 전역 CSS에 `@tailwind` 지시어 추가하고... 이 모든 귀찮은 과정을 명령어 한 줄로 끝내줍니다. "복잡한 건 얘가 다 할게, 넌 코딩만 해."
2.  **SvelteKit 최적화**: SvelteKit 프로젝트 구조에 맞게 필요한 파일들을 알아서 수정해줍니다. (`app.css`, `+layout.svelte` 등) "SvelteKit이랑 Tailwind는 원래 한 몸이었던 것처럼!"
3.  **추가 플러그인도 쉽게**: `@tailwindcss/typography` (글꼴, 자간 등 예쁘게)나 `@tailwindcss/forms` (폼 요소 기본 스타일 개선) 같은 공식 플러그인도 옵션으로 간단하게 추가할 수 있습니다. "이것도 필요할 것 같아서 준비했어."
4.  **Prettier 연동 (선택 사항)**: 만약 프로젝트에 Prettier(코드 자동 정렬 도구)가 이미 설치되어 있다면, Tailwind CSS 클래스 순서도 예쁘게 정렬해주는 `prettier-plugin-tailwindcss`까지 알아서 설정해줍니다. "개발자 경험? 그런 것도 챙겨야지."

**언제 불려 나오냐?**
SvelteKit 프로젝트를 `sv create`로 만들고 나서, "이제 Tailwind CSS로 디자인 좀 해볼까?" 싶을 때 프로젝트 루트 디렉토리에서 `npx sv add tailwindcss`를 실행합니다.

**쓸 때 꿀팁 및 주의사항:**
*   **프로젝트 생성 후 실행**: `sv create`로 프로젝트를 먼저 만들고, 그 다음에 `sv add tailwindcss`를 실행해야 합니다. 빈 땅에 집부터 지을 순 없잖아요?
*   **옵션 문법 숙지**: 플러그인 추가할 때 `tailwindcss="plugins:typography,forms"`처럼 쉼표로 여러 개 지정 가능합니다. 따옴표로 감싸는 거 잊지 마세요. "옵션질도 똑똑하게."
*   **`tailwind.config.js`는 너의 것**: 설치 후 생성되는 `tailwind.config.js` 파일에서 커스텀 색상, 폰트, 간격 등을 자유롭게 설정할 수 있습니다. 이 파일을 잘 만져야 Tailwind CSS를 제대로 활용하는 겁니다. "설정 파일은 개발자의 놀이터."
*   **유틸리티 우선(Utility-First) 개념 이해**: Tailwind CSS는 `mt-4` (margin-top: 1rem), `text-red-500` (color: #ef4444) 처럼 작은 단위의 스타일을 클래스로 조합해서 씁니다. 처음엔 "클래스 이름이 너무 많아!" 싶겠지만, 익숙해지면 CSS 파일 뒤적거릴 일 없이 HTML 안에서 빠르게 디자인할 수 있습니다. "레고 블록처럼 조립하는 재미!"
*   **PurgeCSS 자동 적용**: 빌드 시 사용하지 않는 Tailwind 클래스들은 알아서 제거해줘서 최종 CSS 파일 크기를 줄여줍니다. 걱정 말고 필요한 클래스 다 쓰세요. (Vite 플러그인이 이 역할을 합니다) "안 쓰는 건 버리는 게 인지상정."