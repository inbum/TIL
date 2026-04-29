# Replit 활용 바이브코딩으로 앱개발 하고 배포까지

## 📌 Context
Replit은 브라우저 기반의 클라우드 IDE로, 별도의 로컬 환경 세팅 없이 AI 보조 코딩(바이브코딩) 방식으로 빠르게 앱을 개발하고 배포까지 이어갈 수 있는 플랫폼이다. 바이브코딩(Vibe Coding)이란 AI의 코드 생성 능력을 적극 활용하여 개발자가 세부 구현보다 아이디어와 방향성에 집중하는 개발 방식이다. Replit AI Agent를 통해 자연어 프롬프트만으로 앱의 기본 구조부터 배포까지 원스톱으로 처리할 수 있다.

## ⚙️ Core

### 1. Replit 프로젝트 생성

```
1. replit.com 접속 후 로그인
2. [+ Create Repl] 클릭
3. 템플릿 선택 (Python Flask / Node.js / React 등)
4. Repl 이름 지정 후 생성
```

### 2. Replit AI Agent로 바이브코딩

```
# Replit AI 채팅창에 자연어로 요청
예시 프롬프트:
"간단한 TODO 앱을 만들어줘. 항목 추가, 삭제, 완료 체크 기능이 있어야 해."
```

### 3. Flask 기반 간단 앱 예시 (AI 생성 코드)

```python
from flask import Flask, render_template, request, redirect, url_for

app = Flask(__name__)
todos = []

@app.route('/')
def index():
    return render_template('index.html', todos=enumerate(todos))

@app.route('/add', methods=['POST'])
def add():
    todo = request.form.get('todo')
    if todo:
        todos.append({'text': todo, 'done': False})
    return redirect(url_for('index'))

@app.route('/toggle/<int:idx>')
def toggle(idx):
    todos[idx]['done'] = not todos[idx]['done']
    return redirect(url_for('index'))

@app.route('/delete/<int:idx>')
def delete(idx):
    todos.pop(idx)
    return redirect(url_for('index'))

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=8080)
```

### 4. Replit 배포

```
1. 우측 상단 [Deploy] 버튼 클릭
2. Deployment 타입 선택:
   - Autoscale: 트래픽에 따라 자동 스케일링 (권장)
   - Reserved VM: 항상 켜져 있는 서버
   - Static: 정적 사이트 배포
3. [Deploy Repl] 클릭
4. 배포 완료 후 *.replit.app 도메인으로 공개 접근 가능
```

## 💡 Insight
- Replit AI Agent는 프롬프트 품질에 따라 결과물 수준이 크게 달라진다. 기능 단위로 구체적으로 요청할수록 코드 완성도가 높아진다.
- 바이브코딩은 프로토타이핑 속도를 획기적으로 높여주지만, 보안 취약점이나 엣지 케이스 처리가 부족할 수 있어 실서비스 전 코드 리뷰가 필수다.
- Replit의 무료 플랜은 Repl이 일정 시간 비활성화 시 슬립 상태로 전환되므로, 상시 운영 서비스에는 유료 플랜이 필요하다.
- GitHub 연동을 통해 버전 관리 및 CI/CD 파이프라인과 연결하면 Replit을 개발·배포 허브로 더욱 효과적으로 활용할 수 있다.
- 빠른 아이디어 검증(PoC), 해커톤, 포트폴리오 프로젝트 등에 특히 유용한 접근 방식이다.