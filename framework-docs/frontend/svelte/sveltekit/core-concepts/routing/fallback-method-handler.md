Fallback method handler (나머지 떨이 처리반)

`+server.js` 파일에서 `POST`처럼 특정 HTTP 메서드에 대한 처리 함수를 만들 수 있잖아? 근데 만약 `PUT`, `PATCH`, `DELETE`처럼 네가 미처 준비 못한 메서드로 요청이 들어오거나, 심지어 `MOVE` 같이 듣도 보도 못한 메서드로 요청이 오면 어쩔 거야? 이때 나서는 게 바로 이 `fallback` 핸들러야. "어떤 메서드든 일단 내가 다 받아줄게!" 하는 역할이지.

```javascript
// src/routes/api/add/+server.js

import { json, text } from '@sveltejs/kit';
import type { RequestHandler } from './$types';

export const POST: RequestHandler = async ({ request }) => {
	const { a, b } = await request.json();
	return json(a + b); // POST 요청은 덧셈해서 JSON으로 응답!
};

// 이 핸들러가 PUT, PATCH, DELETE 등등 나머지 요청 다 처리함
export const fallback: RequestHandler = async ({ request }) => {
	return text(`내가 니 ${request.method} 요청 받았다!`); // "어떤 메서드든 일단 접수!"
};
```

참고로 `HEAD` 요청의 경우, `GET` 핸들러가 있으면 `GET`이 먼저 처리하고, `GET` 핸들러가 없을 때만 이 `fallback` 핸들러가 나선다.

---

**얘 뭐 하는 애냐?**
`+server.js`에서 `export const GET = ...`, `export const POST = ...` 이런 식으로 특정 HTTP 메서드별 담당자를 지정해두잖아? 근데 클라이언트가 만약 `PUT`, `DELETE` 아니면 듣도 보도 못한 `PROPFIND` 같은 메서드로 요청을 보냈는데 해당 담당자가 없으면? 그때 "야, 내가 처리할게!" 하고 나서는 놈이 바로 이 `fallback` 핸들러다. 한마디로 **"기타 등등 모든 HTTP 메서드 처리반"**인 셈.

**왜 쓰는데?**
1.  **예상 못한 요청에도 서버 안정성 확보**: 클라이언트가 이상한 메서드로 요청 보내도 서버가 "나 몰라" 하고 뻗어버리는 대신, `fallback`이 "님, 그건 좀..." 하고 점잖게 응대할 수 있게 해준다. "일단 진정하고, 무슨 일인지 들어나 보자."
2.  **API 유연성 증대**: 모든 HTTP 메서드에 대해 일일이 핸들러를 만들 필요 없이, `fallback` 하나로 "기본 응답"이나 "지원하지 않는 메서드입니다" 같은 메시지를 통일성 있게 보낼 수 있다. 개발자 편의성 UP.
3.  **디버깅 및 로깅**: 어떤 종류의 (이상한) 요청들이 우리 서버로 들어오는지 `fallback` 핸들러에서 `request.method`를 찍어보면 파악하기 쉽다. "별의별 요청이 다 들어오네, 구경 한번 해볼까?"
4.  **특정 메서드 그룹핑**: 예를 들어 `PUT`, `PATCH`, `DELETE` 요청에 대해 비슷한 방식(예: "관리자에게 문의하세요")으로 응답하고 싶다면, 각 핸들러를 만들기보다 `fallback`에서 `request.method`를 확인하고 한꺼번에 처리할 수도 있다. (물론 권장하는 방식은 아님)

**언제 불려 나오냐?**
클라이언트가 `+server.js` 파일이 위치한 경로로 HTTP 요청을 보냈는데, 그 요청의 HTTP 메서드(예: `PUT`)에 해당하는 `export const PUT = ...` 같은 전용 핸들러가 정의되어 있지 않을 때. 그때 이 `fallback` 핸들러가 "나왔다!" 하고 호출된다.
단, `HEAD` 요청은 좀 특별해서, 만약 `GET` 핸들러가 정의되어 있다면 `GET` 핸들러가 우선적으로 처리한다. `GET` 핸들러가 없을 때만 `HEAD` 요청도 `fallback`으로 넘어온다.

**쓸 때 꿀팁 및 주의사항:**
*   **`HEAD` 요청의 특수성 기억**: `HEAD`는 `GET`의 그림자 같은 놈이라, `GET` 있으면 `GET`이 처리한다. `fallback`은 `GET` 없을 때만 `HEAD` 만날 수 있다.
*   **만능이라고 남용은 금물**: `fallback`은 말 그대로 "기타"를 위한 거다. API의 주요 동작(생성, 조회, 수정, 삭제 등)은 `POST`, `GET`, `PUT`, `DELETE` 같은 명시적인 핸들러로 만드는 게 정석이다. 다 `fallback`으로 퉁치면 나중에 코드 보고 "이게 뭔 API여?" 하게 된다.
*   **적절한 HTTP 상태 코드 사용**: `fallback`에서 "지원 안 하는 메서드임"이라고 알려줄 땐 HTTP 상태 코드 405 (Method Not Allowed)를 반환하는 게 좋다. 예제처럼 그냥 텍스트만 띡 보내는 건 "일단 돌아는 간다" 수준.
*   **`request.method`로 분기 가능**: `fallback` 안에서 `request.method` 값을 확인하면 "이 메서드일 땐 이렇게, 저 메서드일 땐 저렇게" 다르게 대응할 수 있다. 하지만 이게 복잡해지면 그냥 각 메서드별 핸들러를 만드는 게 낫다.
*   **보안은 항상 신경 써라**: `fallback`이 모든 요청을 받는다고 해서 아무 작업이나 막 하면 안 된다. 특히 데이터 변경이나 민감 정보 접근과 관련된 로직은 절대 금물. "아무나 문 열어준다고 집문서까지 내주진 말자."