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
