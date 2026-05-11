# ComfyUI: 노드 기반 생성형 AI GUI 플랫폼

## 📌 Context
Stable Diffusion 및 고성능 생성형 AI 모델을 GUI 환경에서 유연하게 다루기 위한 방법이 필요할 때, ComfyUI는 노드 기반 워크플로우로 복잡한 파이프라인을 시각적으로 구성할 수 있는 오픈소스 플랫폼을 제공한다. 코드 없이도 모델 로딩, 샘플링, 이미지/영상 출력까지 전체 생성 파이프라인을 커스터마이징할 수 있어 연구자 및 크리에이터 모두에게 적합하다.

## ⚙️ Core

### 설치 (로컬 환경 기준)
```bash
# 1. 저장소 클론
git clone https://github.com/comfyanonymous/ComfyUI.git
cd ComfyUI

# 2. 의존성 설치 (CUDA 환경 권장)
pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu121
pip install -r requirements.txt

# 3. 모델 배치
# models/checkpoints/ 폴더에 .safetensors 또는 .ckpt 파일 복사

# 4. 실행
python main.py
# 기본 접속: http://127.0.0.1:8188
```

### 기본 워크플로우 구성 요소 (노드)
| 노드 | 역할 |
|------|------|
| Load Checkpoint | AI 모델(.safetensors) 로드 |
| CLIP Text Encode | 프롬프트 텍스트 인코딩 |
| KSampler | 노이즈 제거 샘플링 수행 |
| VAE Decode | 잠재 공간 → 이미지 변환 |
| Save Image | 결과 이미지 저장 |

### API를 통한 워크플로우 실행 (Python)
```python
import json
import requests

def queue_prompt(workflow: dict, server_url: str = "http://127.0.0.1:8188") -> dict:
    payload = {"prompt": workflow}
    response = requests.post(f"{server_url}/prompt", json=payload)
    response.raise_for_status()
    return response.json()

# workflow.json은 ComfyUI에서 Export한 API 형식 파일
with open("workflow_api.json", "r", encoding="utf-8") as f:
    workflow = json.load(f)

result = queue_prompt(workflow)
print(result)  # {'prompt_id': '...', 'number': 1, ...}
```

## 💡 Insight
- **노드 기반 설계의 장점**: 각 처리 단계가 노드로 분리되어 있어 LoRA 적용, ControlNet 삽입, 업스케일링 등 복잡한 파이프라인을 코드 수정 없이 시각적으로 조립할 수 있다.
- **영상 생성 확장**: AnimateDiff, VideoHelperSuite 커스텀 노드를 추가하면 이미지 생성 워크플로우를 그대로 영상 생성으로 확장 가능하다.
- **API 연동 가능**: ComfyUI는 REST API를 기본 제공하므로, n8n이나 Python 스크립트로 자동화 파이프라인 구축이 용이하다.
- **주의사항**: GPU VRAM이 부족한 환경에서는 `--lowvram` 또는 `--cpu` 플래그로 실행하되, 생성 속도가 현저히 저하될 수 있다.
- **트레이드오프**: 유연성이 높은 만큼 초기 학습 곡선이 존재하며, 커스텀 노드 간 버전 충돌이 발생할 수 있어 가상 환경 관리가 중요하다.