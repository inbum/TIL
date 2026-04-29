# TIL README.md 자동 업데이트 (JavaScript)

## 📌 Context
TIL(Today I Learned) 프로젝트를 운영하다 보면, 새로운 문서를 추가할 때마다 README.md를 수동으로 업데이트하는 반복 작업이 발생합니다. Node.js를 활용해 폴더 구조를 자동으로 스캔하고 README.md를 갱신하는 스크립트를 작성함으로써 이 과정을 자동화했습니다.

## ⚙️ Core

### README.md 기본 구조

```markdown
# TIL (Today I Learned)

> 매일 배운 것들을 기록하는 저장소입니다.

## 카테고리
- [AI](./ai)
- [n8n](./n8n)
- [Python](./python)
- [JavaScript](./javascript)
- [etc](./etc)

## 최근 TIL
<!-- TIL_LIST -->
<!-- /TIL_LIST -->
```

### JavaScript 자동 업데이트 스크립트 (`update-readme.js`)

```javascript
const fs = require('fs');
const path = require('path');

const ROOT_DIR = path.resolve(__dirname);
const README_PATH = path.join(ROOT_DIR, 'README.md');
const EXCLUDED = ['node_modules', '.git', '.github'];

function getTILFiles(category) {
  const fullPath = path.join(ROOT_DIR, category);
  if (!fs.existsSync(fullPath)) return [];

  return fs.readdirSync(fullPath)
    .filter(f => f.endsWith('.md') && f !== 'README.md')
    .map(file => ({
      category,
      path: `./${category}/${file}`,
      title: file.replace(/^\d{4}-\d{2}-\d{2}-/, '').replace(/-/g, ' ').replace('.md', ''),
      date: file.match(/^(\d{4}-\d{2}-\d{2})/)?.[1] || ''
    }));
}

function updateReadme() {
  const categories = fs.readdirSync(ROOT_DIR).filter(f => {
    const fullPath = path.join(ROOT_DIR, f);
    return fs.statSync(fullPath).isDirectory() && !EXCLUDED.includes(f);
  });

  const allEntries = categories
    .flatMap(getTILFiles)
    .sort((a, b) => b.date.localeCompare(a.date));

  const tilList = allEntries
    .slice(0, 20)
    .map(e => `- [${e.date} | ${e.category} | ${e.title}](${e.path})`)
    .join('\n');

  let readme = fs.readFileSync(README_PATH, 'utf-8');
  readme = readme.replace(
    /<!-- TIL_LIST -->[\s\S]*?<!-- \/TIL_LIST -->/,
    `<!-- TIL_LIST -->\n${tilList}\n<!-- /TIL_LIST -->`
  );

  fs.writeFileSync(README_PATH, readme, 'utf-8');
  console.log(`README.md updated: ${allEntries.length} entries found.`);
}

updateReadme();
```

### GitHub Actions 연동 (`.github/workflows/update-readme.yml`)

```yaml
name: Update README

on:
  push:
    branches: [main]

jobs:
  update:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '18'
      - run: node update-readme.js
      - run: |
          git config user.email "action@github.com"
          git config user.name "GitHub Action"
          git add README.md
          git diff --cached --quiet || git commit -m "docs: auto-update README"
          git push
```

## 💡 Insight
- `<!-- TIL_LIST -->` / `<!-- /TIL_LIST -->` 마커 사이만 교체하므로 README의 나머지 내용은 보존됩니다.
- 날짜 기준 역순 정렬로 최신 TIL이 항상 상단에 노출됩니다.
- `.slice(0, 20)` 제한으로 README 비대화를 방지하며, 전체 목록은 각 카테고리 폴더에서 확인합니다.
- 카테고리 폴더가 추가되더라도 동적 스캔 방식이므로 스크립트 수정 없이 확장됩니다.
- GitHub Actions와 결합하면 파일 push 즉시 README가 자동 갱신되어 수동 관리 비용이 제거됩니다.