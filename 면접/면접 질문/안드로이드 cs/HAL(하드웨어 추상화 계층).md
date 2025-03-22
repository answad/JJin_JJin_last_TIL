HAL(Hardware Abstraction Layer

HAL(할)은 Android에서 하드웨어와 상위 애플리케이션 프레임워크(Framework API) 사이에 위치해서, 하드웨어 기능을 일관되고 표준화된 인터페이스로 제공하는 계층이다

HAL은 애플리케이션이나 프레임워크가 특정 하드웨어 구현에 종속되지 않도록 추상화를 하고, 하드웨어 제조업체가 새로운 하드웨어를 Android 시스템이 사용할수 있도록 도운다

---

HAL의 역할은 애플리케이션과 하드웨어 간의 interface(하드웨어의 구현을 추상화하는), 중개자 역할을 하고
하드웨어별로 다른 드라이버 및 펌웨어를 단일 인터페이스로 추상화 하드웨어 변경이 있어도 Android 프레임워크, 애플리케이션 코드 변경이 없거나 최소화된다
Android 시스템이 필요할 때만 해당 특정 하드웨어의 모듈을 로드해서ㅓ 성능 최적화를 한다  

---

HAL의 동작 흐름을 설명하기 위해 카메라 앱에서 사진을 촬영하는 과정을 설명했다

1 애플리케이션
사용자가 카메라 앱을 실행하고 사진 촬영 버튼을 누른다

2 Framework API (Java/Kotlin 레벨)
`CameraManager` → `CameraDevice` 같은 Android 프레임워크 API가 해당 요청을 수신하고, Native 레벨의 HAL에 요청을 전달한다

3 JNI (Java Native Interface)
Framework API는 JNI를 통해 Native 레벨의 C/C++ HAL 인터페이스를 호출

4 HAL (Hardware Abstraction Layer)
HAL은 요청을 받아서 실제 카메라 하드웨어와 연결된 드라이버에 전달한다
  
5 드라이버 & 하드웨어
카메라 모듈이 사진을 촬영하고, 그 결과를 HAL을 통해 상위로 전달한다

6 Framework API → 애플리케이션
Framework API가 촬영된 이미지를 애플리케이션으로 반환하여 화면에 표시한다

---

HAL의 구성: HAL은 여러 개의 라이브러리 모듈로 구성되며, 각 모듈은 특정 하드웨어를 담당한다

| HAL 모듈 | 설명 ||---|---|
| **Camera HAL** | 카메라 모듈과 상호 작용하는 인터페이스 |
| **Bluetooth HAL** | Bluetooth 모듈과 통신 |
| **Audio HAL** | 마이크 및 스피커 관련 오디오 기능 지원 |
| **Sensor HAL** | 가속도계, 자이로스코프, 조도 센서 등과 상호 작용 |
| **Wi-Fi HAL** | Wi-Fi 모듈과 연결 |
| **Graphics HAL** | OpenGL ES 및 하드웨어 가속 렌더링 관련 기능 제공 |

---

HAL 인터페이스 구현 방식에는 3가지가 있다

1. Legacy HAL
   - 특징: 초기 Android에서 사용된 방식으로, HAL 인터페이스가 C/C++ 기반의 구조체로 정의되었고, 시스템은 이를 직접 로드하여 하드웨어와 통신했다
   - 히스토리: 시스템과 하드웨어가 직접 연결되어 있어 간단하게 구현할 수 있었지만, 하드웨어와 시스템 간 결합도가 높아지면서 다양한 하드웨어 플랫폼에 대응하기 어려워졌고, 확장성과 유지보수가 힘들어졌다
   - 예제 코드:
     C++ 기반으로 인터페이스를 정의한다
     ```cpp
     // Camera HAL 인터페이스 정의
     struct camera_module_t {
         struct hw_module_t common;
         int (*get_number_of_cameras)();
         int (*get_camera_info)(int camera_id, struct camera_info *info);
     };
     ```

2. HIDL (HAL Interface Definition Language)
   - 특징 Android 8 (Oreo)에서 도입되어, HAL과 시스템 서비스를 분리하여 보다 유연하고 안전하게 하드웨어를 추상화할 수 있게 했다
   - 히스토리 HIDL은 HAL 구현을 시스템 서비스와 분리하여 하드웨어와의 상호작용을 안전하고 관리하기 쉬운 방식으로 만들었다
   - 이를 통해 시스템과 하드웨어 간 의존성이 줄어들었고, 다양한 하드웨어에 대한 지원을 쉽게 확장할 수 있게 되었다
   - 예제 코드
     HIDL 자체가 인터페이스 정의 언어이다
     ```hidl
     interface CameraDevice {
         open();
         close();
         captureImage();
     };
     ```

3. AIDL (Android Interface Definition Language)
   - 특징 Android 10부터 지원되며, HIDL보다 직관적이고 사용하기 쉬운 방식으로, 프로세스 간 통신(IPC)을 지원하여 하드웨어 액세스를 효율적으로 처리할 수 있다
   - 히스토리 AIDL은 직관적인 인터페이스 정의 방식으로 개발자 친화적이며, 프로세스 간 데이터 전송을 간단하게 처리할 수 있다
   - 이를 통해 하드웨어 자원에 접근하는 데 필요한 복잡성을 줄이고 성능을 향상시킬 수 있다
   - HIDL보다 더 쉽게 사용할 수 있어 개발자가 빠르게 구현할 수 있는 장점이 있다
   - 예제 코드
    AIDL도 인터페이스 정의 언어이다
     ```aidl
     interface CameraService {
         Camera openCamera(int cameraId);
         void closeCamera(Camera camera);
     };
     ```
