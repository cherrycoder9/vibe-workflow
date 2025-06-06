# Markdown 코딩 가이드라인 (최종 수정판)

이 문서는 Markdown(마크다운) 작성 표준 및 컨벤션을 정의합니다. 모든 Markdown 코드는 이 가이드라인을 엄격히 준수하여 가독성과 일관성을 높이는 것을 목표로 합니다.

## A. Markdown 공백 및 들여쓰기: 가독성의 숨은 공신

Markdown에서 공백(whitespace)과 들여쓰기(indentation)는 눈에 잘 띄지 않지만, 문서의 구조와 렌더링에 결정적인 영향을 미칩니다. `markdownlint`와 같은 도구를 활용하여 일반적인 오류를 사전에 방지하는 것이 좋습니다.

### 1. 빈 줄의 올바른 사용 (MD012, MD022, MD031, MD032)

- **핵심**: 문단, 제목, 목록, 코드 블록 등 Markdown의 여러 블록 요소들은 서로 구분되기 위해 주변에 빈 줄이 필요합니다.
  - 문단과 문단 사이에는 반드시 **하나의 빈 줄**을 사용합니다.
  - 제목, 목록, 코드 블록 앞뒤에도 빈 줄을 삽입하여 렌더링 오류를 방지하고 가독성을 높입니다.
  - 연속된 여러 빈 줄은 피합니다.
- **`markdownlint` 규칙**:
  - `MD012 (no-multiple-blanks)`: 연속된 여러 빈 줄을 하나로 제한합니다.
  - `MD022 (blanks-around-headings)`: 제목 위아래로 빈 줄을 요구합니다.
  - `MD031 (blanks-around-fences)`: 구분된 코드 블록(fenced code block) 주변에 빈 줄을 요구합니다.
  - `MD032 (blanks-around-lists)`: 목록 주변에 빈 줄을 요구합니다.
- **중요성**: 많은 Markdown 렌더러가 빈 줄을 기준으로 블록을 인식하므로, 빈 줄 누락은 가장 흔한 실수 중 하나입니다.

### 2. 후행 공백 및 불필요한 선행 공백 (MD009)

- **핵심**: 줄 끝에 불필요한 공백(후행 공백)을 남기거나, 문단 시작 부분에 의도치 않은 공백이나 탭을 삽입하는 것을 피해야 합니다.
  - 후행 공백은 때로 줄 바꿈으로 해석될 수 있으며(두 칸 이상 공백 시), 대부분 불필요하고 버전 관리 시스템(예: Git)에서 불필요한 변경으로 기록될 수 있습니다.
  - 문단 정렬을 위해 문단 앞에 공백이나 탭을 사용하면 코드 블록으로 잘못 해석될 수 있습니다.
- **`markdownlint` 규칙**:
  - `MD009 (no-trailing-spaces)`: 후행 공백을 금지합니다. (단, 의도적인 줄 바꿈을 위한 후행 공백은 `br_spaces` 파라미터로 허용 가능)
- **중요성**: 예기치 않은 서식 문제를 유발할 수 있습니다.

### 3. 탭(Hard tabs) 대 공백(Soft tabs) (MD010)

- **핵심**: 들여쓰기나 정렬을 위해 탭 문자 대신 **공백 문자**를 사용하는 것이 일관성과 호환성 측면에서 권장됩니다. 탭 문자는 편집기나 환경에 따라 다르게 표시될 수 있습니다.
- **`markdownlint` 규칙**:
  - `MD010 (no-hard-tabs)`: 하드 탭 사용을 금지합니다.
- **중요성**: 일관된 문서 표현을 위해 공백 사용을 권장합니다.

### 4. 목록 들여쓰기 일관성 (MD005, MD007)

- **핵심**: 중첩된 목록을 사용할 때, 각 레벨의 들여쓰기는 일관되어야 합니다.
  - CommonMark 명세는 일반적으로 중첩 목록에 4칸 공백 들여쓰기를 권장하지만, 2칸 공백도 널리 사용됩니다. 중요한 것은 **일관성**입니다.
  - 같은 레벨의 목록 항목인데 들여쓰기가 다르거나, 중첩 시 들여쓰기 칸 수가 일정하지 않은 경우를 피해야 합니다.
- **`markdownlint` 규칙**:
  - `MD005 (list-indent)`: 같은 수준의 목록 항목에 대해 일관되지 않은 들여쓰기를 검사합니다.
  - `MD007 (ul-indent)`: 순서 없는 목록의 들여쓰기 칸 수를 지정합니다 (기본 2칸, 프로젝트별 표준 설정 가능).
- **중요성**: 편집기 및 플랫폼 간 호환성 문제를 줄이고, 팀의 표준을 정하는 것이 중요합니다.

## B. Markdown 구조적 요소: 문서의 뼈대 바로 세우기

제목, 목록, 코드 블록과 같은 구조적 요소는 문서의 가독성과 의미 전달에 핵심적인 역할을 합니다.

### 1. 제목: 레벨, 구두점, 마크업 (MD001, MD003, MD026, MD041)

- **핵심**:
  - 제목은 `#`의 개수로 수준을 표현하며, **순차적으로 증가**해야 합니다 (예: `<h1>` 다음 `<h2>`, `<h2>` 다음 `<h3>`). `<h1>` 다음 `<h3>`와 같이 단계를 건너뛰지 않습니다.
  - 제목 끝에 구두점(`.`, `?`, `!` 등)을 사용하지 않습니다.
  - 제목 내부에 다른 마크업(굵게 `**`, 기울임 `*` 등)을 사용하는 것은 피해야 합니다.
  - 문서의 첫 줄은 최상위 제목(H1, `#`)으로 시작하는 것이 좋습니다.
  - ATX 스타일 제목 (`# 제목`) 사용 시 `#` 뒤에는 반드시 공백이 하나 있어야 합니다.
- **`markdownlint` 규칙**:
  - `MD001 (heading-increment)`: 제목 수준은 한 번에 한 단계씩만 증가해야 합니다.
  - `MD003 (heading-style)`: 일관된 제목 스타일을 강제합니다 (예: ATX 스타일 `#`).
  - `MD026 (no-trailing-punctuation)`: 제목 끝 구두점을 금지합니다.
  - `MD041 (first-line-heading/first-line-h1)`: 파일의 첫 번째 줄이 최상위 제목이어야 합니다.
- **중요성**: 잘 작성된 제목은 문서 구조를 명확히 하고 탐색에 도움을 줍니다. 제목에 특수문자 사용 시 앵커 링크 생성에 문제가 발생할 수 있습니다.

### 2. 목록: 순서 및 마커 일관성 (MD004, MD029)

- **핵심**:
  - **순서 있는 목록**: 모든 항목을 `1.`로 시작해도 렌더링 시 자동으로 번호가 매겨집니다. 이는 항목 추가, 삭제, 재정렬 시 수동으로 번호를 수정할 필요가 없어 편리합니다.
  - **순서 없는 목록**: 일관된 마커(예: `-`, `*`, `+` 중 하나)를 사용해야 합니다. 한 목록 내에서 여러 마커를 혼용하지 않습니다.
  - 목록 항목 마커(예: `1.`, `-`) 뒤에는 반드시 공백이 하나 있어야 합니다.
- **`markdownlint` 규칙**:
  - `MD004 (ul-style)`: 순서 없는 목록의 마커 스타일 일관성을 강제합니다.
  - `MD029 (ol-prefix)`: 순서 있는 목록 항목 접두사 스타일을 확인합니다 (예: `one`은 `1.`, `ordered`는 순차적 번호). `one` 사용을 권장합니다.
- **중요성**: 일관된 목록 스타일은 가독성을 높이고, 순서 있는 목록에 `1.`을 사용하는 것은 유지보수를 용이하게 합니다.

### 3. 코드 블록: 구분 및 명시 (MD031, MD040, MD046, MD048)

- **핵심**:
  - **구분된 코드 블록 (Fenced Code Blocks)**: ` ``` (백틱 3개) 또는 `~~~` (물결표 3개)를 사용하여 코드 블록을 감쌉니다.
    - 코드 블록 위아래로 **빈 줄**을 추가하여 다른 콘텐츠와 명확히 구분합니다.
    - 구문 강조(syntax highlighting)를 위해 코드 블록 시작 부분에 **언어 지정자**를 명시하는 것이 좋습니다 (예: ` ```javascript`).
  - **코드 블록 내 백틱**: 코드 블록 자체를 백틱으로 감싸는 경우, 코드 내용에 백틱을 표시하려면 바깥쪽보다 더 많은 수의 백틱을 사용하거나, 물결표 구분자를 사용합니다. (예: 코드 내부에 ` ```를 표현하려면 바깥쪽을 ` ```` (백틱 4개)로 감싸거나 `~~~~` 사용)
  - **들여쓰기 코드 블록**: 4칸 이상 들여쓰기 된 텍스트는 코드 블록으로 간주됩니다. 일관성을 위해 구분된 코드 블록 사용을 권장합니다.
- **`markdownlint` 규칙**:
  - `MD031 (blanks-around-fences)`: 구분된 코드 블록 주변 빈 줄을 요구합니다.
  - `MD040 (fenced-code-language)`: 구분된 코드 블록에 언어 지정자를 요구합니다.
  - `MD046 (code-block-style)`: 코드 블록 스타일 일관성(예: `fenced` 또는 `indented`)을 확인합니다. `fenced` 권장.
  - `MD048 (code-fence-style)`: 코드 구분자 스타일(백틱 ` ``` 또는 물결표 `~~~`) 일관성을 확인합니다.
- **중요성**: 명확한 코드 표현과 가독성 향상, 정확한 구문 강조를 위해 중요합니다.

## C. 콘텐츠 및 링크: 명확성과 정확성이 핵심

문서의 내용은 명확해야 하며, 참조되는 링크는 정확하고 의미 있는 방식으로 제공되어야 합니다.

### 1. 일반 URL(Bare URL) 대 적절한 링크 (MD034)

- **핵심**: 단순히 URL을 텍스트로 붙여넣기보다는, 의미 있는 텍스트에 하이퍼링크를 거는 것이 좋습니다.
  - 예: `https://www.example.com` (X) → `[예제 사이트](https://www.example.com)` (O)
  - 일부 파서는 일반 URL을 자동으로 링크로 변환하지 않을 수 있습니다.
- **`markdownlint` 규칙**:
  - `MD034 (no-bare-urls)`: 일반 URL 사용을 금지하고 `<URL>` 또는 `[text](URL)` 형식을 권장합니다.
- **중요성**: 의미 있는 앵커 텍스트 사용은 사용자 경험과 검색 엔진 최적화(SEO) 모두에 긍정적인 영향을 미칩니다.

### 2. 이미지 대체 텍스트(alt text) 및 설명적인 링크 (MD045, MD059)

- **핵심**:
  - **이미지**: 모든 이미지에는 이미지를 설명하는 **대체 텍스트(alt text)**를 제공해야 합니다.
    - 예: `![유용한 마크다운 로고](logo.png)` (O) / `![](logo.png)` (X)
    - "이미지", "사진"과 같이 무의미하거나 중복되는 대체 텍스트는 피합니다.
  - **링크**: 링크 텍스트는 링크가 가리키는 대상을 명확히 설명해야 합니다.
    - 예: `[더 알아보기](details.md)` (X) → `[마크다운 상세 가이드라인](details.md)` (O)
- **`markdownlint` 규칙**:
  - `MD045 (no-alt-text)`: 이미지에 대체 텍스트가 없는 경우 경고합니다.
  - `MD059 (descriptive-link-text)`: 링크 텍스트가 설명적인지 확인합니다.
- **중요성**: 대체 텍스트는 스크린 리더 사용자 등 접근성을 향상시키고 SEO에도 중요합니다. 설명적인 링크는 사용자에게 명확한 정보를 제공합니다.

### 3. 마크다운으로 대체 가능한 인라인 HTML 회피 (MD033)

- **핵심**: 간단한 서식(줄 바꿈, 강조 등)을 위해 HTML 태그(` <br> `, ` <b> ` 등)를 사용하는 것보다 가능한 **마크다운 고유 문법**을 사용하는 것이 좋습니다.
  - 예: `<br>` 대신 문단 사이 빈 줄 또는 줄 끝 공백 두 칸 사용.
  - 이미지 크기 조절 등 마크다운으로 제어가 어려운 경우에 한해 제한적으로 HTML 사용을 고려할 수 있으나, 기본적으로는 마크다운 문법을 우선합니다.
- **`markdownlint` 규칙**:
  - `MD033 (no-inline-html)`: 인라인 HTML 사용을 기본적으로 금지합니다. `allowed_elements` 파라미터를 통해 특정 태그를 허용할 수 있습니다.
- **중요성**: 마크다운 고유 문법 사용은 문서의 순수성과 이식성을 높입니다. HTML 혼용은 파서에 따라 예기치 않은 결과를 초래할 수 있습니다.

## D. 테이블: 까다로운 구문과의 씨름

Markdown 테이블은 정보를 구조화하여 보여주는 강력한 방법이지만, 구문이 다소 까다로워 실수가 잦을 수 있습니다.

### 1. 일반적인 구문 오류 (파이프, 하이픈, 빈 줄)

- **핵심**: 마크다운 테이블은 파이프(`|`)와 하이픈(`-`)을 사용하여 구조를 만듭니다. 헤더와 본문 구분, 각 열 구분, 정렬 지정 시 정확한 구문이 필요합니다.
  - 헤더와 구분선(hyphen row) 사이에는 빈 줄이 없어야 합니다.
  - 구분선의 하이픈 개수는 각 열의 내용과 정렬 표시자(`:`)를 포함하여 충분해야 합니다 (최소 3개 권장).
  - 테이블 시작과 끝에 파이프(`|`)를 사용하는 것이 호환성 측면에서 권장됩니다.
  - 테이블 위아래로 빈 줄을 넣어 다른 요소와 명확히 구분합니다.
- **흔한 실수**:
  - 헤더와 구분선 사이에 빈 줄 삽입.
  - 구분선의 하이픈 개수 부족 또는 파이프와 하이픈 사이 공백 누락 (일부 파서에서 문제 발생 가능).
  - 테이블 시작/끝 파이프 생략.
  - 테이블 위아래 빈 줄 누락.
- **`markdownlint` 규칙**:
  - `MD055 (table-pipe-style)`: 테이블 파이프 스타일을 확인합니다. (일관된 파이프 사용 권장)
  - `MD056 (table-column-count)`: 테이블 열 개수 일관성을 확인합니다.
  - `MD058 (blanks-around-tables)`: 테이블 주변 빈 줄을 요구합니다. (MD032와 유사)
- **중요성**: 테이블 구문은 정확성이 중요하며, 사소한 오류가 테이블 렌더링 실패로 이어질 수 있습니다. 테이블 생성기 사용도 좋은 대안이 될 수 있습니다. 가독성을 위해 셀 정렬에 충분한 공백을 사용하는 것이 좋습니다.

## E. VSCode `markdownlint` 효과적으로 활용하기

VSCode 환경에서 `markdownlint` 익스텐션을 올바르게 이해하고 설정하는 것은 Markdown 문서의 품질을 일관되게 유지하고 팀 전체의 생산성을 향상시키는 데 매우 중요합니다.

### 1. 주요 `markdownlint` 규칙 심층 분석

엔지니어들이 자주 위반하거나 그 의미를 오해하기 쉬운 `markdownlint` 규칙들을 선별하여, 각 규칙의 목적, 위반 사례, 올바른 사용법, 그리고 관련 파라미터를 예시와 함께 상세히 설명합니다.

#### 가. MD007 (ul-indent - 순서 없는 목록 들여쓰기)

이 규칙은 순서 없는 목록 항목의 들여쓰기 스타일을 일관되게 유지하는 데 목적이 있습니다.

- **주요 파라미터**:
  - `indent` (기본값: 2): 중첩 시 각 레벨당 들여쓰기할 공백 수를 지정합니다.
  - `start_indented` (기본값: false): `true`로 설정하면 문서의 첫 번째 레벨 목록도 `start_indent`만큼 들여쓰기됩니다.
  - `start_indent` (기본값: 2): `start_indented`가 `true`일 때 첫 번째 레벨 목록의 들여쓰기 칸 수입니다.
- **설명 및 권장 사항**:
  - 일반적으로 2칸 들여쓰기는 부모 목록의 내용 시작점과 정렬되어 가독성이 좋다고 평가됩니다.
  - 4칸 들여쓰기는 코드 블록 들여쓰기(일반적으로 4칸)와 일관성을 가지며, 일부 Markdown 파서(특히 MultiMarkdown)와의 호환성을 높일 수 있습니다.
  - 팀의 표준이나 사용하는 파서의 특성에 맞춰 `indent` 값을 2 또는 4로 설정하는 것이 좋습니다.
- **주의할 점**:
  - MD007은 순서 있는 목록 내에 중첩된 순서 없는 목록에는 적용되지만, 그 반대의 경우(순서 없는 목록 내 순서 있는 목록)에는 직접 적용되지 않을 수 있습니다.
  - `start_indented: true` 설정과 `indent` 값을 0보다 크게 설정했을 때, 블록쿼트 내에서 MD027 (no-multiple-space-blockquote) 규칙과 충돌할 가능성이 보고된 바 있습니다.

#### 나. MD013 (line-length - 줄 길이)

이 규칙은 한 줄의 최대 길이를 제한하여 가독성을 높이고, 특히 버전 관리 시스템에서 변경 사항을 추적하기 용이하게 합니다.

- **주요 파라미터**:
  - `line_length` (기본값: 80): 일반 텍스트 줄의 최대 길이입니다.
  - `heading_line_length` (기본값: 80): 제목 줄의 최대 길이입니다.
  - `code_block_line_length` (기본값: 80): 코드 블록 내 줄의 최대 길이입니다.
  - `code_blocks` (기본값: true): 코드 블록에 규칙 적용 여부입니다.
  - `tables` (기본값: true): 테이블에 규칙 적용 여부입니다.
  - `headings` (기본값: true): 제목에 규칙 적용 여부입니다.
  - `strict` (기본값: false): `true`로 설정 시, 공백 없이 길게 이어지는 URL 등도 위반으로 처리합니다.
  - `stern` (기본값: false): `true`로 설정 시, 수정 가능한(즉, 공백이 포함된) 긴 줄에 대해서만 경고합니다.
- **설명 및 권장 사항**:
  - VSCode의 `markdownlint` 익스텐션은 기본적으로 MD013을 비활성화하는 경우가 많은데, 이는 많은 파일이 관습적인 80자 제한을 초과하기 때문입니다.
  - 팀 표준에 따라 적절한 길이(예: 100자 또는 120자)를 설정하고, 코드 블록이나 테이블처럼 줄 바꿈이 어려운 요소는 `code_blocks: false`, `tables: false` 등으로 예외 처리하는 것이 현실적인 접근 방식입니다.
  - 긴 URL이나 이미지 참조 정의는 기본적으로 이 규칙에서 예외 처리됩니다.
- **주의할 점**:
  - 한글이나 이모지와 같은 다중 바이트 유니코드 문자가 포함된 줄의 길이를 계산할 때 부정확할 수 있다는 점이 보고된 바 있습니다.

#### 다. MD029 (ol-prefix - 순서 있는 목록 접두사)

이 규칙은 순서 있는 목록의 숫자 접두사 스타일을 일관되게 유지하도록 합니다.

- **주요 파라미터**:
  - `style` (기본값: "one_or_ordered"): 목록 접두사 스타일을 지정합니다.
    - `"one"`: 모든 목록 항목을 `1.`로 시작합니다. (예: `1. 첫째`, `1. 둘째`)
    - `"ordered"`: 숫자가 순차적으로 증가합니다. (예: `1. 첫째`, `2. 둘째`)
    - `"one_or_ordered"`: 위 두 가지 스타일을 모두 허용합니다.
    - `"zero"`: 목록이 `0.`으로 시작하고 순차적으로 증가합니다. (예: `0. 영점`, `1. 일점`)
- **설명 및 권장 사항**:
  - 많은 가이드에서 `style: "one"`을 권장합니다. 이는 목록 항목을 추가, 삭제, 재정렬할 때 번호를 일일이 수정할 필요가 없어 유지보수성이 크게 향상되기 때문입니다.
- **주의할 점**:
  - 목록 내에 코드 블록이나 다른 블록 요소가 잘못 들여쓰기되어 목록의 연속성이 깨지면, MD029 위반으로 이어질 수 있으므로 주의해야 합니다.

#### 라. MD033 (no-inline-html - 인라인 HTML)

Markdown 문서의 순수성과 이식성을 위해 인라인 HTML 사용을 제한하는 규칙입니다.

- **주요 파라미터**:
  - `allowed_elements` (기본값: 매우 제한적, 주석 태그 등만 허용): 허용할 HTML 태그 이름들을 문자열 배열 형태로 명시하여 예외를 둘 수 있습니다.
- **설명 및 권장 사항**:
  - 예를 들어, 줄 바꿈을 위해 `<br>` 태그를 사용하거나, GitHub 등에서 접고 펴는 섹션(collapsible sections)을 만들기 위해 `<details>`와 `<summary>` 태그를 사용하는 경우, 이들을 `allowed_elements`에 추가해야 합니다.
  - 이미지 크기 조절이나 복잡한 테이블 셀 서식 등 Markdown만으로는 표현이 어려운 경우에도 HTML이 필요할 수 있습니다.
- **주의할 점**:
  - 현재 `allowed_elements`는 태그 이름 수준에서만 허용 여부를 제어하며, 특정 속성을 가진 경우에만 허용하는 등의 세밀한 제어는 기본적으로 지원하지 않습니다.

#### 마. MD044 (proper-names - 적절한 이름 대소문자)

이 규칙은 기술 문서에서 제품명, 기술 용어, 회사명 등의 고유명사가 일관되고 정확한 대소문자로 사용되도록 강제합니다.

- **주요 파라미터**:
  - `names` (배열): 검사 대상이 되는 고유명사 목록입니다.
  - `code_blocks` (기본값: true): 코드 블록 내에서도 규칙 적용 여부입니다. (단, 백틱으로 감싸인 단어는 제외)
- **설명 및 권장 사항**:
  - 흔한 실수는 JavaScript를 `Javascript`로 쓰거나, NGINX를 `Nginx`로 쓰는 등 공식적인 대소문자 표기를 따르지 않는 것입니다.
  - 명령어(`git clone`), 파일명(`.gitlab-ci.yml`), 설정값 등은 백틱(\`)으로 감싸야 하는데, 이를 누락하면 MD044 위반으로 이어질 수 있습니다. (예: `.gitlab-ci.yml`을 백틱 없이 사용하면 `gitlab` 부분이 대문자가 아니라는 이유로 위반될 수 있음)
  - 이 규칙은 정확한 기술 용어 사용을 장려하여 문서의 전문성을 높이는 데 기여합니다.
- **주의할 점**:
  - 설정 파일의 `names` 파라미터를 통해 프로젝트별로 자주 사용하는 고유명사 목록을 관리해야 합니다.
  - 백틱(\`)으로 감싸인 단어는 이 규칙의 검사에서 제외됩니다.

### 2. 핵심 조언 및 설정 파일 예시

- **핵심 조언 요약**:
  - **MD007 (ul-indent)**: `indent: 4` 권장 (코드 블록과 일관성, MD027 충돌 주의).
  - **MD013 (line-length)**: `line_length: 100`, `code_blocks: false`, `tables: false` 권장 (유니코드 길이 계산 오류 가능성 인지).
  - **MD029 (ol-prefix)**: `style: "one"` 권장 (유지보수 용이).
  - **MD033 (no-inline-html)**: 필요한 HTML 태그(`br`, `details`, `img` 등)는 `allowed_elements`에 추가.
  - **MD044 (proper-names)**: 프로젝트별 고유명사 `names`에 등록, 명령어/파일명은 백틱(`) 사용.
- **설정 파일 예시 (`.markdownlint.json`)**:

    ```json
    {
      "MD007": { "indent": 4 },
      "MD013": { "line_length": 100, "code_blocks": false, "tables": false },
      "MD029": { "style": "one" },
      "MD033": { "allowed_elements": ["br", "details", "summary", "img"] },
      "MD044": { "names": ["Git", "NGINX", "Docker"], "code_blocks": true }
    }
    ```

### 3. `markdownlint` 설정 전략

`markdownlint`의 효과는 올바른 설정에서 비롯됩니다. 다양한 설정 방법과 그 우선순위, 그리고 일반적인 설정 관련 실수를 이해하는 것이 중요합니다.

#### 가. 설정 우선순위 및 방법 (1이 가장 높음)

1. **`.markdownlint-cli2.{jsonc,yaml,cjs}`**: 프로젝트 루트 또는 상위 디렉터리. `markdownlint-cli2` 주 설정 파일. 팀 전체 표준 강제에 최적.
2. **`.markdownlint.{jsonc,json,yaml,yml,cjs}`**: 프로젝트 루트 또는 상위 디렉터리. 위 파일 없을 시 사용. 일반적 형식.
3. **VSCode `settings.json` - `markdownlint.configFile`**: 작업 공간 설정에서 특정 설정 파일 경로 지정.
4. **VSCode `settings.json` - `markdownlint.config`**: 사용자 또는 작업 공간 설정에 직접 규칙 정의. 개인 선호도용.
5. **인라인 주석**: `` 등. 특정 부분 예외 처리. 남용 주의.
6. **`markdownlint` 기본값**: 별도 설정 없을 시 적용.

#### 나. 프로젝트 설정 파일 vs VSCode 설정

- **프로젝트 설정 파일 (`.markdownlint.json` 등)**: 버전 관리에 포함하여 팀 전체 일관성 유지에 효과적. **권장 방식**.
- **VSCode `settings.json`**: 개인 선호도 반영 또는 프로젝트 파일 없을 시 대안.

#### 다. 일반적인 설정 오류

- **파일명/위치 오류**: 설정 파일이 정확한 이름으로 프로젝트 루트에 있는지 확인.
- **구문 오류**: JSON/YAML 구문(쉼표, 중첩 등) 확인. `.jsonc` 사용 시 주석 가능.
- **우선순위 오해**: 프로젝트 설정 파일이 VSCode 설정을 덮어쓰는지 확인.
- **도구 버전 간 비호환성**: `markdownlint-cli2` (Node.js)와 `mdl` (Ruby) 등 도구 간 설정 호환성 주의.
- **`.editorconfig`와의 혼동**: `markdownlint`는 `.editorconfig`를 직접 읽지 않음. EditorConfig 익스텐션의 자동 수정 기능과 충돌처럼 보일 수 있음.

#### 라. 규칙 재정의 및 인라인 비활성화

- **규칙 재정의**: 설정 파일에서 규칙 ID를 키로 `false`(비활성화), `true`(기본값 활성화), 또는 파라미터 객체로 설정.

    ```json
    // .markdownlint.json 예시
    {
      "default": true, // 다른 모든 규칙은 기본값으로 활성화
      "MD033": { // 인라인 HTML 규칙 (no-inline-html)
        "allowed_elements": [ "details", "summary", "br", "img", "sup", "sub" ] // 허용할 태그 목록
      }
    }
    ```

- **인라인 비활성화**: HTML 주석 형태로 특정 줄, 섹션, 파일에 규칙 예외 처리. 남용 시 린터 효과 감소.
  - `` (현재 줄 MD001, MD005 비활성화)
  - `` (다음 줄 MD013 비활성화)
  - `...` (섹션 전체 규칙 비활성화/활성화)
  - `` (파일 전체 MD041 비활성화)
  - `` (파일 전체 MD013 파라미터 설정)

## F. `markdownlint` 문제 해결 및 심층 이해

`markdownlint` 사용 시 예상치 못한 경고나 오작동은 규칙 자체의 한계, 규칙 간 충돌, 마크다운 구조 해석 차이 등에서 비롯될 수 있습니다.

### 1. 오탐(False Positives) 및 규칙 충돌

- **중첩 컨텍스트에서의 오탐**: 목록 안의 코드 블록, 블록쿼트 안의 목록 등 복잡한 구조에서 MD022(제목 주변 빈 줄), MD031(코드 블록 주변 빈 줄), MD032(목록 주변 빈 줄) 등이 오탐을 발생시킬 수 있습니다.
  - MD022와 MD012(연속된 여러 빈 줄 금지)는 제목 위아래 빈 줄 설정에 따라 충돌처럼 보일 수 있습니다.
  - 목록 내 중첩 요소는 MD031, MD032 규칙 적용 시 복잡성을 증가시키고 오탐을 유발할 수 있습니다.
- **규칙 충돌 예시 (MD007 vs MD027)**: MD007(`ul-indent`)의 `start_indented: true` 설정이 MD027(`no-multiple-space-blockquote`)과 충돌할 수 있습니다. 블록쿼트 내 목록 들여쓰기 공백이 MD027 위반으로 간주될 수 있습니다.
- **대처 방안**: 문제 원인 파악 후, 인라인 주석으로 규칙 임시 비활성화, 규칙 설정 조정, 또는 마크다운 구조 변경을 고려합니다.

### 2. 특정 규칙 관련 문제점

- **MD013 (line-length) - 유니코드 길이 계산**: 한글, 이모티콘 등 다중 바이트 유니코드 문자 길이 계산이 부정확하여 오탐 발생 가능.
- **MD044 (proper-names) - 혼동 유발**: 고유명사 대소문자 외에 명령어/파일명 등의 백틱 사용 여부까지 검사하여 혼란 야기 가능. (예: `.gitlab-ci.yml`을 백틱 없이 쓰면 `gitlab`이 소문자라 위반될 수 있음). `names` 배열 관리 및 백틱 사용 숙지 필요.
- **MD033 (no-inline-html) - `allowed_elements` 세부 제어 부족**: 태그 이름 수준 허용만 가능. 특정 속성을 가진 태그만 허용하는 등 세밀한 제어는 현재 미지원.

### 3. 설정의 함정과 진화하는 규칙의 중요성

- **설정 자체의 오류 가능성**:
  - `markdownlint`는 다양한 환경(CLI, Ruby gem, VSCode 익스텐션)에서 사용되며, 설정 파일 형식/해석 방식에 차이가 있을 수 있습니다.
  - VSCode 내 설정 우선순위 (프로젝트 `.markdownlint.json` > 작업 공간 `settings.json` 내 `markdownlint.config` > 사용자 `settings.json` 내 `markdownlint.config`)를 이해해야 합니다.
  - `.editorconfig` 등 다른 익스텐션과의 상호작용으로 예상치 못한 동작이 발생할 수 있습니다.
- **규칙의 지속적 개선 및 변경**:
  - `markdownlint` 규칙은 고정된 것이 아니며, 커뮤니티 피드백에 따라 지속적으로 개선됩니다. (버그 수정, 새 규칙 제안, 기존 규칙 개선 등)
  - 최신 업데이트 확인 및 커뮤니티 동향 파악이 중요합니다.
  - 팀 워크플로우에 맞지 않는 규칙은 합리적으로 커스터마이징(비활성화, 파라미터 조정, 인라인 주석)하는 유연성이 필요합니다.

## G. 견고한 Markdown 작성을 위한 고급 지침

플랫폼 간 호환성, AI 도구와의 협업, 팀 내 지속 가능한 문서 관리는 중요한 고급 주제입니다.

### 1. 플랫폼 간 호환성 및 GFM(GitHub Flavored Markdown) 고려 사항

Markdown은 다양한 "맛(flavor)"이 있으며, CommonMark와 GFM이 널리 사용됩니다. GFM은 CommonMark에 유용한 확장을 추가했지만, 이 차이점을 이해하고 일관된 렌더링을 보장하는 것이 중요합니다.

#### 가. GFM vs CommonMark 주요 기능 비교 (간략)

- **테이블**: GFM 지원, CommonMark 미지원 (확장 필요).
- **작업 목록 (Task List Items)**: GFM 지원 (`- [x]`), CommonMark 미지원.
- **자동 링크 (URL, 이메일)**: GFM은 `www.example.com` 자동 링크, CommonMark는 `< >` 필요.
- **취소선 (Strikethrough)**: GFM 지원 (`~~text~~`), CommonMark 미지원.

#### 나. 일관된 렌더링 팁

- **CommonMark 우선 사용**: 가장 넓은 호환성을 위해 CommonMark 표준 준수.
- **GFM 기능 사용 시 인지**: GFM 특정 기능 사용 시 대상 환경(예: GitHub) 명확화 또는 다른 환경에서의 차이 인지.
- **HTML 사용 최소화**: 플랫폼 간 렌더링 차이 감소.
- **파서/플랫폼 문서 확인**: 특정 플랫폼의 Markdown 지원 범위 확인.
- **기타 렌더링 불일치 원인**: 브라우저 캐시, CSS 스타일링 차이 등 고려.

### 2. AI 친화적인 Markdown 작성: LLM과의 효과적인 소통

LLM이 Markdown 문서를 효과적으로 이해하도록 작성하는 것은 인간 독자의 가독성을 높이는 관행과 일치합니다.

- **명확한 콘텐츠 구조화**: 의미론적 제목 체계(H1, H2, H3 논리적 사용, MD001/MD041 준수), 관련 정보 목록화.
- **설명적인 링크/이미지 텍스트**: 모호한 링크 텍스트("클릭")나 무의미한 alt text("이미지") 지양. 링크 대상 요약/키워드 포함, 이미지 내용 정확히 설명 (MD045 준수).
- **스타일/서식 일관성 유지**: 목록 마커, 강조, 코드 블록 형식 등 일관되게 사용 (MD003, MD004, MD049, MD050 등 활용). 팀 스타일 가이드 정의/공유.

"AI 친화적인 Markdown"은 결국 잘 구조화되고 명확한 문서를 만드는 기존 모범 사례의 확장입니다.

### 3. 팀 협업을 위한 모범 사례: 지속 가능한 문서 관리

- **팀 전체 Markdown 표준 수립**:
  - `markdownlint` 설정 파일(`.markdownlint.json` 등)을 프로젝트에 포함하여 규칙 공유.
  - 팀 합의 하에 규칙셋 결정 및 커스터마이징. 설정 파일 버전 관리.
- **CI/CD 파이프라인에서 린터 사용**:
  - PR/MR 시 변경된 Markdown 자동 검증 (GitHub Actions, GitLab CI 등에서 `markdownlint-cli` 사용).
  - 스타일 위반 및 잠재적 오류 병합 방지.
- **문서 정기 검토 및 업데이트**:
  - 문서도 코드처럼 "살아있는 존재"로 취급. 정기적 검토/업데이트로 정확성/유용성 유지.
  - 변경 사항 동료 검토. "문서 부채" 추적. "마지막 업데이트" 날짜 명시.
