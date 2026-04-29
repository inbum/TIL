# GPT-Image-2 모델 성능과 다양한 Usecase

## 📌 Context

OpenAI의 `gpt-image-2`는 2025년 공개된 최신 이미지 생성 모델로, DALL-E 시리즈의 후속으로 포지셔닝된다. 기존 모델 대비 프롬프트 이해력, 텍스트 렌더링, 이미지 일관성이 크게 향상되었으며, API를 통해 programmatic하게 접근 가능하다는 점에서 프로덕션 워크플로우에 통합하기 용이하다.

주요 특징:
- **고해상도 생성**: 최대 1024×1024 (standard), 1792×1024 / 1024×1792 (wide/tall) 지원
- **텍스트 렌더링**: 이미지 내 글자 표현 정확도가 이전 모델 대비 현저히 향상
- **지시 추종**: 복잡한 구성 요구사항도 높은 정확도로 반영
- **Inpainting / Editing**: 기존 이미지의 특정 영역 수정 지원

## ⚙️ Core

### 기본 이미지 생성

```python
from openai import OpenAI
import base64
from pathlib import Path

client = OpenAI()  # OPENAI_API_KEY 환경변수 필요

# 텍스트 → 이미지 생성
response = client.images.generate(
    model="gpt-image-1",  # 실제 API 모델 ID
    prompt="A photorealistic cup of coffee with latte art on a wooden table, morning light",
    size="1024x1024",
    quality="high",
    n=1,
)

# base64로 저장
image_data = base64.b64decode(response.data[0].b64_json)
Path("output.png").write_bytes(image_data)
print("이미지 저장 완료: output.png")
```

### 이미지 편집 (Inpainting)

```python
import base64
from pathlib import Path

def edit_image(image_path: str, mask_path: str, prompt: str) -> bytes:
    """
    mask: 수정할 영역을 흰색(255)으로 표시한 PNG (원본과 동일 크기, RGBA)
    """
    with open(image_path, "rb") as img, open(mask_path, "rb") as mask:
        response = client.images.edit(
            model="gpt-image-1",
            image=img,
            mask=mask,
            prompt=prompt,
            size="1024x1024",
        )
    return base64.b64decode(response.data[0].b64_json)

result = edit_image(
    image_path="photo.png",
    mask_path="mask.png",
    prompt="Replace the background with a sunset beach scene",
)
Path("edited.png").write_bytes(result)
```

### 다양한 Usecase 패턴

```python
USECASES = {
    # 1. 이커머스 상품 배경 교체
    "product_bg": "Product photo of {item} on a clean white studio background, professional lighting",

    # 2. UI 목업 / 와이어프레임 시각화
    "ui_mockup": "Mobile app UI mockup for {app_type}, clean minimal design, iOS style",

    # 3. 소셜 미디어 콘텐츠
    "social_card": "Vibrant social media post graphic with text '{headline}', modern typography, gradient background",

    # 4. 건축/인테리어 시각화
    "interior": "Photorealistic interior design of a {room_type}, {style} style, natural lighting",

    # 5. 캐릭터/마스코트 생성
    "mascot": "Cute cartoon mascot for a {brand_type} brand, friendly expression, vector style, white background",
}

def generate_from_usecase(case: str, **kwargs) -> bytes:
    prompt = USECASES[case].format(**kwargs)
    response = client.images.generate(
        model="gpt-image-1",
        prompt=prompt,
        size="1024x1024",
        quality="high",
    )
    return base64.b64decode(response.data[0].b64_json)

# 사용 예시
img = generate_from_usecase("product_bg", item="wireless earbuds")
Path("product.png").write_bytes(img)
```

## 💡 Insight

- **텍스트 렌더링 한계**: 영문 텍스트는 비교적 안정적이지만 한글·일본어 등 동아시아 문자는 여전히 오탈자·왜곡이 발생하는 경우가 있다. 텍스트 오버레이는 후처리(Pillow, Canvas API)로 별도 처리하는 것이 안전하다.
- **비용 구조 이해 필요**: 이미지 품질(`low`/`medium`/`high`)과 크기에 따라 API 비용이 크게 달라지므로, 프로토타입 단계에서는 `low` quality로 구성 검증 후 `high`로 최종 렌더링하는 2-pass 전략이 효율적이다.
- **Inpainting 마스크 품질이 결과를 결정**: 마스크 경계가 거칠면 합성 흔적이 남는다. SAM(Segment Anything Model)과 조합하면 정밀한 객체 분리 마스크를 자동 생성할 수 있어 실무 품질을 크게 높일 수 있다.
- **n8n 연동 가능성**: HTTP Request 노드로 OpenAI Images API를 호출하고, 결과 base64를 S3/Cloudinary에 업로드하는 자동화 파이프라인 구성이 가능하다. 콘텐츠 대량 생산 워크플로우에 적합.
- **브랜드 일관성 유지**: 단일 세션 내 스타일 고정은 어렵다. 동일 스타일 유지를 위해서는 `style reference` 이미지를 프롬프트에 설명적으로 포함하거나, fine-tuning(현재 gpt-image 계열 미지원)을 대기해야 한다.