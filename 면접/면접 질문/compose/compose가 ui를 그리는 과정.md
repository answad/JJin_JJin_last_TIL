jetpack compose가 ui를 그리는 과정은 컴포지션, 레이아웃(측정, 배치), 그리기 주요한 3가지 단계가 있다.

컴포지션 단계에서는 composable 함수를 실행하고 composeable 함수가 의미하는 ui 애 대한 자세한 설명을 만듭니다.

![image](https://github.com/user-attachments/assets/131857c3-62f5-4579-989a-a9978cc9ded2)

composable 함수 -> ui tree
레이아웃 단계에서는 ui tree 의 각 노드가 실제 핸드폰에서 어떤 크기를 가지고 어떤 위치를 차지할지 결정한다.

![image](https://github.com/user-attachments/assets/97b9bde8-800b-47ec-ad5b-74a87b8efba3)

레이아웃 단계는 측정과 배치 2단계로 나눠지는데 
컴포지션 단계에서 생성된 ui tree 를 처리할때 
하위 요소 측정: 노드의 하위요소가 있는경우에 노드가 하위 요소를 측정합니다.
자체 크기 결정: 이러한 하위 요소의 측정치를 기반으로 노드가 자체 크기를 결정합니다.
하위 요소 배치: 각 하위 노드는 노드의 자체 위치를 기준으로 배치됩니다.
이러한 측정과 배치 2단계가 끝나면 ui tree의 각 노드에는 width와 height, x,y 좌표값이 있다.
그리기 단계에서는 트리를 위에서 아래로 탐색하고 각 노드가 순서대로 화면에 그려진다.

---------

compose 가 ui 를 recompose 할때는 상태 변화가 있을때 recompose 를 하는데
위에서 말한 3 단계중 어떤 단계에서 상태 변화를 처리해야하냐에 따라서 3단계가 따로 작동한다

우선 Recomposer 는 변경한 상태를 읽는(사용하는) 모든 컴포저블 함수의 재실행을 예약한다

그러나 컴포저블 함수의 ui에 변경점이 없다면 recompose를 건너뛸수 있다 

만약 레이아웃의 변경점이 있다면 레이아웃 단계를 실행하고 측정, 배치를 실행하는데 특별히 배치 단계에서는 Modifier.offset { … } 의 람다 블록을 실행한다

그리고 레이아웃 단계에서 상태 읽기가 있다면 레이아웃에 영향을 미치고 그리기 단계도 실행시킬수 있다

그리고 레이아웃의 변경점이 있다면 그리기 단계도 실행되는데 여기서도 특별히 Canvas(), Modifier.drawBehind, Modifier.drawWithContent 를 실행한다 

그리기 단계에서의 상태 읽기는 그리기단계만 재실행한다

![image](https://github.com/user-attachments/assets/d85890f7-c764-460c-b5fa-14a84fc79e16)

다음은 이런 단계를 이용한 최적화 예시이다

컴포지션 단계
```kotlin
Box {
    val listState = rememberLazyListState()

    Image(
        // ...
        // Non-optimal implementation!
        Modifier.offset(
            with(LocalDensity.current) {
                // State read of firstVisibleItemScrollOffset in composition
                (listState.firstVisibleItemScrollOffset / 2).toDp()
            }
        )
    )

    LazyColumn(state = listState) {
        // ...
    }
}
```
레이아웃 단계
```kotlin
Box {
    val listState = rememberLazyListState()

    Image(
        // ...
        Modifier.offset {
            // State read of firstVisibleItemScrollOffset in Layout
            IntOffset(x = 0, y = listState.firstVisibleItemScrollOffset / 2)
        }
    )

    LazyColumn(state = listState) {
        // ...
    }
}
```


위의 두 코드는 lazyColumn 의 스크롤 상태에 따라 이미지의 위치를 변경하는 코드이다 

첫번쩨 코드는 
```kotlin
Modifier.offset(
            with(LocalDensity.current) {
                // State read of firstVisibleItemScrollOffset in composition
                (listState.firstVisibleItemScrollOffset / 2).toDp()
            }
        )
```
compose가 firstVisibleItemScrollOffset의 값이 바뀔때마다 offset에 전달하는 값이 변경되으로 컴포지션부터, 레이아웃, 그리기 단계가 실행됨다 

두번쩨코드는 offset 의 값을 직접 계산해서 넘기지 않고 계산할수 있는 람다를 넘겼다 그러므로 레이아웃 단계에서 계산이 이뤄져서 레이아웃, 그리기 단계만 실행할수 있다
