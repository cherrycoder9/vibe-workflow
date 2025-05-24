```typescript
// 가상의 html 템플릿 리터럴 함수 (실제 Hono에서는 c.html() 내부적으로 처리)
const html = (strings: TemplateStringsArray, ...values: any[]) => {
  let result = strings[0];
  values.forEach((value, i) => {
    result += value + strings[i + 1];
  });
  return result; // 실제로는 더 복잡한 이스케이핑 및 처리 로직이 들어감
};

// raw 함수 (실제 Hono에서는 hono/helper/html 또는 hono/jsx/html 에 유사 기능 존재)
const raw = (value: string) => {
  // 이 예시에서는 간단히 객체로 감싸서 특별 취급을 표시
  // 실제 Hono의 raw는 이스케이핑을 건너뛰도록 하는 내부 메커니즘을 가짐
  return { isRaw: true, value };
};

// Hono 앱 및 라우트 핸들러 예시
app.get('/', (c) => {
  const name = 'John &quot;Johnny&quot; Smith'; // 이미 HTML 엔티티로 인코딩된 문자열
  // c.html 내부에서 html 템플릿 리터럴을 처리한다고 가정
  // ${raw(name)} 부분은 이스케이핑을 건너뛰고 name 변수 값을 그대로 삽입
  return c.html(html`<p>I'm ${raw(name)}.</p>`);
  // 예상 출력: <p>I'm John &quot;Johnny&quot; Smith.</p>
  // 만약 raw()를 안 썼다면?
  // return c.html(html`<p>I'm ${name}.</p>`);
  // 예상 출력 (Hono의 기본 이스케이핑 동작에 따라):
  // <p>I'm John &amp;quot;Johnny&amp;quot; Smith.</p> (이중 이스케이핑 발생 가능성)
});
```

**얘 뭐 하는 애냐?**
`raw()` 함수는 Hono의 HTML 템플릿 기능(`c.html`이나 JSX) 안에서 문자열을 출력할 때, "야, 이 부분은 내가 알아서 할 테니 너(Hono의 자동 HTML 이스케이핑 기능)는 건드리지 마!" 하고 선언하는 녀석이야. HTML 태그나 특수 문자가 포함된 문자열을 있는 그대로, 날것으로 출력하고 싶을 때 사용해.

**왜 쓰는데?**
1.  **이미 이스케이핑된 HTML 삽입**: 데이터베이스나 외부 API에서 가져온 데이터가 이미 HTML 엔티티(`&lt;`, `&gt;`, `&quot;` 등)로 안전하게 처리되어 있거나, 의도적으로 HTML 태그를 포함하고 있을 때가 있어. 이때 Hono가 기본으로 제공하는 자동 이스케이핑 기능이 또 한 번 작동하면 `&amp;lt;`처럼 이중으로 꼬여버려서 화면에 이상한 문자가 나오게 돼. `raw()`는 이걸 막아줘. "이미 양념 다 했는데 또 소금 치지 마!"
2.  **동적으로 생성된 HTML 조각 삽입**: 서버 사이드에서 특정 로직에 따라 HTML 조각을 만들어서 기존 템플릿에 끼워 넣고 싶을 때, 이 HTML 조각이 문자열 안에 태그 형태로 들어있다면 `raw()`로 감싸서 "이건 코드니까 그대로 넣어줘" 라고 알려줘야 해.
3.  **SVG 아이콘, 외부 스크립트 등 직접 삽입**: 가끔 SVG 코드나 특정 `<script>` 태그 내용을 문자열 변수에 담아 직접 HTML에 꽂아 넣어야 할 때가 있는데, 이때도 `raw()`가 유용하게 쓰일 수 있어. (물론 보안상 매우 신중해야 함!)

**언제 불려 나오냐?**
Hono의 `c.html` 메서드 안에서 템플릿 리터럴(` `` `)을 사용하거나, Hono의 JSX를 사용할 때, HTML로 출력될 문자열 변수 앞에 `raw()`를 붙여서 호출해.

```typescript
// Hono 앱 인스턴스가 이미 생성되어 있다고 가정 (const app = new Hono())

app.get('/', (c) => {
  // name 변수에는 이미 HTML 엔티티(&quot;)가 포함되어 있음
  const name = 'John &quot;Johnny&quot; Smith';
  // ${raw(name)}: "name 변수 안의 내용은 HTML 이스케이핑 하지 말고 그대로 출력해!"
  return c.html`<p>I'm ${raw(name)}.</p>`;
  // 브라우저에 실제로 렌더링될 HTML: <p>I'm John "Johnny" Smith.</p>
});

app.get('/danger', (c) => {
  // 이런 식으로 사용자 입력을 raw()로 직접 넣으면 XSS 공격에 매우 취약해짐! 절대 금지!
  const userInput = '<script>alert("XSS!")</script>';
  // return c.html`<p>Welcome, ${raw(userInput)}</p>`; // 이렇게 쓰면 큰일남!
  return c.html`<p>Welcome, ${userInput}</p>`; // Hono가 알아서 이스케이핑 해줌: <p>Welcome, &lt;script&gt;alert(&quot;XSS!&quot;)&lt;/script&gt;</p>
});
```

**쓸 때 꿀팁 및 주의사항:**
*   **XSS 공격의 주범이 될 수 있음! (매우 중요)**: `raw()`는 양날의 검이야. 사용자로부터 입력받은 내용을 아무런 검증 없이 `raw()`로 감싸서 HTML에 때려 박으면, 악의적인 사용자가 `<script>alert('해킹 성공!')</script>` 같은 코드를 입력했을 때 그 스크립트가 그대로 실행돼서 사이트가 홀라당 털릴 수 있어 (크로스 사이트 스크립팅, XSS 공격). "믿는 도끼에 발등 찍힌다"는 말이 딱 어울리지. **출처를 100% 신뢰할 수 있는 내용에만, 아주 제한적으로 사용해야 해.**
*   **언제 써야 할지 명확히 알기**: `raw()`는 "나는 이 HTML의 안전을 내가 책임진다"는 강력한 선언이야. 그냥 귀찮아서, 혹은 잘 몰라서 남용하면 보안 구멍이 숭숭 뚫릴 수 있어. Hono의 기본 HTML 이스케이핑 기능을 믿고, 정말 필요한 경우에만 전문가처럼 써야 해.
*   **대안 고려**: 사용자 생성 콘텐츠(UGC)를 안전하게 표시하려면, HTML을 직접 넣기보다는 마크다운(Markdown) 같은 안전한 형식을 사용하고 서버나 클라이언트에서 안전하게 HTML로 변환하는 라이브러리(예: `marked`, `DOMPurify`)를 쓰는 게 훨씬 나아.
*   **Hono의 JSX에서도 동일 원리**: `c.html` 템플릿 리터럴뿐만 아니라 Hono의 JSX에서도 `{raw(variable)}` 형태로 사용할 수 있어. 동작 원리는 똑같아.
*   **"이스케이프 해치"임을 기억**: `raw()`는 말 그대로 비상 탈출구 같은 거야. 평소에는 Hono가 제공하는 안전장치(자동 이스케이핑)를 최대한 활용하고, 정말 어쩔 수 없는 상황, 그리고 그 내용이 100% 안전하다고 확신할 때만 조심스럽게 사용해야 한다는 걸 명심해. "안전벨트 풀고 운전하는 격이니, 사고 나면 네 책임!"