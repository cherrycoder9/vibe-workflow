Drizzle ORM은 타입스크립트 ORM인데, 관계형 데이터베이스랑 SQL스러운 쿼리 API 둘 다 지원하고, 설계 자체가 서버리스 환경에 최적화되어 있어요.

**사용법**

```bash
npx sv add drizzle
```

**이거 쓰면 뭐가 딸려오냐?**

*   SvelteKit 서버 파일 안에서만 데이터베이스에 접근하도록 깔끔하게 정리된 설정
*   DB 접속 정보 같은 비밀번호는 `.env` 파일에 안전하게 보관
*   인기 있는 Lucia 인증 애드온이랑 찰떡궁합
*   로컬에서 DB 돌려볼 때 쓰라고 Docker 설정도 선택적으로 제공 (선택 사항)

**옵션**

*   `database`
    어떤 DB 쓸래?
    *   `postgresql` — 인싸 오픈소스 DB
    *   `mysql` — 얘도 만만찮은 오픈소스 DB
    *   `sqlite` — DB 서버 없이 파일 하나로 퉁치는 초경량 DB

    ```bash
    npx sv add drizzle=database:postgresql
    ```
*   `client`
    어떤 SQL 클라이언트 쓸래? (DB 종류에 따라 다름)
    *   PostgreSQL용: `postgres.js`, `neon`
    *   MySQL용: `mysql2`, `planetscale`
    *   SQLite용: `better-sqlite3`, `libsql`, `turso`

    ```bash
    npx sv add drizzle=database:postgresql+client:postgres.js
    ```
    Drizzle은 사실 드라이버 맛집이라 십수 개가 넘는 DB 드라이버랑 다 친해요. 여기선 그냥 대표 선수 몇 명만 보여주는 거고, '나만의 드라이버' 쓰고 싶으면 일단 아무거나 골라서 설치한 다음에 Drizzle 공식 드라이버 리스트 보고 스윽 바꾸면 됩니다. "일단 아무거나 잡고 시작해, 커스텀은 나중에!"
*   `docker`
    Docker Compose 설정도 같이 깔아줄까? (postgresql이나 mysql 쓸 때만 가능. SQLite는 해당 사항 없음)

    ```bash
    npx sv add drizzle=database:postgresql+client:postgres.js+docker:yes
    ```

---

**얘 뭐 하는 애냐? (기능 및 목적)**
`sv add drizzle`은 이미 만들어진 SvelteKit 프로젝트에 Drizzle ORM을 후딱 설치하고 기본 세팅까지 해주는 마법사 같은 명령어입니다. DB 연결 설정, 필수 파일 생성, 관련 패키지 설치까지 알아서 착착 진행해주죠. "DB 연동? 복잡하게 생각 말고 얘한테 시켜!"

**왜 쓰는데?**
1.  **DB 연동 자동화**: DB 종류랑 클라이언트만 고르면 나머진 알아서. "DB 설정하다 빡칠 일 줄여줌."
2.  **타입스크립트와 찰떡궁합**: Drizzle ORM 자체가 타입스크립트 기반이라, SvelteKit의 타입스크립트 환경에서 물 흐르듯 자연스럽게 DB 작업을 할 수 있습니다. "타입 오류? 컴파일 타임에 다 잡아줄게."
3.  **SvelteKit 최적화**: SvelteKit의 서버 환경(`.server.ts` 파일)에 DB 로직을 안전하게 배치하고, `.env` 파일로 민감 정보를 관리하는 등 SvelteKit스러운 방식으로 DB를 쓰게 해줍니다.
4.  **부가 기능 호환성**: Lucia 같은 인증 라이브러리랑 같이 쓸 때 필요한 설정을 미리 고려해줍니다. "인증? DB? 한 번에 가자!"

**언제 불려 나오냐?**
SvelteKit 프로젝트 진행 중에 "아, 이제 DB 붙여야겠다!" 싶을 때 터미널에 `npx sv add drizzle`을 외치면 됩니다. 프로젝트 생성(`sv create`) 후 DB 기능 추가 단계에서 주로 사용되죠.

**쓸 때 꿀팁 및 주의사항:**
*   **DB 선택은 시작 전에**: `postgresql`, `mysql`, `sqlite` 중 뭘 쓸지는 프로젝트 성격에 맞춰 미리 정하는 게 좋습니다. "일단 깔고 나중에 바꾸려면... 좀 귀찮아진다."
*   **클라이언트 궁합 체크**: DB 종류에 맞는 클라이언트를 선택해야 합니다. 예를 들어 PostgreSQL 쓰는데 `mysql2` 클라이언트 고르면... "너 T야? F야? 확실히 해!" 같은 상황이 펼쳐집니다.
*   **`.env`는 소중히**: DB 접속 정보가 담긴 `.env` 파일은 절대 Git에 올리지 마세요. `.gitignore`에 등록되어 있는지 꼭 확인! "내 DB 비밀번호는 나만 안다."
*   **Docker는 개발용**: `docker:yes` 옵션은 로컬 개발 환경에서 편하게 DB 띄우라고 있는 겁니다. 실제 서비스 올릴 땐 진짜 DB 서버나 클라우드 DB 쓰세요. "연습은 Docker로, 실전은 프로 장비로!"
*   **Drizzle-Kit은 별도**: 이 명령어는 Drizzle ORM '설치'만 도와줍니다. DB 스키마 만들고 변경사항 DB에 반영(마이그레이션)하려면 `drizzle-kit`이라는 도구를 따로 써야 해요. "집은 지어줬는데, 가구는 네가 채워야지."
*   **Drizzle ORM 공부는 필수**: `sv add drizzle`은 세팅 도우미일 뿐, Drizzle ORM 자체의 강력한 기능(쿼리 빌더, 스키마 정의 등)을 제대로 쓰려면 Drizzle 공식 문서를 파야 합니다. "설치했다고 다 아는 거 아니다. 이제부터 시작이다!"