# n8n을 활용한 GitHub TIL 문서 자동 작성

## 📌 Context
매일 학습한 내용을 GitHub TIL 레포지토리에 마크다운 문서로 기록하는 것은 개발자에게 중요한 습관이다. 하지만 매번 파일 생성 → 내용 작성 → 커밋 → 푸시의 반복 과정은 번거롭고 지속성을 떨어뜨린다. n8n을 활용하면 입력 트리거(Webhook, Form, Telegram 등)부터 GitHub API 연동까지 노코드/로우코드 방식으로 TIL 자동 발행 파이프라인을 구성할 수 있다.

## ⚙️ Core

### 워크플로우 구성 (n8n)

```
[Trigger] → [AI 문서 생성] → [GitHub 파일 생성 API]
```

**1. Trigger 노드 예시 (Webhook)**
- Method: POST
- Path: `/til`
- Body: `{ "content": "오늘 배운 내용..." }`

**2. HTTP Request 노드 — Claude API로 TIL 문서 생성**

```json
{
  "method": "POST",
  "url": "https://api.anthropic.com/v1/messages",
  "headers": {
    "x-api-key": "{{ $env.ANTHROPIC_API_KEY }}",
    "anthropic-version": "2023-06-01",
    "content-type": "application/json"
  },
  "body": {
    "model": "claude-sonnet-4-6",
    "max_tokens": 2048,
    "messages": [
      {
        "role": "user",
        "content": "다음 내용을 TIL 마크다운 문서로 작성해줘: {{ $json.content }}"
      }
    ]
  }
}
```

**3. GitHub 노드 — 파일 최초 생성 (PUT)**

```json
{
  "method": "PUT",
  "url": "https://api.github.com/repos/{owner}/{repo}/contents/{path}",
  "headers": {
    "Authorization": "Bearer {{ $env.GITHUB_TOKEN }}",
    "Content-Type": "application/json"
  },
  "body": {
    "message": "docs: add TIL {{ $now.format('YYYY-MM-DD') }}",
    "content": "{{ $base64Encode($json.markdownContent) }}"
  }
}
```

> **최초 생성 시**: `sha` 필드 없이 PUT 요청하면 신규 파일로 생성된다.
> **수정 시**: 기존 파일의 `sha` 값을 먼저 GET으로 조회한 뒤 포함해야 한다.

**4. n8n Code 노드 — 파일 경로 동적 생성**

```javascript
const today = new Date().toISOString().split('T')[0]; // 2026-04-29
const title = $input.first().json.title
  .replace(/\s+/g, '-')
  .replace(/[^\w\-가-힣]/g, '');
const folder = $input.first().json.folder || 'etc';

return [{
  json: {
    path: `${folder}/${today}-${title}.md`,
    filename: `${today}-${title}.md`
  }
}];
```

## 💡 Insight
- **최초 생성 vs 업데이트**: GitHub Contents API는 PUT 메서드를 공유하지만, `sha` 포함 여부로 생성/수정이 구분된다. 최초 작성 플로우에서는 `sha`를 생략하면 된다.
- **Base64 인코딩 필수**: GitHub API는 파일 내용을 반드시 Base64로 인코딩하여 전송해야 한다. n8n의 `$base64Encode()` 헬퍼를 활용하면 별도 코드 없이 처리 가능하다.
- **트리거 다양화**: Webhook 외에도 n8n Form, Telegram Bot, Slack Slash Command 등으로 트리거를 교체하면 모바일에서도 빠르게 TIL을 기록할 수 있다.
- **AI 보강 연계**: Claude API를 중간에 삽입하면 단편적인 메모도 완결성 있는 TIL 문서로 자동 변환되어 문서 품질이 일관되게 유지된다.
- **한계**: n8n 셀프호스팅 환경에서는 GitHub Token과 API Key를 환경변수로 안전하게 관리하는 것이 중요하며, 클라우드 n8n 사용 시 크레딧 소모에 주의해야 한다.