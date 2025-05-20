# 01 – Tech Stack Template: 기술 스택 선정 가이드 🛠️

> 이 파일은 프로젝트에 적용할 주요 기술 영역과 선택지를 정리한 질문 목록입니다. 직접 수정할 필요 없이, 각 항목을 참고해 대화를 진행하세요. 최종 결정 내용은 `finalized.md`에 기록됩니다.

---

## 0. 준비 단계

* `00-planning` 결과(주요 요구 사항·예상 트래픽·핵심 기능 등)를 먼저 검토하세요.
* 모든 옵션을 사용할 필요는 없습니다. 프로젝트 규모와 목표에 맞춰 핵심 기술을 선택하세요.

---

## 1. 데이터베이스

### 질문

* 주로 다룰 데이터의 종류와 구조는 무엇인가요?
* 예상 데이터량, 읽기·쓰기 성능, 확장성 중 어떤 요소가 더 중요한가요?
* 관계형(SQL)과 비관계형(NoSQL) 중 어떤 특성이 필요합니까?

### 옵션 (하나 선택)

* 🤖 AI 추천 (요구사항 분석 후 추천)
* 사용 안함 (정적 콘텐츠)
* **간편 BaaS**: Supabase, Firebase, Appwrite, PocketBase
* **SQL**: PostgreSQL, MariaDB, SQLite, AWS Aurora, CockroachDB, TiDB
* **NoSQL**: MongoDB, DynamoDB, Couchbase, Cosmos DB, Firestore
* **인메모리/캐시**: Redis, Memcached
* **그래프 DB**: Neo4j, Amazon Neptune
* 기타: \_\_\_\_\_\_\_\_\_\_

---

## 2. 백엔드 프레임워크

### 질문

* 선호 언어와 개발 생산성·성능·커뮤니티 지원 중 우선순위는 무엇인가요?
* 서비스 복잡도에 따라 마이크로서비스 아키텍처를 고려하나요?

### 옵션 (하나 선택)

* 🤖 AI 추천 (서비스 특성 기반 추천)
* 사용 안함 (프론트엔드 직접 BaaS 호출)
* **Python**: FastAPI
* **JS/TS**: Hono, Fastify, ElysiaJS, NestJS
* **Rust**: Actix-Web
* **Go**: Fiber
* 기타: \_\_\_\_\_\_\_\_\_\_

---

## 3. 런타임 환경 (JS/TS 백엔드)

### 질문

* Node.js, Deno, Bun 중 어떤 런타임을 선호하시나요?
* 패키지 호환성, 성능, 안정성 측면에서 고려할 점은?

### 옵션 (하나 선택)

* 🤖 AI 추천 (프레임워크 호환성 기반 추천)
* Node.js
* Deno
* Bun
* 기타: \_\_\_\_\_\_\_\_\_\_

---

## 4. 서비스 간 통신

### 질문

* 내부 마이크로서비스 또는 외부 API와 어떤 방식으로 통신할 계획인가요?
* 단순 요청·응답, 실시간 스트리밍, 메시지 큐 등이 필요합니까?

### 옵션 (다중 선택)

* 🤖 AI 추천 (통신 패턴 기반 추천)
* 사용 안함 (단일 서비스)
* **REST / GraphQL / gRPC**
* **WebSocket / WebTransport / gRPC Streaming**
* **데이터 포맷**: JSON, Protocol Buffers, Avro, MessagePack
* 기타: \_\_\_\_\_\_\_\_\_\_

---

## 5. API 게이트웨이

### 질문

* 다수의 서비스 라우팅·인증·로깅·모니터링이 필요한가요?
* 관리형 서비스와 자체 호스팅 중 선호 방식은 무엇인가요?

### 옵션 (하나 선택)

* 🤖 AI 추천 (인프라 복잡도 기반 추천)
* 사용 안함 (게이트웨이 불필요)
* **관리형**: AWS API Gateway, Apigee, Azure API Management
* **오픈소스/자체 호스팅**: Kong, Tyk, Envoy, Traefik, Nginx, KrakenD
* 기타: \_\_\_\_\_\_\_\_\_\_

---

## 6. GraphQL 게이트웨이 (GraphQL 사용 시)

### 질문

* 여러 GraphQL 스키마 통합·페더레이션·캐싱 기능이 필요한가요?

### 옵션 (하나 선택)

* 🤖 AI 추천 (GraphQL 아키텍처 기반 추천)
* 사용 안함
* Apollo Gateway, GraphQL Mesh, Hasura(Remote Schemas), Mercurius, StepZen
* AWS AppSync
* 기타: \_\_\_\_\_\_\_\_\_\_

---

## 7. 로드 밸런서

### 질문

* 트래픽 분산을 위한 로드 밸런싱이 필요한가요?
* 클라우드 관리형 vs 소프트웨어 기반 중 어떤 방식을 선호하시나요?

### 옵션 (하나 선택)

* 🤖 AI 추천 (트래픽 패턴 기반 추천)
* 사용 안함 (단일 서버)
* **관리형**: AWS ELB/ALB/NLB, Google Cloud LB, Azure LB
* **소프트웨어**: Nginx, HAProxy, Traefik
* 기타: \_\_\_\_\_\_\_\_\_\_

---

## 8. 캐시 계층

### 질문

* 서비스 응답 속도 향상과 DB 부하 감소를 위해 캐시를 도입할 계획이신가요?
* 어떤 데이터를 캐시하고, 캐시 일관성 수준은 어느 정도로 유지하시겠습니까?
* 애플리케이션 캐시와 브라우저 캐시 중 무엇에 중점 두실 건가요?

### 옵션 (다중 선택)

* 🤖 AI 추천 (캐시 전략 기반 추천)

#### 8-1. 서버 사이드 캐시

* 사용 안함
* **인메모리 Key-Value**: Redis, Memcached, Dragonfly, KeyDB
* **분산 캐시**: Hazelcast, etcd
* **Java 기반**: Ehcache
* **관리형**: AWS ElastiCache, Google Memorystore, Azure Cache
* 기타: \_\_\_\_\_\_\_\_\_\_

#### 8-2. 클라이언트 사이드 캐시

* 사용 안함
* **HTTP Caching**: Cache-Control, ETag, Last-Modified
* **Service Worker + Cache API**
* **localStorage / sessionStorage**
* **IndexedDB**
* 기타: \_\_\_\_\_\_\_\_\_\_

---

## 9. CDN & WAF

### 질문

* 전 세계 사용자에게 정적 콘텐츠를 빠르게 제공하고, 보안을 강화할 필요가 있나요?
* 주요 대상 지역은 어디인가요?

### 옵션 (하나 또는 조합 선택)

* 🤖 AI 추천 (지역·보안 요구 기반 추천)
* 사용 안함
* **통합**: Cloudflare
* **AWS**: CloudFront + WAF
* **GCP**: Cloud CDN + Armor
* **Azure**: Front Door + WAF
* 기타: \_\_\_\_\_\_\_\_\_\_

---

## 10. 프론트엔드 / 그래픽스 / 게임 엔진

### 질문

* UI 개발, 2D/3D 그래픽, 게임 엔진 중 어떤 기술이 필요하신가요?
* 개발 생산성, 성능, 플랫폼 지원 중 우선순위는?

### 옵션 (다중 선택)

* 🤖 AI 추천 (프로젝트 요구 기반 추천)
* 사용 안함
* **웹 UI**: Vanilla, SvelteKit, Astro, Next.js, Qwik, Remix, Nuxt, Fresh
* **WebGL/3D**: Three.js, Babylon.js, PixiJS, WebGPU
* **게임 엔진**: Unity, Unreal, Godot, Bevy
* 기타: \_\_\_\_\_\_\_\_\_\_

---

## 11. 스타일링 툴

### 질문

* CSS, 유틸리티-퍼스트, CSS-in-JS 중 어떤 스타일링 방식을 선호하시나요?
* 디자인 시스템 통합이 필요한가요?

### 옵션 (하나 선택)

* 🤖 AI 추천 (디자인 요구 기반 추천)
* 사용 안함
* **Vanilla CSS / Modules**
* **유틸리티-퍼스트**: Tailwind, UnoCSS
* **CSS-in-JS**: StyleX, Styled Components, Emotion
* 기타: \_\_\_\_\_\_\_\_\_\_

---

## 12. 추가 도구 & 서비스

프로젝트 운영에 필요한 부가 도구들을 골라보세요.

### 12-1. 결제 처리

* 🤖 AI 추천 (시장·수수료 기반 추천)
* 사용 안함
* Stripe, PayPal, Toss, Wise, Payoneer
* 기타: \_\_\_\_\_\_\_\_\_\_

### 12-2. 분석 & A/B 테스트

* 🤖 AI 추천 (데이터 프라이버시·분석 규모 기반 추천)
* 사용 안함
* **분석**: Google Analytics, Plausible, PostHog
* **A/B 테스트**: GrowthBook, Flagsmith
* 기타: \_\_\_\_\_\_\_\_\_\_

### 12-3. SEO & SSR

* 🤖 AI 추천 (SEO·렌더링 요구 기반 추천)
* 사용 안함
* Prerender.io
* 프레임워크 내 SSR/SSG 활용
* 기타: \_\_\_\_\_\_\_\_\_\_

### 12-4. 컨테이너 & 배포

* 🤖 AI 추천 (운영 복잡성·확장성 기반 추천)
* 사용 안함
* Docker, Kubernetes
* Fly.io, AWS ECS/EKS/Fargate
* Vercel, Netlify, Render
* 기타: \_\_\_\_\_\_\_\_\_\_

### 12-5. CI/CD

* 🤖 AI 추천 (저장소·배포 환경 기반 추천)
* 사용 안함
* GitHub Actions, GitLab CI, CircleCI
* Jenkins, Bitbucket Pipelines
* 기타: \_\_\_\_\_\_\_\_\_\_

### 12-6. 테스트 자동화

* 🤖 AI 추천 (테스트 유형 기반 추천)
* 사용 안함
* Playwright, Vitest, k6, Newman
* 기타: \_\_\_\_\_\_\_\_\_\_

### 12-7. 광고 & 수익화

* 🤖 AI 추천 (콘텐츠·타겟 기반 추천)
* 사용 안함
* Google AdSense, Mediavine, Raptive, Ezoic, Monumetric
* 기타: \_\_\_\_\_\_\_\_\_\_

---

## 13. 데스크탑 애플리케이션 프레임워크

### 질문

* 데스크탑 앱 개발 계획이 있으신가요? 지원 OS와 주요 요구사항은 무엇인가요?

### 옵션 (하나 선택)

* 🤖 AI 추천 (플랫폼·언어 기반 추천)
* 사용 안함
* Tauri
* WinUI
* Qt
* Iced, Slint, egui (Rust)
* 기타: \_\_\_\_\_\_\_\_\_\_

---

> 위 항목을 참고해 대화를 진행하고, 최종 선택은 `finalized.md`에서 확인하세요.
