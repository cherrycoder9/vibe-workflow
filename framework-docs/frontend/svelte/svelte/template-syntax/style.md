`style:` 디렉티브는 HTML 엘리먼트에 여러 스타일을 한 방에, 좀 더 편하게 먹이는 방법이야. CSS 인라인 스타일을 좀 더 Svelte스럽게 쓰는 거지.

```svelte
<!-- 이거 두 개 똑같은 거임 -->
<div style:color="red">...</div>
<div style="color: red;">...</div>
```

값으로는 아무 표현식이나 다 때려 박을 수 있어:

```svelte
<div style:color={myColor}>...</div>
```

줄임말도 가능 (스크립트에 `color` 변수가 선언되어 있다고 가정):

```svelte
<div style:color>...</div>
```

한 엘리먼트에 여러 스타일 한꺼번에 박을 수 있음:

```svelte
<div style:color style:width="12rem" style:background-color={darkMode ? 'black' : 'white'}>...</div>
```

스타일 뒤에 `!important` 박고 싶으면 `|important` 수식어를 붙이면 됨:

```svelte
<div style:color|important="red">...</div>
```

`style:` 디렉티브랑 그냥 `style` 속성이랑 같이 쓰면, `style:` 디렉티브가 이김. 형님이 먼저다 이거지:

```svelte
<div style="color: blue;" style:color="red">이건 빨간색으로 나옴</div>
```

---

**얘 뭐 하는 애냐? (기능 및 목적)**
`style:` 디렉티브는 Svelte에서 HTML 태그에 직접 스타일 먹일 때 쓰는 문법 설탕(syntactic sugar)이야. 그냥 `style="속성:값; 속성:값;"` 이렇게 줄줄이 쓰는 것보다 훨씬 깔끔하고, 자바스크립트 변수랑 연동하기도 편하게 해줘. "CSS 직접 박을 때 좀 더 있어 보이게, 편하게 하자!" 이거지. 가독성도 올리고, 동적 스타일링도 쉽게 만들어주는 역할이야.

**왜 쓰는데?**
1.  **가독성 UP**: `style="font-size: 16px; color: blue; background-color: #eee;"` 이런 거보다 `style:font-size="16px" style:color="blue" style:background-color="#eee"` 이게 훨씬 보기 좋잖아? "코드는 얼굴이다!"
2.  **자바스크립트 변수랑 찰떡궁합**: `<div style:color={myVariable}>` 이런 식으로 변수 값을 바로 스타일로 연결 가능. 상태에 따라 동적으로 스타일 바꿀 때 개꿀. "데이터 바뀌면 바로바로 반영해!"
3.  **조건부 스타일링 용이**: `{darkMode ? 'black' : 'white'}` 같은 삼항연산자도 바로 써서 조건에 따라 다른 스타일 적용. "어두우면 검게, 밝으면 희게!"
4.  **`!important`도 간편하게**: `style:color|important="red"` 이렇게 `|important`만 붙이면 끝. "내 말이 법이다! 무조건 이 스타일 써!"

**언제 불려 나오냐?**
컴포넌트 템플릿(`.svelte` 파일) 안에서 특정 HTML 엘리먼트에 직접 인라인 스타일을 먹이고 싶을 때, 특히 그 스타일 값이 동적이거나 여러 개를 깔끔하게 적용하고 싶을 때 써.

**쓸 때 꿀팁 및 주의사항:**
*   **줄임말은 변수명 일치**: `style:color` 이렇게 쓰려면 `<script>` 안에 `let color = 'red';` 이런 식으로 같은 이름의 변수가 있어야 해. "이름표 똑바로 붙여라!" 없으면 에러 나니까 주의.
*   **CSS 속성명은 케밥 케이스(kebab-case)**: `backgroundColor` 아니고 `background-color` 써야 함. `style:background-color` 이렇게. HTML `style` 속성 규칙 그대로 따름.
*   **`style` 속성이랑 같이 쓰면 얘가 이김**: `<div style="color: blue;" style:color="red">` 면 빨간색으로 나와. `style:` 디렉티브가 나중에 적용돼서 덮어쓰는 구조거든. "내가 왕이 될 상인가!"
*   **너무 남발하진 말자**: 인라인 스타일은 재사용성이 떨어지고, CSS 복잡도를 높일 수 있어. 간단한 동적 스타일이나 특정 요소에만 적용할 때 쓰고, 공통 스타일은 `<style>` 태그나 별도 CSS 파일로 빼는 게 장기적으로 좋아. "모든 걸 인라인으로 해결하려 들면 스파게티 코드 직행열차 탄다."