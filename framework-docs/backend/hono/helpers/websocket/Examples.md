**웹소켓 헬퍼 예제들**

Hono에서 웹소켓을 어떻게 써먹는지 보여주는 예제들이야. 실시간으로 서버랑 클라이언트가 데이터를 주고받고 싶을 때 이놈을 쓰지. "카톡처럼 실시간 채팅방 만들고 싶다고? 그럼 웹소켓이지!"

**1. 서버와 클라이언트 (Cloudflare Workers 환경 예시)**

이건 가장 기본적인 웹소켓 서버랑 클라이언트 구조야. 서버는 `/ws` 경로로 웹소켓 연결 요청이 오면 업그레이드해주고, 클라이언트에서 메시지 받으면 콘솔에 찍어줘. 클라이언트는 1초마다 현재 시간을 서버로 쏴주는 간단한 녀석이지.

```typescript
// server.ts (서버 쪽 코드)
import { Hono } from 'hono'
// Cloudflare Workers 환경용 웹소켓 업그레이드 함수를 불러온다.
import { upgradeWebSocket } from 'hono/cloudflare-workers'

const app = new Hono().get(
  '/ws', // '/ws' 경로로 GET 요청이 오면
  upgradeWebSocket(() => { // 웹소켓 연결로 업그레이드!
    return {
      onMessage: (event, ws) => { // 클라이언트가 메시지 보내면 실행
        console.log('클라이언트 메시지:', event.data) // 받은 데이터를 콘솔에 찍어보자
        ws.send('서버가 잘 받았음!') // 답장도 보내주자
      },
      onClose: () => {
        console.log('클라이언트 연결 끊김')
      }
    }
  })
)

export default app // Hono 앱을 내보낸다.
```

```typescript
// client.ts (클라이언트 쪽 코드, 보통 브라우저나 Node.js 환경)
import { hc } from 'hono/client' // Hono 클라이언트 라이브러리
import type app from './server' // 서버 앱의 타입을 가져와서 자동완성 꿀 빨자

// 서버 주소랑 타입을 알려주고 클라이언트 객체 생성
const client = hc<typeof app>('http://localhost:8787') // 로컬 개발 서버 주소
// '/ws' 경로로 웹소켓 연결 시도! hc의 $ws를 사용한다.
const ws = client.ws.$ws() // $ws()는 기본적으로 서버의 첫 번째 웹소켓 라우트를 찾는다.
                           // 명시적으로 경로를 주고 싶으면 client.ws('/ws').$ws() 처럼 쓴다.

ws.addEventListener('open', () => { // 웹소켓 연결 성공하면 실행
  console.log('서버랑 웹소켓 연결 성공!')
  setInterval(() => {
    const message = `지금은 ${new Date().toISOString()} 입니다.`
    console.log('서버로 보낼 메시지:', message)
    ws.send(message) // 1초마다 현재 시간을 서버로 전송
  }, 1000)
})

ws.addEventListener('message', (event) => { // 서버가 메시지 보내면 실행
  console.log('서버로부터 받은 메시지:', event.data)
})

ws.addEventListener('close', () => {
  console.log('서버랑 연결 끊김')
})

ws.addEventListener('error', (error) => {
  console.error('웹소켓 에러 발생:', error)
})
```

**2. Bun 환경에서 JSX와 함께 사용하기**

이건 Bun 런타임 환경에서 Hono의 JSX 기능이랑 웹소켓을 같이 쓰는 예제야. 서버는 `/` 경로로 접속하면 간단한 HTML 페이지를 보여주고, 이 페이지 안의 자바스크립트가 `/ws` 경로로 웹소켓 연결을 시도해. 서버는 연결된 클라이언트한테 0.2초마다 현재 시간을 보내주고, 클라이언트는 받은 시간을 화면에 업데이트해.

```typescript
// bun-server.tsx (Bun 환경 서버 코드)
import { Hono } from 'hono'
// Bun 환경용 웹소켓 생성 함수랑 HTML 템플릿 함수를 가져온다.
import { createBunWebSocket } from 'hono/bun'
import { html } from 'hono/html' // 또는 { jsxRenderer } from 'hono/jsx-renderer' 등 사용 가능

// Bun 환경에 맞는 웹소켓 업그레이드 함수랑 웹소켓 핸들러를 뽑아낸다.
const { upgradeWebSocket, websocket } = createBunWebSocket()

const app = new Hono()

// 루트 경로('/')로 접속하면 HTML 페이지를 보여준다.
app.get('/', (c) => {
  return c.html(
    <html>
      <head>
        <meta charSet='UTF-8' />
        <title>Bun 웹소켓 실시간 시간</title>
      </head>
      <body>
        <h1>실시간 현재 시간:</h1>
        <div id='now-time' style='font-size: 2em; color: blue;'>로딩 중...</div>
        {/* 인라인 스크립트로 웹소켓 클라이언트 로직을 바로 박아버리기! */}
        {html`
          <script>
            // 'ws://localhost:3000/ws' 주소로 웹소켓 연결 시도
            // 실제 배포 시에는 프로토콜(ws/wss)과 호스트, 포트를 환경에 맞게 바꿔야 함
            const wsProtocol = window.location.protocol === 'https:' ? 'wss:' : 'ws:';
            const wsUrl = \`\${wsProtocol}//\${window.location.host}/ws\`;
            const ws = new WebSocket(wsUrl);
            const $nowTime = document.getElementById('now-time');

            ws.onopen = () => {
              console.log('Bun 서버와 웹소켓 연결 성공!');
              $nowTime.textContent = '연결 완료! 시간 수신 대기 중...';
            };

            ws.onmessage = (event) => { // 서버가 메시지 보내면
              $nowTime.textContent = event.data; // 받은 데이터(시간)를 화면에 표시
            };

            ws.onclose = () => {
              console.log('Bun 서버와 연결 끊김');
              $nowTime.textContent = '연결 끊김. 새로고침 해주세요.';
            };

            ws.onerror = (error) => {
              console.error('웹소켓 에러:', error);
              $nowTime.textContent = '에러 발생!';
            };
          </script>
        `}
      </body>
    </html>
  )
})

// '/ws' 경로로 웹소켓 연결 요청 처리
app.get(
  '/ws',
  upgradeWebSocket((c) => { // 웹소켓 연결 요청을 업그레이드
    let intervalId: Timer | undefined // 타입스크립트니까 타입 명시
    console.log('클라이언트와 웹소켓 연결 시도...')
    return {
      onOpen: (_event, ws) => { // 연결 성공 시
        console.log('클라이언트와 웹소켓 연결 성공!')
        intervalId = setInterval(() => {
          ws.send(new Date().toString()) // 0.2초마다 현재 시간을 클라이언트로 전송
        }, 200)
      },
      onMessage: (event, ws) => { // 메시지 수신 시 (이 예제에서는 클라이언트가 보내는 메시지 처리는 없음)
        console.log('클라이언트로부터 메시지 받음:', event.data)
      },
      onClose: () => { // 연결 종료 시
        console.log('클라이언트와 웹소켓 연결 종료됨')
        if (intervalId) {
          clearInterval(intervalId) // setInterval 정리
        }
      },
      onError: (error) => { // 에러 발생 시
        console.error('웹소켓 서버 에러:', error)
        if (intervalId) {
          clearInterval(intervalId)
        }
      }
    }
  })
)

// Bun 서버 실행을 위해 fetch 핸들러와 websocket 핸들러를 내보낸다.
export default {
  port: 3000, // Bun이 사용할 포트
  fetch: app.fetch,
  websocket, // Bun에게 웹소켓 요청 처리 방법을 알려준다.
}
```

---

**얘 뭐 하는 애냐?**
Hono의 웹소켓 헬퍼는 서버랑 클라이언트 사이에 실시간 양방향 통신 채널(웹소켓)을 쉽게 만들고 관리하게 해주는 도구야. HTTP 요청/응답처럼 한 번 쏘고 끝나는 게 아니라, 한 번 연결해두면 계속해서 데이터를 주고받을 수 있어. "전화기처럼 한 번 연결하면 계속 대화하는 거지, 편지처럼 한 번 보내고 답장 기다리는 게 아니고."

**왜 쓰는데?**
1.  **실시간 데이터 전송**: 채팅 앱, 실시간 주식 시세, 게임, 알림 서비스처럼 서버가 클라이언트에게 즉각적으로 데이터를 밀어줘야 할 때(Server-Sent Events, SSE와 비슷하지만 양방향) 또는 클라이언트도 서버에게 빈번하게 데이터를 보내야 할 때 쓴다.
2.  **HTTP 폴링(Polling) 대체**: 예전에는 실시간처럼 보이려고 클라이언트가 1초마다 서버에 "새 소식 없어요?" 하고 계속 물어보는 폴링 방식을 썼는데, 이건 서버 자원 낭비가 심했어. 웹소켓은 연결 유지 비용이 훨씬 적어서 효율적이야. "맨날 전화해서 용건 묻는 것보다, 한 번 연결해놓고 필요할 때만 말하는 게 낫잖아?"
3.  **다양한 런타임 지원**: Hono는 Cloudflare Workers, Bun, Node.js 등 여러 환경에서 돌아가는데, 각 환경에 맞는 웹소켓 구현체를 쉽게 쓸 수 있도록 셔틀 역할을 해줘. `hono/cloudflare-workers`, `hono/bun`처럼 환경에 맞는 모듈만 가져다 쓰면 돼.

**언제 불려 나오냐?**
Hono 앱에서 특정 경로(예: `/ws`)로 GET 요청이 들어왔을 때, `upgradeWebSocket` 함수를 사용해서 일반 HTTP 연결을 웹소켓 연결로 "업그레이드" 시켜. 일단 연결되면 `onOpen` (연결 성공), `onMessage` (메시지 받음), `onClose` (연결 끊김), `onError` (에러 발생) 같은 이벤트 핸들러 안에서 실제 통신 로직을 짜는 거야.

**쓸 때 꿀팁 및 주의사항:**
*   **런타임별 임포트 경로 확인**: Cloudflare Workers 쓸 거면 `hono/cloudflare-workers`, Bun이면 `hono/bun`에서 `upgradeWebSocket` (또는 유사 함수)를 가져와야 해. Node.js는 `@hono/node-server` 패키지에 있는 걸 써야 하고. "동네마다 쓰는 연장이 다르다!"
*   **클라이언트 측 구현**: 서버만 웹소켓 준비한다고 끝나는 게 아냐. 브라우저 자바스크립트 (`new WebSocket('ws://...')`)나 Hono 클라이언트 (`hc().ws.$ws()`) 같은 걸로 클라이언트 쪽에서도 웹소켓 연결 코드를 짜줘야 해.
*   **상태 관리의 중요성**: 웹소켓은 한 번 연결되면 상태를 유지하기 때문에, 여러 클라이언트가 접속했을 때 각 클라이언트별로 데이터를 관리하거나 특정 클라이언트에게만 메시지를 보내는 등의 로직이 필요하면 상태 관리가 복잡해질 수 있어. (예: 사용자 ID랑 웹소켓 인스턴스 매핑) "손님들 방 번호랑 주문 내역 잘 챙겨야지, 안 그러면 옆방 음식 나간다."
*   **에러 처리 및 연결 끊김 복구**: 네트워크는 불안정할 수 있으니 `onError`, `onClose` 이벤트 핸들러를 잘 만들어서 예기치 않은 연결 끊김에 대비해야 해. 필요하면 클라이언트에서 자동으로 재연결하는 로직도 넣어주고.
*   **보안 고려**: `wss://` (WebSocket Secure) 프로토콜을 사용해서 통신 내용을 암호화하는 게 기본이야. 특히 민감한 데이터를 주고받는다면 필수! 그리고 아무나 웹소켓 연결 막 맺지 못하게 인증/인가 로직도 추가해야 할 수 있어. "비밀 대화는 암호 채널에서!"
*   **`Context` 객체 활용**: `upgradeWebSocket` 콜백 함수 안에서 `c` (Hono 컨텍스트 객체)를 사용할 수 있어. 이걸로 요청 헤더를 읽거나, 다른 미들웨어에서 설정한 값을 가져오는 등 다양한 작업을 할 수 있지.
*   **Bun 환경의 `websocket` 객체**: Bun 예제에서 `export default { ..., websocket }` 부분은 Bun 서버에게 "이 경로로 오는 웹소켓 요청은 이 핸들러가 처리할 거야"라고 알려주는 중요한 설정이야. 빼먹으면 웹소켓 연결 안 돼.