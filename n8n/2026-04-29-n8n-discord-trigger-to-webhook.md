# n8n에서 Discord Trigger를 Webhook으로 대체

## 📌 Context

n8n에서 Discord 이벤트를 수신하기 위해 기본적으로 Discord Trigger 노드를 사용할 수 있다. 그러나 Discord Trigger 노드는 내부적으로 WebSocket Gateway 연결을 유지해야 하므로, 설정 복잡도나 인증 문제(봇 토큰 권한, Intent 설정 등)로 인해 운영 환경에서 불안정할 수 있다. 이를 대체하기 위해 Discord의 Interactions Endpoint(Webhook) 방식을 n8n의 Webhook 노드와 연동하면, HTTP 기반의 가볍고 디버깅하기 쉬운 파이프라인을 구성할 수 있다.

## ⚙️ Core

### 1. n8n Webhook 노드 설정

- **Method**: POST
- **Path**: `discord-interactions`
- **Response Mode**: `Using Respond to Webhook Node` (Discord는 즉각 응답 필요)

```
https://your-n8n-instance.com/webhook/discord-interactions
```

### 2. Discord Application Interactions Endpoint 등록

[Discord Developer Portal](https://discord.com/developers/applications) → 앱 선택 → **General Information** → `Interactions Endpoint URL`에 위 n8n Webhook URL 입력

### 3. Discord 서명 검증 (n8n Code 노드)

Discord는 Ed25519 서명 검증을 요구한다. n8n의 Code 노드에서 검증 후 처리:

```javascript
// n8n Code 노드 (Node.js)
const { createVerify } = require('crypto');

const PUBLIC_KEY = 'YOUR_DISCORD_APPLICATION_PUBLIC_KEY';

const signature = $input.first().json.headers['x-signature-ed25519'];
const timestamp = $input.first().json.headers['x-signature-timestamp'];
const rawBody = $input.first().json.rawBody; // 원본 body 문자열

const verify = createVerify('SHA256');
verify.update(timestamp + rawBody);

const isValid = verify.verify(
  {
    key: Buffer.from(PUBLIC_KEY, 'hex'),
    format: 'der',
    type: 'spki',
  },
  Buffer.from(signature, 'hex')
);

if (!isValid) {
  throw new Error('Invalid Discord signature');
}

const body = JSON.parse(rawBody);
return [{ json: body }];
```

### 4. PING 응답 처리 (Respond to Webhook 노드)

Discord는 Endpoint 등록 시 PING(type: 1) 요청을 보낸다. IF 노드로 분기 후 응답:

```json
{ "type": 1 }
```

### 5. n8n Workflow 구조

```
[Webhook] → [Code: 서명 검증] → [IF: type == 1 (PING)]
                                      ├─ true  → [Respond: {type:1}]
                                      └─ false → [IF: 커맨드 분기] → [처리 로직] → [Respond]
```

## 💡 Insight

- Discord Trigger 노드(WebSocket 방식)는 모든 Gateway 이벤트(메시지 생성, 반응 등)를 수신할 수 있지만, Webhook 방식은 **슬래시 커맨드·버튼·모달 등 Interaction 이벤트**에 특화되어 있다. 수신 범위를 먼저 파악하고 선택해야 한다.
- Discord의 서명 검증은 필수이며, 검증을 생략하면 Discord Developer Portal에서 Endpoint 등록 자체가 거부된다.
- n8n의 기본 Webhook 노드는 rawBody를 별도로 보존하지 않을 수 있으므로, 서명 검증을 위한 원본 바이트 보존 여부를 확인해야 한다. 필요 시 n8n 앞단에 Nginx/프록시에서 rawBody를 헤더로 주입하는 방식도 고려할 수 있다.
- Webhook 방식은 상태를 유지하지 않아 서버 재시작에 강하고, n8n 워크플로우 단위로 독립 관리가 가능하여 유지보수에 유리하다.