### Jetpack Compose는?  
Jetpack Compose는 **안드로이드 네이티브 UI**를 구축하기 위한 현대적인 툴로, 기존 XML 방식과 달리 **선언형 UI**를 제공합니다.  

### **Jetpack Compose의 특징**  

#### 1. **선언형 UI**  
- 기존 XML 방식과 달리 **UI 상태를 직접 변경할 필요 없이**, 상태 변화에 따라 **자동으로 UI가 업데이트**됩니다.  
- `ViewBinding` 없이 **Activity와 직접 연결할 필요가 없습니다.**  

#### 2. **Kotlin 기반**  
- 모든 UI 코드가 **Kotlin 코드로 동작**하며, **XML을 사용할 필요가 없습니다.**  
- UI를 **함수, 선언형 스타일로 구성**하여 보다 간결하고 직관적인 코드 작성이 가능합니다.  

#### 3. **핫 리로드 지원**  
- 코드 수정 후 **즉시 반영되는 프리뷰 기능**을 제공하여 빠르게 UI를 확인할 수 있습니다.  
- 기존 XML 대비 **개발 속도가 크게 향상**됩니다.  

#### 4. **UI 코드의 재사용성**  
- UI를 **함수처럼 재사용 가능**하여 유지보수와 확장성이 뛰어납니다.  
- `@Composable` 함수로 **재사용 가능한 UI 컴포넌트를 쉽게 작성**할 수 있습니다.  

#### 5. **Jetpack Navigation과의 통합**  
- 일반적으로 **하나의 Activity에서 Jetpack Navigation을 사용**하여 여러 화면을 전환하는 방식이 주로 사용됩니다.  
- Navigation Compose 라이브러리를 활용하여 **탭 네비게이션, 백 스택 관리 등을 손쉽게 구현**할 수 있습니다.  

#### 6. **성능 최적화**  
- Compose는 **레이아웃 계층을 단순화**하여 성능을 향상시킵니다.  
- 불필요한 **리컴포지션(재구성)을 최소화**하는 `remember`, `LaunchedEffect`, `derivedStateOf` 등의 API를 제공합니다.  

#### 7. **모던 UI 개발 지원**  
- `Material Design 3`을 지원하여 최신 UI 트렌드에 맞는 디자인을 쉽게 적용할 수 있습니다.  
- 애니메이션, 리스트, 제스처 등의 **직관적인 API**를 제공하여 **보다 자연스러운 UI/UX를 구현**할 수 있습니다.  

#### 8. **기존 View 시스템과의 호환**  
- `AndroidView`를 사용하면 기존 **XML 기반의 View와 Compose를 함께 사용**할 수 있습니다.  
- 점진적으로 **기존 프로젝트를 Compose로 마이그레이션**할 수 있습니다.  

---------------------------------------------

### 1. **UI 선언 방식**  
- **기존 View 시스템**: UI를 정의할 때 **XML** 파일을 사용하여 뷰 계층을 선언합니다. 각 뷰(예: `TextView`, `Button`)를 XML에 배치하고, 액티비티에서 이를 참조합니다. 이 방식은 명령형(Imperative) 프로그래밍 방식에 가깝습니다.
- **Jetpack Compose**: UI를 **Kotlin 코드**로 작성합니다. `@Composable` 어노테이션을 붙인 함수로 UI를 정의하고, 상태 변화에 따라 UI를 재구성합니다. 이 방식은 **선언형(Declarative)** 프로그래밍 방식입니다.

### 2. **UI 업데이트 방식 (Recomposition)**  
- **기존 View 시스템**: UI 업데이트가 필요할 때 명시적으로 **UI 요소를 갱신**해야 합니다. 예를 들어, 버튼을 클릭하면 `setText()`와 같은 메서드를 호출하여 UI를 갱신합니다.
- **Jetpack Compose**: UI는 상태(State)에 의해 자동으로 업데이트됩니다. 상태가 변경되면 **자동으로 UI가 재구성**(Recomposition)되며, 개발자는 이를 신경 쓸 필요가 없습니다. 예를 들어, `mutableStateOf`로 상태를 관리하고, 해당 상태가 변경되면 UI가 자동으로 갱신됩니다.

### 3. **뷰 계층 구조 및 관리**  
- **기존 View 시스템**: `Activity`나 `Fragment`에 **View**를 추가하고 관리하는 방식입니다. UI는 **중첩된 뷰 트리**를 구성하고, 각 뷰는 별도로 관리됩니다.
- **Jetpack Compose**: UI는 **Compose Runtime**에서 경량화된 **컴포지션 트리**로 관리됩니다. 각 컴포저블 함수가 UI 요소를 정의하며, 이들이 서로 **합성(Composition)**되어 화면을 구성합니다.

### 4. **상태 관리**  
- **기존 View 시스템**: UI 요소의 상태는 **명시적으로 관리**해야 합니다. 예를 들어, `LiveData`, `ViewModel` 또는 `findViewById` 등을 사용하여 UI 상태를 관리합니다.
- **Jetpack Compose**: Compose는 상태가 **변경될 때 자동으로 UI를 갱신**합니다. `remember`, `mutableStateOf`, `derivedStateOf` 등을 사용해 **선언적으로 상태를 관리**합니다. 상태가 변경되면 해당 상태와 연관된 UI 부분만 **재구성(Recomposition)**됩니다.

### 5. **코드 재사용성**  
- **기존 View 시스템**: XML에서 정의한 레이아웃은 재사용이 어려운 부분이 있습니다. 예를 들어, 하나의 레이아웃을 여러 액티비티에서 사용하려면 별도의 레이아웃 파일을 만들어야 합니다.
- **Jetpack Compose**: UI를 구성하는 요소를 **Composable 함수로 추상화**할 수 있어, 코드의 재사용이 더 용이합니다. 동일한 **Composable 함수**를 여러 곳에서 사용하여 UI를 재사용할 수 있습니다.

### 6. **성능**  
- **기존 View 시스템**: XML 파싱과 뷰 계층의 관리를 위해 시스템 리소스를 많이 사용하고, **뷰 트리가 복잡해질수록 성능이 저하**될 수 있습니다.
- **Jetpack Compose**: UI는 **Composition 트리**로 관리되어, 변경이 필요한 부분만 업데이트되므로 성능이 더 뛰어난 경우가 많습니다. **불필요한 Recomposition을 피하고, 효율적으로 UI를 갱신**할 수 있습니다.

### 7. **디버깅 및 테스트**  
- **기존 View 시스템**: UI를 변경하려면 이벤트를 처리하고 뷰 계층을 추적하는 방식으로 디버깅합니다.
- **Jetpack Compose**: **UI의 선언적 특성** 덕분에 디버깅이 훨씬 간단하고, **UI 테스트**도 더 직관적으로 작성할 수 있습니다. `ComposeTestRule` 등을 활용하여 컴포넌트 단위로 테스트할 수 있습니다.

### 8. **호환성**  
- **기존 View 시스템**: 기존의 **View 기반 시스템**과 **혼합하여** 사용할 수 있습니다. 예를 들어, `Fragment`와 `Activity`에서 **Compose UI**를 삽입하거나, Compose에서 **기존 View**를 삽입하는 방식으로 호환이 가능합니다.
- **Jetpack Compose**: Compose는 완전히 **새로운 UI 시스템**이지만, 기존 `XML 기반 UI`와 **혼합해서 사용할 수 있는 호환성**을 제공합니다.

### 9. **라이프사이클 관리**  
- **기존 View 시스템**: `Activity`, `Fragment`의 생명주기를 관리해야 하며, 이 과정에서 UI 상태를 적절히 유지해야 합니다.
- **Jetpack Compose**: Compose는 **자체적인 상태 관리**가 내장되어 있어, **Recomposition**을 통해 화면 전환, 상태 변화 등을 자연스럽게 처리합니다.

---------------------------------------------

### **Composable 함수란?**  
`@Composable` 함수는 **Jetpack Compose가 안드로이드 네이티브 UI와 결합할 수 있는 함수**입니다.  
- **재사용이 가능**하며, 일반 함수처럼 **파라미터를 받아서 UI를 그릴 수 있습니다.**  
- 함수의 **반환값이 없으며**, UI를 직접 그리는 역할을 합니다.  

---

### **Composable 함수의 유형**  

#### 1. **Stateless Composable** (무상태 Composable)  
- **함수 내에 상태를 가지지 않고**, **파라미터로 상태를 전달받아 UI를 구성**하는 함수입니다.  
- 재사용성이 높고, 외부에서 상태를 관리할 수 있어 **테스트와 유지보수가 용이**합니다.  
- 예시:  
  ```kotlin
  @Composable
  fun Greeting(name: String) {
      Text(text = "Hello, $name!")
  }
  ```
  - `name`을 외부에서 받아 UI를 구성하므로 **Stateless Composable**입니다.  

#### 2. **Stateful Composable** (상태 저장 Composable)  
- 함수 내부에서 `remember`나 `mutableStateOf`를 사용하여 **자체적으로 상태를 관리**하는 Composable 함수입니다.  
- UI 내부적으로 상태를 변경할 필요가 있을 때 사용됩니다.  
- 예시:  
  ```kotlin
  @Composable
  fun Counter() {
      var count by remember { mutableStateOf(0) }

      Column {
          Text(text = "Count: $count")
          Button(onClick = { count++ }) {
              Text("Increase")
          }
      }
  }
  ```
  - `count`라는 변수를 내부에서 직접 관리하므로 **Stateful Composable**입니다.  

---
