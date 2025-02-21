### Compose의 부수 효과(Side Effects) API 전체 정리

Compose는 기본적으로 순수 함수처럼 동작하여, 동일한 입력에 대해 동일한 UI를 생성합니다. 하지만 실제 앱에서는 토스트, 스낵바, 화면 전환, 네트워크 요청 등 일회성 이벤트나 비동기 작업과 같이 UI 외부의 상태를 변경하는 부수 효과가 필요할 때가 있습니다. 이런 상황에서 Compose는 컴포저블의 생명 주기를 인식하는 여러 효과 API를 제공하여 예측 가능한 동작을 보장합니다. 이들 API는 UI를 반환하지 않는 순수 부수 효과 함수들입니다.

---

#### 1. **LaunchedEffect**

- **정의:**  
  비동기 작업(예: 코루틴)을 컴포저블의 생명 주기에 맞춰 실행하는 부수 효과 API입니다.

- **특징:**  
  - **첫 번째 Composition 시 실행:**  
    컴포저블이 처음 화면에 나타날 때 실행됩니다.
  - **Key 값 변경 시 재실행:**  
    전달된 key 값이 변경되면 기존 코루틴은 취소되고, 새 코루틴이 실행됩니다.
  - **컴포저블 제거 시 자동 취소:**  
    컴포저블이 UI 트리에서 제거되면 내부 코루틴이 자동으로 취소됩니다.
  - 내부적으로 `rememberCoroutineScope()`를 사용할 필요 없이 코루틴 관리가 자동으로 이루어집니다.

---

#### 2. **rememberCoroutineScope**

- **정의:**  
  현재 컴포저블에 바인딩된 `CoroutineScope`를 반환하는 함수입니다.

- **특징:**  
  - 컴포저블 내부뿐 아니라 이벤트 핸들러(예: 버튼 클릭) 등 컴포저블 외부에서 코루틴을 실행할 수 있습니다.
  - 반환된 코루틴 스코프는 해당 컴포저블이 Composition에서 사라지면 자동으로 취소됩니다.

- **용도:**  
  사용자 이벤트 처리, 비동기 작업 실행 등에서 직접 코루틴을 실행하고자 할 때 사용합니다.

---

#### 3. **rememberUpdatedState**

- **정의:**  
  remember로 저장된 값이 재구성(Recomposition) 시에도 최신 값으로 업데이트되어, 부수 효과(예: LaunchedEffect)나 코루틴에서 항상 최신 값을 참조하도록 보장하는 함수입니다.

- **필요성:**  
  - 기본 remember는 첫 번째 컴포지션 시 초기값만 저장되므로, 이후 값 변경이 부수 효과 내에 반영되지 않는 문제를 해결합니다.
  - 효과를 재실행하지 않고도 최신 값을 사용할 수 있게 해, 타이머, 네트워크 요청, 애니메이션 등에서 유용합니다.

- **예제:**
  ```kotlin
  @Composable
  fun LandingScreen(onTimeout: () -> Unit) {
      // onTimeout의 최신 값을 반영하도록 업데이트된 참조 생성
      val currentOnTimeout by rememberUpdatedState(onTimeout)

      // LaunchedEffect는 key가 변하지 않으므로 한 번만 실행되지만,
      // 내부에서는 currentOnTimeout()을 호출하여 항상 최신 onTimeout을 사용
      LaunchedEffect(true) {
          delay(3000) // 3초 대기
          currentOnTimeout()
      }
      
      // LandingScreen UI 구성 요소
  }
  ```
  
- **내부 구현 예:**
  ```kotlin
  @Composable
  fun <T> rememberUpdatedState(newValue: T): State<T> = remember {
      mutableStateOf(newValue)
  }.apply { value = newValue }
  ```

---

#### 4. **DisposableEffect**

- **정의:**  
  컴포저블의 수명 주기에 따라 부수 효과를 생성하고, 해당 컴포저블이 UI 트리에서 제거되거나 key 값이 변경될 때 정리(cleanup) 작업을 수행하는 API입니다.

- **특징:**  
  - 첫 번째 Composition 시 효과가 실행되고, onDispose 블록에서 리소스 정리 작업(예: 리스너 해제, 자원 반환)을 수행합니다.
  - 컴포저블이 제거되면 자동으로 onDispose가 호출되어 효과를 정리합니다.

- **용도:**  
  센서 리스너 등록 및 해제, 리소스 관리 등 외부 자원과의 연결을 정리할 때 사용합니다.

- **예제:**
  ```kotlin
  @Composable
  fun SensorScreen() {
      DisposableEffect(Unit) {
          // 효과 시작: 센서 리스너 등록
          val sensorManager = SensorManager()
          val listener = object : SensorEventListener {
              override fun onSensorChanged(event: SensorEvent) {
                  // 센서 이벤트 처리
              }
              override fun onAccuracyChanged(sensor: Sensor?, accuracy: Int) { }
          }
          sensorManager.registerListener(listener)

          // onDispose: 컴포저블이 제거되면 리스너 해제
          onDispose {
              sensorManager.unregisterListener(listener)
          }
      }

      Text("Sensor Screen")
  }
  ```

---

#### 5. **produceState**

- **정의:**  
  비동기 작업의 결과를 State로 관리하기 위해 사용하는 API입니다.

- **특징:**  
  - 초기값과 함께 시작하여, 내부 코루틴에서 값을 업데이트하면 해당 State가 자동으로 UI에 반영됩니다.
  - 컴포저블의 생명 주기에 맞춰 관리되며, 컴포저블이 제거되면 관련 코루틴도 취소됩니다.

- **용도:**  
  타이머, 네트워크 요청, 데이터베이스 쿼리 등 비동기 작업의 결과를 UI와 연동할 때 유용합니다.

- **예제:**
  ```kotlin
  @Composable
  fun Timer(): State<Int> {
      return produceState(initialValue = 0) {
          while (true) {
              delay(1000)
              value++  // 매 초마다 value 증가 및 UI 업데이트
          }
      }
  }
  ```

---

#### 6. **derivedStateOf**

- **정의:**  
  여러 상태 값을 기반으로 새로운 상태를 계산하여, 실제 UI 업데이트가 필요한 경우에만 값이 재계산되도록 최적화하는 API입니다.

- **특징:**  
  - 내부에서 사용된 상태 값이 변경될 때만 새 값을 계산하며, 그렇지 않으면 캐시된 값을 반환합니다.
  - 불필요한 리컴포지션을 줄여 성능 최적화에 도움을 줍니다.

- **용도:**  
  계산 비용이 높은 값이나, 상태 변화가 잦지만 실제 UI 변경이 필요하지 않은 경우 사용합니다.

- **예제:**
  ```kotlin
  @Composable
  fun MessageList(messages: List<Message>) {
      val listState = rememberLazyListState()
      val showButton by remember {
          derivedStateOf { listState.firstVisibleItemIndex > 0 }
      }
      LazyColumn(state = listState) {
          // 메시지 목록 구성
      }
      AnimatedVisibility(visible = showButton) {
          ScrollToTopButton()
      }
  }
  ```

---

#### 7. **snapshotFlow**

- **정의:**  
  컴포저블 내부의 상태 값을 관찰하여, 그 상태의 스냅샷을 Flow로 변환하는 API입니다.

- **특징:**  
  - 반환값이 있는 함수 블록을 인자로 받아, 해당 블록의 결과를 Flow에 내보냅니다.
  - 기본적으로 cold Flow를 반환하며, 구독이 시작될 때만 값을 계산합니다.
  - 함수 블록의 반환값이 이전 값과 다를 때에만 새로운 값을 내보내므로 `distinctUntilChanged` 효과를 가집니다.

- **용도:**  
  Compose 상태의 변화를 Flow로 감지하여 비동기 스트림 처리나 코루틴 기반 연산에 활용할 때 유용합니다.

---
