# n8n Discord Message Trigger - 특정 문자열 패턴 필터링

## 📌 Context
n8n 워크플로우에서 Discord 메시지 트리거를 사용할 때, 모든 메시지에 반응하면 불필요한 워크플로우 실행이 발생한다. 특정 접두사(예: `/til`)로 시작하는 메시지에만 반응하도록 필터링 조건을 설정하여 TIL 수집 자동화 봇을 구현할 수 있다.

## ⚙️ Core

### 트리거 설정
- **Node**: Discord Trigger
- **Event**: Message Created

### IF Node 조건 설정 (문자열 패턴 필터링)
```json
{
  "conditions": {
    "string": [
      {
        "value1": "={{ $json.content }}",
        "operation": "startsWith",
        "value2": "/til"
      }
    ]
  }
}
```

### 워크플로우 구성
```
[Discord Trigger] → [IF: content startsWith '/til'] → [True: 처리 로직] → [False: No Operation]
```

### IF Node 설정 경로
1. Discord Trigger 노드 이후 **IF** 노드 추가
2. **Conditions** → **String** 선택
3. **Value 1**: `{{ $json.content }}`
4. **Operation**: `Starts With`
5. **Value 2**: `/til`

### 정규식 패턴 사용 시 (더 유연한 매칭)
```json
{
  "operation": "regex",
  "value1": "={{ $json.content }}",
  "value2": "^\/til\\s+.+"
}
```
이 패턴은 `/til ` 이후 최소 한 글자 이상의 내용이 있을 때만 통과시킨다.

## 💡 Insight
- `startsWith` 방식은 단순하고 빠르지만, `/tilemap` 같은 의도치 않은 명령어도 통과시킬 수 있다. 필요 시 `startsWith '/til '` (공백 포함)으로 더 엄격하게 필터링할 것
- 정규식(`regex`) 방식은 유연하지만 n8n 표현식 내 이스케이프 처리에 주의가 필요하다
- Discord Bot 권한에서 **Message Content Intent**가 활성화되어 있어야 `$json.content` 값이 정상적으로 수신된다
- 여러 명령어 패턴을 처리할 경우 IF 노드의 OR 조건을 활용하거나, Switch 노드로 대체하면 확장성이 좋다