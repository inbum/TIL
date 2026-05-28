# ESP32가 아두이노보다 좋은 점들 (IoT 프로젝트 관점)

## 📌 Context
아두이노(Arduino Uno/Nano 기준)는 입문용 MCU로 널리 사용되지만, 실제 IoT 프로젝트를 진행하다 보면 Wi-Fi/Bluetooth 미지원, 낮은 연산 성능, 제한된 메모리 등의 한계에 부딪히게 된다. ESP32는 이러한 한계를 극복한 MCU로, IoT 프로젝트의 요구사항을 대부분 내장 기능으로 충족시킨다.

## ⚙️ Core

### 주요 스펙 비교

| 항목 | Arduino Uno | ESP32 |
|------|-------------|-------|
| CPU | 8-bit AVR @ 16MHz | 32-bit Dual-core @ 240MHz |
| RAM | 2KB SRAM | 520KB SRAM |
| Flash | 32KB | 4MB (보드에 따라 16MB) |
| Wi-Fi | 없음 (실드 필요) | 내장 (802.11 b/g/n) |
| Bluetooth | 없음 | 내장 (BT 4.2 + BLE) |
| ADC | 10-bit x 6채널 | 12-bit x 18채널 |
| GPIO | 14개 | 34개 |
| 가격 | ~$5 (정품 기준) | ~$3~5 (모듈 기준) |

### ESP32 기본 IoT 예제 (Wi-Fi + MQTT)

```cpp
#include <WiFi.h>
#include <PubSubClient.h>

const char* ssid = "YOUR_SSID";
const char* password = "YOUR_PASSWORD";
const char* mqttServer = "broker.hivemq.com";
const int mqttPort = 1883;

WiFiClient espClient;
PubSubClient client(espClient);

void setup() {
  Serial.begin(115200);
  WiFi.begin(ssid, password);

  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println("\nWi-Fi connected: " + WiFi.localIP().toString());

  client.setServer(mqttServer, mqttPort);
  while (!client.connected()) {
    client.connect("ESP32Client");
  }
  Serial.println("MQTT connected");
}

void loop() {
  client.publish("iot/sensor", "hello from ESP32");
  client.loop();
  delay(5000);
}
```

### Deep Sleep을 활용한 저전력 설계

```cpp
#include <esp_sleep.h>

#define SLEEP_DURATION_US 10 * 1000000  // 10초

void setup() {
  Serial.begin(115200);
  // 센서 읽기 및 데이터 전송 로직
  Serial.println("Data sent. Going to deep sleep...");
  esp_sleep_enable_timer_wakeup(SLEEP_DURATION_US);
  esp_deep_sleep_start();
}

void loop() {
  // deep sleep 사용 시 loop()는 실행되지 않음
}
```

## 💡 Insight
- **Wi-Fi/BLE 내장**: 아두이노에서는 ESP8266 실드나 HC-05 모듈을 별도로 연결해야 하는 반면, ESP32는 추가 부품 없이 무선 통신이 가능하여 회로 복잡도와 비용을 동시에 줄일 수 있다.
- **듀얼 코어 활용**: FreeRTOS 기반으로 두 개의 코어를 독립적으로 사용할 수 있어, 한 코어에서 센서 데이터 수집, 다른 코어에서 네트워크 통신을 병렬 처리하는 구조가 가능하다.
- **Deep Sleep으로 배터리 연장**: 전류 소모를 약 10µA 수준까지 낮출 수 있어, 배터리 구동 IoT 디바이스 설계에 매우 유리하다.
- **주의사항**: ESP32는 3.3V 로직 레벨을 사용하므로 5V 아두이노용 센서/모듈 연결 시 레벨 시프터가 필요하다. 또한 일부 GPIO는 부팅 시 특정 상태로 설정되므로 핀 선택에 주의해야 한다.
- **결론**: 단순 LED 점멸 수준의 학습 목적이라면 아두이노가 적합하지만, Wi-Fi, BLE, 멀티태스킹, 저전력이 필요한 실제 IoT 제품 개발에는 ESP32가 사실상 표준에 가깝다.