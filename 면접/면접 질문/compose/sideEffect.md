compose 에서의 sideEffect 

compose 는 기본적을로 순수 험수 이다 
입력값에 따라서 하나의 ui 출력값을 출력한다
그러나 특정 상황에서는 sideEffect가 필요할수도 있다 
예시토 토스트를 띄우거나, 스낵바를 표시하거나 특정 상테에서 화면을 이동하는것 같은 일회성 이벤트를 트리거할때 필요한데
이러한 sideEffect를 컴포저블 안에서 처리하면 sideEffect가 리컴포지션때문에 원하지 않게 작동할수 있다

그래서 컴포저블의 생명주기를 인식하는 compose 가 재공하는 Effect api 를 사용하여 예측 가능한 동작응 보장할수 있다
이 Effect api 는 ui 를 내보내지 않는 컴포저블 함수이다

 LaunchedEffect 정리

LaunchedEffect는 비동기 작업을 수행할 때 사용되는 컴포즈의 부수 효과 API이다.

첫 번째 Composition → 실행됨.
Key 값 변경 → 기존 코루틴이 취소되고, 새로운 코루틴 실행.
컴포저블이 UI 트리에서 제거되면 → 자동으로 코루틴 취소.

첫 컴포지션에서 실행됨.
key 값이 변경될 때마다 기존 코루틴을 취소하고 새 코루틴을 실행함.
UI 트리에서 사라지면 자동으로 취소됨.
내부적으로 rememberCoroutineScope()를 사용하지 않아도 코루틴 관리가 자동으로 이루어짐.

다시 돌아보면 일반적인 컴포저블 함수처럼 첫컴포지션에는 동작하고 상태변경이 없으면 리컴포지션을 건너뛰고 상태변경시에 리컴포지션을 실행한다 당연하다

rememberCoroutineScope 정리

rememberCoroutineScope는 LaunchedEffect와 다르게 호출돠는 컴포지션에 바인딩된 coroutineScope 를 반환한다
이 코루틴 스코프는 onClick 의 람다같은 컴포저블 함수가 아닌곳에서도 사용할수 있다

rememberUpdatedState 정리

  remember로 저장된 값이 재구성 시 최신 값으로 갱신되도록 해, 부수 효과(예: LaunchedEffect)나 코루틴이 항상 최신 값을 사용하도록 보장한다.
  remember는 첫 컴포지션 때만 초기화되므로, 이후 값 변경이 부수 효과에 반영되지 않는 문제를 해결한다.
  효과를 재실행하지 않고, 타이머, 네트워크 요청, 애니메이션 등에서 최신 값을 반영해야 할 때 유용하다.

  ```kotlin
@Composable
fun LandingScreen(onTimeout: () -> Unit) {
    // onTimeout의 최신 값을 항상 반영하도록 업데이트된 참조 생성
    val currentOnTimeout by rememberUpdatedState(onTimeout)

    // LaunchedEffect는 key가 변하지 않으므로 한 번만 실행되지만,
    // 내부에서는 currentOnTimeout()을 호출하여 항상 최신 onTimeout을 사용
    LaunchedEffect(true) {
        delay(3000) // 3초 대기
        currentOnTimeout() // 최신 onTimeout 호출
    }
    
    /* LandingScreen UI 구성 요소 */
}
```
rememberUpdatedState내부 코드
```kotlin
@Composable
fun <T> rememberUpdatedState(newValue: T): State<T> = remember {
    mutableStateOf(newValue)
}.apply { value = newValue }
```

DisposableEffect 정리

DisposableEffect는 key 값과 실행블록, onDispose블록을 받는다

실행블록은 첫 컴포지션시 실행되고 
onDispose 블록은 key 값이 바뀌거나 DisposableEffect가 uitree에서 사라질때 실행된다

보통 리스너 등록, 들록 해재및 리소스 정리에 사용한다

SideEffect 정리 

SideEffect 는 함수 블럭 하나만 받고 
컴포지션 시마다 함수블럭을 실행한다

Compose 안의 상태를 비 Compose 코드에 전달할때 사용하기 좋다

produceState 정리


produceState 는 initialValue(선택), key값, LaunchedEffect 안에서 실행되는 반환값을 가진함수를 받는다 
Composable 내에서 비동기 작업을 하고 그 결과를 State타입으로 반환하는 함수이다

```kotlin 
@Composable
fun Timer(): State<Int> {
    // 초기값 0으로 설정하고, 비동기 작업을 통해 값을 업데이트
    return produceState(initialValue = 0) {
        while (true) {
            delay(1000)
            value++  // 매 초마다 value를 증가시키며 State 업데이트
        }
    }
}
```

derivedStateOf 정리

derivedStateOf는 상태가 실제로 ui 업데이트 돼어야하는것보다 더 자주 바뀔때 사용한다
derivedStateOf는 계산식 람다를 받고 실제로 값이 변경되엇을때 State 형식으로 반환한다
```kotlin
@Composable
// When the messages parameter changes, the MessageList
// composable recomposes. derivedStateOf does not
// affect this recomposition.
fun MessageList(messages: List<Message>) {
    Box {
        val listState = rememberLazyListState()

        LazyColumn(state = listState) {
            // ...
        }

        // Show the button if the first visible item is past
        // the first item. We use a remembered derived state to
        // minimize unnecessary compositions
        val showButton by remember {
            derivedStateOf {
                listState.firstVisibleItemIndex > 0
            }
        }

        AnimatedVisibility(visible = showButton) {
            ScrollToTopButton()
        }
    }
}
```
이 코드를 보면 derivedStateOf 가 없었다면 listState.firstVisibleItemIndex 가 1,2,3,4.... 일때는 listState.firstVisibleItemIndex > 0
의 값이 true 임에도 showButton 의 값이 바뀌었다고 판단해서 불필요한 리컴포징을 트리거 했을것이다
그러나 derivedStateOf를 사용해서 실제 값이 변하지 않았을때는 새로운 state 를 반환하지 않아서 불필요한 리컴포징을 줄일수 있다
주의!!! derivedStateOf는 오버해드가 큼으로 리컴포징을 줄이기 위해 사용하여야한다

snapshotFlow 정리

snapshotFlow는 반환값이 있는 함수블록을 파라미터로 받는다 기본적으로 cold Flow 를 반환하고 
파라미터로 받은 블록의 반환값이 이전값과 다를때에만 새로운 값을 반환한다(distinctUntilChanged)
