# n8n을 활용한 GitHub TIL 문서 자동 작성

## 📌 Context

n8n 워크플로우를 사용하여 GitHub 레포지토리에 TIL(Today I Learned) 마크다운 파일을 자동으로 생성하고 커밋하는 파이프라인을 처음 구축함. 매번 수동으로 파일을 작성하고 push하는 번거로움을 줄이고, 입력한 학습 내용을 자동으로 포맷팅하여 GitHub에 저장하는 것이 목표.

## ⚙️ Core

### 워크플로우 구성

1. **Trigger** — Manual Trigger 또는 Webhook으로 학습 내용 수신
2. **Code Node** — 날짜 포맷팅 및 마크다운 콘텐츠 생성
3. **GitHub Node (또는 HTTP Request)** — GitHub API로 파일 커밋

### Code Node: 콘텐츠 생성

```javascript
const today = new Date().toISOString().split('T')[0]; // YYYY-MM-DD
const title = $input.first().json.title || 'til';
const body = $input.first().json.body || '';

const filename = `${today}-${title.replace(/\s+/g, '-').replace(/[^a-zA-Z0-9가-힣-]/g, '')}.md`;
const folder = $input.first().json.folder || 'etc';

const content = `# ${title}\n\n## 📌 Context\n\n${body}\n`;
const encoded = Buffer.from(content).toString('base64');

return [{ json: { folder, filename, path: `${folder}/${filename}`, encoded, today } }];
```

### GitHub Node 설정 (파일 생성)

```json
{
  "resource": "file",
  "operation": "create",
  "owner": "<GitHub 유저명>",
  "repository": "<레포명>",
  "filePath": "={{ $json.path }}",
  "fileContent": "={{ $json.encoded }}",
  "commitMessage": "docs: add TIL {{ $json.today }} - {{ $json.filename }}",
  "branch": "main"
}
```

> GitHub Node 대신 HTTP Request Node를 사용할 경우:

```
PUT https://api.github.com/repos/{owner}/{repo}/contents/{path}
Authorization: Bearer <GITHUB_TOKEN>
Content-Type: application/json

{
  "message": "docs: add TIL",
  "content": "<base64 encoded content>",
  "branch": "main"
}
```

## 💡 Insight

- **파일 중복 주의**: GitHub API는 동일 경로에 파일이 이미 존재하면 `sha` 값을 함께 전달해야 업데이트 가능. 최초 작성(create) 시에는 `sha` 불필요.
- **Base64 인코딩 필수**: GitHub Contents API는 파일 내용을 반드시 Base64로 인코딩하여 전달해야 함. n8n Code Node에서 `Buffer.from(...).toString('base64')` 사용.
- **Credentials 관리**: n8n의 GitHub credential에 `repo` 스코프 이상의 Personal Access Token(PAT) 등록 필요.
- **확장 가능성**: Webhook Trigger와 연결하면 다른 자동화(예: Claude API 연동 TIL 생성)와 쉽게 체이닝 가능.