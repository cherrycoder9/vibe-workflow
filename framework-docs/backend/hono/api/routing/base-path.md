베이스 경로(Base Path)

베이스 경로를 지정할 수 있습니다.

```javascript
const api = new Hono().basePath('/api')
api.get('/book', (c) => c.text('책 목록')) // GET /api/book 요청 시 처리
```

---

**얘 뭐 하는 애냐?**
`basePath()`는 Hono 앱에 등록하는 모든 라우트 앞에 공통적으로 붙는 기본 경로, 즉 "간판 주소" 같은 걸 설정하는 기능입니다. `api.basePath('/api')`라고 해두면, 그 뒤에 `api.get('/users', ...)`를 등록하든 `api.post('/items', ...)`를 등록하든, 실제로는 각각 `/api/users`, `/api/items`로 요청해야 접근할 수 있게 되는 거죠. 마치 모든 파일들을 'api'라는 폴더 안에 넣어두는 것과 비슷합니다.

**왜 쓰는데?**
1.  **API 버전 관리**: `/v1/users`, `/v2/users`처럼 API 버전을 URL로 깔끔하게 구분하고 싶을 때 아주 유용합니다. `const v1 = new Hono().basePath('/v1')`, `const v2 = new Hono().basePath('/v2')` 이런 식으로 각각 다른 버전의 API 그룹을 만들 수 있죠. "사장님, 저희 가게 1호점, 2호점 간판 따로 답니다!"
2.  **라우트 그룹핑**: `/admin/dashboard`, `/admin/settings`처럼 특정 기능과 관련된 API들을 하나의 `basePath` 아래로 묶어서 관리하면 코드가 훨씬 체계적으로 보입니다. "어드민 관련 기능은 다 이쪽으로 모여!"
3.  **배포 환경 유연성**: 개발할 때는 루트(`/`)에서 앱을 돌렸는데, 실제 서버에서는 `/myapp/` 같은 하위 경로로 배포해야 할 때가 있습니다. 이때 앱 코드에서 모든 경로를 일일이 수정하는 대신 `app.basePath('/myapp')` 한 줄이면 깔끔하게 해결됩니다. "이사 가도 간판만 바꿔 달면 장사 가능!"

**언제 불려 나오냐?**
Hono 인스턴스를 만들고 나서, 각종 라우트(`get`, `post` 등)를 정의하기 *전에* 보통 한 번 호출해서 설정합니다. `const app = new Hono().basePath('/my-app')` 이런 식으로 체이닝해서 바로 쓰거나, `app.basePath('/my-app')`처럼 나중에 설정할 수도 있지만, 라우트 등록 전에 해주는 게 일반적입니다. 한번 설정되면 그 `app` 인스턴스에 추가되는 모든 라우트에 자동으로 이 `basePath`가 앞에 붙습니다.

**쓸 때 꿀팁 및 주의사항:**
*   **중복은 금물**: `app.basePath('/api')` 해놓고 `app.get('/api/users', ...)` 이렇게 또 `basePath`를 경로에 적으면 최종 경로는 `/api/api/users`가 되는 대참사가 벌어집니다. `basePath`는 한번 설정했으면 그 뒤 라우트 정의할 때는 `basePath`를 뺀 나머지 경로만 적으세요. "사장님이 간판 달아줬는데 알바가 그 앞에 또 간판 다는 격."
*   **슬래시 일관성**: `basePath('/api')`나 `basePath('api/')`나 Hono가 알아서 `/api`로 잘 처리해주긴 하지만, 혼란을 피하려면 그냥 `basePath('/api')`처럼 앞 슬래시는 붙이고 뒤 슬래시는 빼는 식으로 통일하는 게 속 편합니다.
*   **`app.mount()`와의 차이**: `app.mount('/admin', adminApp)`는 다른 Hono 인스턴스나 앱을 특정 경로에 "통째로 끼워넣는" 느낌이라면, `basePath`는 현재 Hono 인스턴스 *내부의* 모든 라우트에 공통 접두사를 붙이는 겁니다. 용도가 살짝 달라요.
*   **테스트할 때도 전체 경로로**: `app.basePath('/api')`를 설정하고 `/test` 라우트를 만들었다면, `app.request('/api/test')`처럼 `basePath`를 포함한 전체 경로로 테스트해야 합니다. "손님은 전체 주소 보고 찾아온다."
*   **여러 `basePath` 호출 시**: `app.basePath('/foo').basePath('/bar')` 이렇게 여러 번 호출하면, 보통 마지막에 호출된 `/bar`만 적용됩니다. 굳이 이렇게 쓸 일은 없겠지만, "어? 왜 경로가 이상하지?" 싶을 때 한번 떠올려보세요. 웬만하면 `basePath`는 한 번만 명확하게 설정하는 게 좋습니다.