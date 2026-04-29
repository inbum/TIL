# n8n-nodes-discord-trigger-new: Publish / Unpublish 버그

## 📌 Context

n8n 커뮤니티 노드인 `n8n-nodes-discord-trigger-new`를 사용하여 Discord 이벤트를 트리거로 활용하던 중, 워크플로우 활성화(publish) 및 비활성화(unpublish) 시점에 오류가 발생하는 버그를 경험했다.

이 노드는 Discord 봇 클라이언트를 내부적으로 생성하여 n8n 워크플로우 활성화 시 `trigger()` 메서드를 호출해 리스너를 등록하고, 비활성화 시 `closeFunction()`을 통해 정리하는 구조다. 문제는 이 라이프사이클 처리 과정에서 발생했다.

---

## ⚙️ Core

### 버그 재현 패턴

워크플로우를 반복적으로 활성화/비활성화할 때 아래와 같은 오류가 발생한다.

```
Error: Already have an active trigger for this workflow
```

또는 비활성화 후 재활성화 시 Discord 클라이언트가 이전 세션을 정리하지 못해:

```
DiscordAPIError: 401: Unauthorized
Error [TOKEN_INVALID]: An invalid token was provided.
```

### 원인 분석

`n8n-nodes-discord-trigger-new` 노드의 `trigger()` / `closeFunction()` 구현 예시:

```typescript
// 문제가 되는 패턴 (closeFunction이 제대로 실행되지 않을 때)
async trigger(this: ITriggerFunctions): Promise<ITriggerResponse> {
  const client = new Client({ intents: [GatewayIntentBits.Guilds, GatewayIntentBits.GuildMessages] });

  await client.login(token);

  client.on('messageCreate', async (message) => {
    this.emit([this.helpers.returnJsonArray([{ content: message.content }])]);
  });

  // closeFunction이 누락되거나 client.destroy()가 호출되지 않으면
  // 재활성화 시 이전 클라이언트가 남아 충돌 발생
  const closeFunction = async () => {
    await client.destroy(); // 이 부분이 누락되거나 await 없이 호출되면 버그 발생
  };

  return { closeFunction };
}
```

### 임시 해결책 — 워크플로우 재시작 전 n8n 프로세스 재시작

```bash
# Docker 환경
docker restart n8n

# PM2 환경
pm2 restart n8n

# 직접 실행 환경
pkill -f n8n && n8n start
```

### 근본적 수정 방향 (커뮤니티 기여 또는 fork)

```typescript
const closeFunction = async () => {
  try {
    client.removeAllListeners(); // 이벤트 리스너 먼저 제거
    if (client.isReady()) {
      await client.destroy();   // 클라이언트 완전 종료
    }
  } catch (error) {
    console.error('Discord client cleanup error:', error);
  }
};

return { closeFunction };
```

---

## 💡 Insight

- **재현 조건**: 워크플로우를 빠르게 반복 활성화/비활성화하거나, n8n이 비정상 종료된 후 재시작될 때 `closeFunction`이 실행되지 않으면 Discord 클라이언트 인스턴스가 메모리에 잔류한다.
- **트레이드오프**: 커뮤니티 노드 특성상 공식 지원이 없으므로, 프로덕션 환경에서는 직접 fork하여 `closeFunction` 안정성을 보강하거나 공식 Discord webhook 노드(`n8n-nodes-base`) 사용을 우선 검토하는 것이 바람직하다.
- **활용 가능성**: 이 패턴은 WebSocket이나 장기 연결(long-lived connection)을 사용하는 모든 n8n 커뮤니티 트리거 노드에 공통적으로 적용되는 문제다. 커스텀 트리거 노드 개발 시 `closeFunction` 내 리소스 정리 로직을 반드시 포함해야 한다.
- **모니터링 팁**: n8n 로그에서 `trigger` 키워드를 필터링하면 publish/unpublish 사이클 오류를 빠르게 감지할 수 있다.
