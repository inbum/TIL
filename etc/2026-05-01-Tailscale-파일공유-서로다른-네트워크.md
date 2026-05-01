# Tailscale 활용 서로 다른 네트워크에서 파일 공유하기

## 📌 Context
집, 회사, 카페 등 서로 다른 네트워크 환경에 있는 장치들 간에 파일을 공유해야 할 때, 포트포워딩 설정이나 공인 IP 없이도 안전하게 파일을 전송하고 싶었다. Tailscale은 WireGuard 기반의 P2P VPN으로, 복잡한 네트워크 설정 없이 여러 기기를 하나의 가상 사설망으로 묶어 파일 공유를 가능하게 한다.

## ⚙️ Core

### 1. Tailscale 설치 및 로그인

```bash
# macOS
brew install tailscale
sudo tailscaled &
tailscale up

# Ubuntu/Debian
curl -fsSL https://tailscale.com/install.sh | sh
sudo tailscale up

# Windows: https://tailscale.com/download 에서 설치 후 로그인
```

### 2. Tailscale IP 확인

```bash
tailscale ip -4
# 예시 출력: 100.64.x.x (Tailscale 가상 IP)

tailscale status
# 연결된 모든 기기 목록 및 상태 확인
```

### 3. 파일 공유 방법

#### 방법 A: Taildrop (GUI 기반, 가장 간편)
```
- macOS/iOS/Android: 공유 메뉴에서 Taildrop 선택
- 수신 측: 알림에서 수락
- CLI: tailscale file cp <파일> <대상기기이름>:
```

#### 방법 B: CLI로 Taildrop 파일 전송
```bash
# 파일 보내기
tailscale file cp ./myfile.zip my-macbook:

# 파일 받기 (수신 대기 폴더 지정)
tailscale file get ~/Downloads/
```

#### 방법 C: Python HTTP 서버 + Tailscale IP
```bash
# 공유할 디렉터리에서 실행 (파일을 제공하는 쪽)
python3 -m http.server 8080

# 다른 기기에서 접근 (Tailscale IP 사용)
# 브라우저 또는 curl로:
curl http://100.64.x.x:8080/myfile.zip -O
```

#### 방법 D: rsync + Tailscale IP
```bash
# 원격 기기로 디렉터리 동기화
rsync -avz ./local-folder/ user@100.64.x.x:/remote/path/

# SSH를 통한 단일 파일 전송
scp ./report.pdf user@100.64.x.x:~/Documents/
```

### 4. Tailscale SSH 활성화 (선택)
```bash
# SSH 기능 활성화 (별도 SSH 서버 설치 불필요)
tailscale up --ssh

# 다른 기기에서 접속
ssh user@my-device-name.tail12345.ts.net
```

## 💡 Insight
- **Taildrop**은 설정이 가장 쉽고 방화벽 걱정 없이 동작하지만, 대용량 파일 전송 시 안정성은 rsync/scp 방식이 더 낫다.
- Tailscale의 가상 IP(`100.64.x.x` 대역)는 인터넷에 노출되지 않으므로, Python HTTP 서버를 열어도 Tailscale 네트워크 밖에서는 접근 불가능해 보안적으로 안전하다.
- `tailscale up --ssh`를 활성화하면 SSH 키 관리 없이 ACL 기반 접근 제어가 가능해 팀 환경에서 특히 유용하다.
- 무료 플랜 기준 최대 100대 기기 연결 가능 — 개인 사용자나 소규모 팀에는 충분하다.
- 기업 환경에서는 Tailscale의 ACL 정책으로 특정 기기 간 통신만 허용하는 제로트러스트 네트워크 구성이 가능하다.