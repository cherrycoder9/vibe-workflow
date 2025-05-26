# Secure Headers Middleware

보안 헤더 미들웨어는 보안 관련 HTTP 헤더 설정을 아주 쉽게 만들어주는 녀석이야. 헬멧(Helmet.js 같은 유명한 보안 미들웨어) 기능에서 일부 영감을 받아서, 특정 보안 헤더를 켜고 끄는 걸 마음대로 제어할 수 있게 해줘. "보안 설정, 이제 골치 아프게 이것저것 만질 필요 없다 이거지."

**임포트**

```javascript
import { Hono } from 'hono'
import { secureHeaders } from 'hono/secure-headers'
```

**사용법**
기본빵으로 써도 알아서 최적 설정이 적용돼.

```javascript
const app = new Hono()
app.use(secureHeaders())
```

쓸데없는 헤더는 `false` 때려서 꺼버릴 수 있어.

```javascript
const app = new Hono()
app.use(
  '*',
  secureHeaders({
    xFrameOptions: false, // X-Frame-Options 헤더 비활성화
    xXssProtection: false, // X-XSS-Protection 헤더 비활성화
  })
)
```

기본 헤더 값을 내 입맛대로 문자열로 덮어쓰는 것도 가능해.

```javascript
const app = new Hono()
app.use(
  '*',
  secureHeaders({
    strictTransportSecurity: // HSTS 설정 강화
      'max-age=63072000; includeSubDomains; preload',
    xFrameOptions: 'DENY', // 모든 프레이밍 차단
    xXssProtection: '1', // XSS 필터 활성화 (구형 브라우저용, 주의해서 사용)
  })
)
```

**지원 옵션**
각 옵션은 아래 표에 있는 헤더 키-값 쌍에 해당해. "뭔 옵션이 뭔 헤더인지 한눈에 딱 들어오지?"

| 옵션                                | 헤더                                  | 값                                                              | 기본값 |
| :---------------------------------- | :------------------------------------ | :-------------------------------------------------------------- | :----- |
| - (옵션 없음)                       | `X-Powered-By`                        | (헤더 삭제)                                                       | `True` |
| `contentSecurityPolicy`             | `Content-Security-Policy`             | 사용법: `Content-Security-Policy` 직접 설정                     | 설정 없음 |
| `contentSecurityPolicyReportOnly`   | `Content-Security-Policy-Report-Only` | 사용법: `Content-Security-Policy` 직접 설정 (리포트 전용)         | 설정 없음 |
| `crossOriginEmbedderPolicy`         | `Cross-Origin-Embedder-Policy`        | `require-corp`                                                  | `False`|
| `crossOriginResourcePolicy`         | `Cross-Origin-Resource-Policy`        | `same-origin`                                                   | `True` |
| `crossOriginOpenerPolicy`           | `Cross-Origin-Opener-Policy`          | `same-origin`                                                   | `True` |
| `originAgentCluster`                | `Origin-Agent-Cluster`                | `?1` (true를 의미하는 HTTP 구조화 필드 값)                         | `True` |
| `referrerPolicy`                    | `Referrer-Policy`                     | `no-referrer`                                                   | `True` |
| `reportingEndpoints`                | `Reporting-Endpoints`                 | 사용법: `Content-Security-Policy` 등과 연계하여 직접 설정        | 설정 없음 |
| `reportTo`                          | `Report-To`                           | 사용법: `Content-Security-Policy` 등과 연계하여 직접 설정        | 설정 없음 |
| `strictTransportSecurity`           | `Strict-Transport-Security`           | `max-age=15552000; includeSubDomains` (약 6개월)                | `True` |
| `xContentTypeOptions`               | `X-Content-Type-Options`              | `nosniff`                                                       | `True` |
| `xDnsPrefetchControl`               | `X-DNS-Prefetch-Control`              | `off`                                                           | `True` |
| `xDownloadOptions`                  | `X-Download-Options`                  | `noopen`                                                        | `True` |
| `xFrameOptions`                     | `X-Frame-Options`                     | `SAMEORIGIN`                                                    | `True` |
| `xPermittedCrossDomainPolicies`     | `X-Permitted-Cross-Domain-Policies`   | `none`                                                          | `True` |
| `xXssProtection`                    | `X-XSS-Protection`                    | `0` (비활성화)                                                    | `True` |
| `permissionPolicy`                  | `Permissions-Policy`                  | 사용법: `Permissions-Policy` 직접 설정                          | 설정 없음 |

---

**얘 뭐 하는 애냐?**
웹 서버가 브라우저한테 "야, 너 이런이런 보안 규칙 좀 지켜라!"하고 명령 내릴 때 쓰는 HTTP 응답 헤더들을 한 방에, 그것도 아주 쉽게 설정해주는 미들웨어다. "보안 설정계의 인스턴트 라면이랄까? 물 붓고 기다리면 끝."

**왜 쓰는데?**
1.  **보안 강화**: 클릭재킹, XSS(교차 사이트 스크립팅), MIME 타입 스니핑 같은 웹 공격들을 막는 데 도움을 준다. "내 소중한 앱, 아무나 못 건드리게 철벽 치는 거지."
2.  **개발 편의성**: 보안 헤더 하나하나 직접 코드로 넣으려면 귀찮고 빼먹기 쉬운데, 얘는 그걸 대신해준다. "개발자는 개발에만 집중해라, 보안은 얘가 알아서 해줄 테니."
3.  **모범 사례 적용**: 헬멧 같은 잘나가는 보안 라이브러리들처럼, "이 정도는 해줘야지~" 하는 보안 설정들을 기본으로 깔아준다. "보안 전문가 빙의 가능."

**언제 불려 나오냐?**
Hono 앱에서 HTTP 요청이 들어오고 나갈 때마다 중간에 껴서 응답 헤더에 보안 관련 딱지를 덕지덕지 붙여준다. 보통 앱 초기 설정에서 `app.use(secureHeaders())` 한 줄이면 모든 요청에 적용 완료. "문지기처럼 모든 응답을 검문한다."

**쓸 때 꿀팁 및 주의사항:**
*   **기본 설정 파악**: 그냥 `secureHeaders()`만 써도 대부분 알아서 잘해주지만, 어떤 헤더가 어떤 값으로 들어가는지는 한번 훑어보는 게 좋다. "자동이라도 뭐가 어떻게 돌아가는지는 알아야지, 암."
*   **CSP는 신중하게**: `contentSecurityPolicy` 옵션은 매우 강력하지만, 잘못 건드리면 웹사이트가 아예 작동안할 수도 있다. 충분히 테스트하고 적용해야 한다. "양날의 검이다. 잘 쓰면 약, 못 쓰면 독."
*   **`X-Powered-By` 헤더 제거**: 기본으로 서버 정보 알려주는 `X-Powered-By` 헤더를 지워주는데, 이거 아주 착한 기능이다. "내 서버 정보는 비밀."
*   **HSTS (`Strict-Transport-Security`) 주의**: 한번 설정하면 브라우저가 지정된 기간 동안 무조건 HTTPS로만 접속하려고 한다. 개발 중에 HTTP 환경에서 테스트할 때 골치 아플 수 있으니, `max-age` 값을 짧게 하거나 개발 환경에서는 꺼두는 것도 방법. 특히 `preload` 옵션은 진짜 신중하게 써야 한다. "한번 걸면 빼도 박도 못할 수 있다."
*   **`X-XSS-Protection` 헤더**: 요즘 브라우저들은 CSP를 더 권장해서 이 헤더는 `0` (비활성화)으로 설정하는 게 대세다. 이 미들웨어 기본값도 그걸 따른다. "옛날 기술이지만, 혹시 모를 상황 대비용."
*   **커스터마이징은 알고 하자**: 특정 헤더를 끄거나 값을 바꾸고 싶으면 옵션으로 조절하면 된다. 근데 뭘 건드리는지는 알고 건드리자. "설정값 하나 바꿨을 뿐인데... 사이트가 터졌다고?"
*   **문서 정독**: 결국 가장 정확한 정보는 공식 문서에 있다. 옵션이 워낙 많으니 한번 쭉 읽어보는 게 좋다. "RTFM! 개발자의 숙명이다."