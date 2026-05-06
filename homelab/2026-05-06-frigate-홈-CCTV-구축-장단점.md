# Frigate + 스마트폰 + 노트북으로 홈 CCTV 구축 장단점

## 📌 Context

상용 CCTV 솔루션 대신 오픈소스 NVR인 Frigate와 유휴 기기(스마트폰, 노트북)를 활용해 비용 효율적인 홈 보안 시스템을 구축하는 방법을 탐색했다. Frigate는 AI 기반 객체 감지(사람, 차량 등)를 지원하며, Home Assistant와의 연동이 가능해 스마트홈 생태계에 통합하기 적합하다.

## ⚙️ Core

### 구성 요소

| 역할 | 장치 | 비고 |
|------|------|------|
| NVR 서버 | 노트북 | Frigate 실행 |
| 카메라 | 스마트폰 | IP Webcam 앱 사용 |
| 모니터링 | 스마트폰/PC | Frigate Web UI 또는 Home Assistant |

### 1. 스마트폰을 IP 카메라로 설정 (Android 기준)

```bash
# IP Webcam 앱 설치 후 스트림 URL 예시
rtsp://192.168.1.100:8080/h264_ulaw.sdp
# 또는
http://192.168.1.100:8080/video
```

### 2. 노트북에 Frigate 설치 (Docker Compose)

```yaml
# docker-compose.yml
version: "3.9"
services:
  frigate:
    container_name: frigate
    privileged: true
    restart: unless-stopped
    image: ghcr.io/blakeblackshear/frigate:stable
    shm_size: "128mb"
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - ./config:/config
      - ./storage:/media/frigate
      - type: tmpfs
        target: /tmp/cache
        tmpfs:
          size: 1000000000
    ports:
      - "5000:5000"   # Web UI
      - "8554:8554"   # RTSP 재스트리밍
    environment:
      FRIGATE_RTSP_PASSWORD: "yourpassword"
```

### 3. Frigate 카메라 설정

```yaml
# config/config.yml
mqtt:
  enabled: false  # Home Assistant 미사용 시

cameras:
  smartphone_cam:
    ffmpeg:
      inputs:
        - path: rtsp://192.168.1.100:8080/h264_ulaw.sdp
          roles:
            - detect
            - record
    detect:
      enabled: true
      width: 1280
      height: 720
      fps: 5
    record:
      enabled: true
      retain:
        days: 7
        mode: motion
    motion:
      threshold: 25

detectors:
  cpu:
    type: cpu
    num_threads: 4
```

### 4. 외부 접근 설정 (선택)

```bash
# Tailscale을 이용한 안전한 원격 접근
curl -fsSL https://tailscale.com/install.sh | sh
sudo tailscale up
# 이후 Tailscale IP로 Frigate UI 접근 가능
```

## 💡 Insight

### 장점
- **비용 절감**: 추가 하드웨어 구매 없이 유휴 기기 재활용 가능
- **데이터 주권**: 모든 영상이 로컬에 저장되어 클라우드 유출 위험 없음
- **AI 감지**: CPU만으로도 사람/차량/동물 구분 가능 (GPU 있으면 성능 대폭 향상)
- **유연한 확장**: 카메라 수 제한 없음, Home Assistant 연동으로 자동화 가능
- **오픈소스**: 지속적인 커뮤니티 업데이트, 무료

### 단점
- **전력 소비**: 노트북을 24/7 구동 시 전기세 및 배터리 열화 문제
- **스마트폰 배터리**: 충전 중 연속 사용 시 배터리 팽창/수명 단축 위험
- **네트워크 의존성**: 동일 Wi-Fi 환경 필요, 무선 연결 불안정 시 스트림 끊김
- **초기 설정 복잡도**: Docker, RTSP, YAML 설정 등 진입 장벽 존재
- **스마트폰 화각 제한**: 광각 렌즈 부재로 넓은 공간 커버 어려움
- **스토리지 관리 필요**: 장기 보관 시 디스크 용량 직접 관리해야 함

### 실무 권장 사항
- 노트북은 전원 어댑터 상시 연결 + 배터리 최대 충전량 80% 제한 설정 권장
- 스마트폰은 전용 거치대 + 상시 충전 + 배터리 보호 모드 활성화
- 중요도가 높은 구역은 전용 IP 카메라(Reolink, TP-Link Tapo 등) 추가 고려
- Google Coral USB Accelerator 추가 시 CPU 부하를 획기적으로 줄일 수 있음