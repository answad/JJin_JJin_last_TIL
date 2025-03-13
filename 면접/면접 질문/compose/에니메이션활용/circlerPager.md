[Screen_recording_20250312_153829.webm](attachment:b4b0eafc-e5b2-4ed0-a22c-e287f5beed67:Screen_recording_20250312_153829.webm)

구현 코드

```kotlin
@Composable
fun LiabilitiesPager(
    modifier: Modifier = Modifier,
    pagerState: PagerState,
    colorList: PersistentList<Color>,
) {
    val pageSpacing = 75.dp
    val rotateDegree = 15F
    val configuration = LocalConfiguration.current
    val widthWeight = 0.6F
    val pageSize = PageSize.Fixed(pageSize = (configuration.screenWidthDp * widthWeight).dp)
    val horizontalContentPadding = (configuration.screenWidthDp * (1F - widthWeight) / 2).dp

    HorizontalPager(
        modifier = modifier,
        state = pagerState,
        pageSize = pageSize,
        pageSpacing = pageSpacing,
        contentPadding = PaddingValues(horizontal = horizontalContentPadding),
    ) { page ->
        Card(
            onClick = {
                
            },
            colors = CardDefaults.cardColors(colorList[page]),
            modifier = Modifier
                .fillMaxWidth()
                .fillMaxHeight(0.5f)
                .graphicsLayer {
                    val pageOffset =
                        (pagerState.currentPageOffsetFraction + (pagerState.currentPage - page))
                    val distance = (configuration.screenWidthDp.dp / 2) - pageSpacing
                    val height = sin(Math.toRadians(rotateDegree.toDouble())) * distance.toPx()

                    translationY = (height * pageOffset.absoluteValue).toFloat()
                    rotationZ = -rotateDegree * pageOffset
                    alpha = 0.7F + (0.3F * (1F - pageOffset.absoluteValue.coerceIn(0F, 1F)))
                }
        ) {

        }
    }
}
```

우선 Pager 함수는 HorizontalPager와 VerticalPager 으로 나뉘는데 기본적으로 좌우로 이동함으로 

HorizontalPager를 사용했다

여기서 이용한 핵심 기술은`HorizontalPager`, `graphicsLayer` , `사인 함수`, `alpha` ,`pagerState.currentPageOffsetFraction`  이다.

# 설명!!

- 일단 HorizontalPager의 파라미터를 설명할것이다
    
    ```kotlin
    @Composable
    fun HorizontalPager(
        state: PagerState,
        modifier: Modifier = Modifier,
        contentPadding: PaddingValues = PaddingValues(0.dp),
        pageSize: PageSize = PageSize.Fill,
        beyondViewportPageCount: Int = PagerDefaults.BeyondViewportPageCount,
        pageSpacing: Dp = 0.dp,
        verticalAlignment: Alignment.Vertical = Alignment.CenterVertically,
        flingBehavior: TargetedFlingBehavior = PagerDefaults.flingBehavior(state = state),
        userScrollEnabled: Boolean = true,
        reverseLayout: Boolean = false,
        key: ((index: Int) -> Any)? = null,
        pageNestedScrollConnection: NestedScrollConnection =
         PagerDefaults.pageNestedScrollConnection(
            state,
            Orientation.Horizontal
        ),
        snapPosition: SnapPosition = SnapPosition.Start,
        pageContent: @Composable PagerScope.(page: Int) -> Unit
    )
    ```
    
        state: PagerState 는 pager의 변하는 상태를 관리하는 객체를 받는 파라미터이다
    
    PagerState 는 
    
    ```kotlin
    val pagerState = rememberPagerState(pageCount = { colorList.size - 1 })
    ```
    
    ### 이런 값들을 가지고 있다
    
    | 속성 이름 | 타입 | 설명 |
    | --- | --- | --- |
    | `canScrollBackward` | `Boolean` | 뒤로 스크롤 가능한지 여부 |
    | `canScrollForward` | `Boolean` | 앞으로 스크롤 가능한지 여부 |
    | `currentPage` | `Int` | 현재 페이지 (가장 가까운 스냅된 위치) |
    | `currentPageOffsetFraction` | `Float` | 현재 페이지의 스냅 위치에서의 오프셋 (범위: -0.5 ~ 0.5) |
    | `interactionSource` | `InteractionSource` | 드래그 이벤트를 처리하는 InteractionSource |
    | `isScrollInProgress` | `Boolean` | 현재 스크롤이 진행 중인지 여부 |
    | `lastScrolledBackward` | `Boolean` | 마지막으로 스크롤이 뒤로 진행되었는지 여부 |
    | `lastScrolledForward` | `Boolean` | 마지막으로 스크롤이 앞으로 진행되었는지 여부 |
    | `layoutInfo` | `PagerLayoutInfo` | `Pager`의 레이아웃 정보 |
    | `pageCount` | `Int` | 전체 페이지 수 |
    | `settledPage` | `Int` | 현재 "안착된" 페이지 |
    | `targetPage` | `Int` | 스냅하려는 목표 페이지 |
    
    파라미터 종류는 이렇게 있다
    
    | **파라미터** | **타입** | **설명** | **예시** |
    | --- | --- | --- | --- |
    | `state` | `PagerState` | 페이지의 현재 상태를 관리 (현재 페이지, 스크롤 위치 등) | `rememberPagerState()`로 생성 |
    | `modifier` | `Modifier` | 컴포저블의 모양, 크기, 동작 변경 | `Modifier.fillMaxWidth()` |
    | `contentPadding` | `PaddingValues` | 페이지 내용과 테두리 사이 간격 | `PaddingValues(16.dp)` |
    | `pageSize` | `PageSize` | 각 페이지 크기 설정 | `PageSize.Fill`, `PageSize.Fixed(200.dp)` |
    | `beyondViewportPageCount` | `Int` | 화면 밖 미리 로딩 페이지 수 | `2` |
    | `pageSpacing` | `Dp` | 페이지 사이 간격 | `8.dp` |
    | `verticalAlignment` | `Alignment.Vertical` | 페이지 내용 세로 정렬 | `Alignment.Top`, `Alignment.Bottom` |
    | `flingBehavior` | `TargetedFlingBehavior` | 휙 넘기기 동작 설정 | `PagerDefaults.flingBehavior(state = state)` |
    | `userScrollEnabled` | `Boolean` | 사용자 스크롤 가능 여부 | `true`, `false` |
    | `reverseLayout` | `Boolean` | 페이지 순서 반전 | `true`, `false` |
    | `key` | `((index: Int) -> Any)?` | 페이지 고유 키 설정 (애니메이션 부드러움) | `{ "page_$it" }` |
    | `pageNestedScrollConnection` | `NestedScrollConnection` | 스크롤 동작 세밀 제어 | `PagerDefaults.pageNestedScrollConnection(...)` |
    | `snapPosition` | `SnapPosition` | 스크롤 끝 위치 스냅 설정 | `SnapPosition.Start`, `SnapPosition.Center` |
    | `pageContent` | `@Composable PagerScope.(page: Int) -> Unit` | 각 페이지 내용 정의 | `{ page -> Text("Page $page") }` |
    
- 코드 윗부분 설명
    
    ```kotlin
    @Composable
    fun LiabilitiesPager(
        modifier: Modifier = Modifier,
        pagerState: PagerState,
        colorList: PersistentList<Color>,
    ) {
        val pageSpacing = 75.dp
        val rotateDegree = 15F
        val configuration = LocalConfiguration.current
        val widthWeight = 0.6F
        val pageSize = PageSize.Fixed(pageSize = (configuration.screenWidthDp * widthWeight).dp)
        val horizontalContentPadding = (configuration.screenWidthDp * (1F - widthWeight) / 2).dp
    
        HorizontalPager(
            modifier = modifier,
            state = pagerState,
            pageSize = pageSize,
            pageSpacing = pageSpacing,
            contentPadding = PaddingValues(horizontal = horizontalContentPadding),
        ) { page ->
    ```
    
    여기서 
    
    pagersize는 dp 단위로 설정해야해서 
    
    CompositionLocal 로받은 LocalConfiguration에서 screenWidthDp(화면의 최대 좌우 DP 값) 와  widthWeight 를 곱해서 PageSize를 구했다
    
    horizontalContentPadding 은 pager와 pageContent 사이에 패딩을 추가한다
    
    ![image.png](attachment:61390725-c8bf-4b1f-9c41-773f435d8e0e:image.png)
    
    적용전
    
    ![image.png](attachment:df713714-1c4f-4e9c-ba92-2a46e0c29203:image.png)
    
    적용후
    
    여기까지는 기본적인 pager세팅과 크게 다를것이없다
    
    [Screen_recording_20250312_181036.webm](attachment:3cfcee1c-e748-4e22-bd54-0196b3c2228e:Screen_recording_20250312_181036.webm)
    
    # 
    
- 지금부터가 핵심이다!!!
    
       
    
    ```kotlin
            Card(
                onClick = {
                    
                },
                colors = CardDefaults.cardColors(colorList[page]),
                modifier = Modifier
                    .fillMaxWidth()
                    .fillMaxHeight(0.5f)
                    .graphicsLayer {
                        val pageOffset =
                            (pagerState.currentPageOffsetFraction + (pagerState.currentPage - page))
                        val distance = (configuration.screenWidthDp.dp / 2) - pageSpacing
                        val height = sin(Math.toRadians(rotateDegree.toDouble())) * distance.toPx()
    
                        translationY = (height * pageOffset.absoluteValue).toFloat()
                        rotationZ = -rotateDegree * pageOffset
                        alpha = 0.7F + (0.3F * (1F - pageOffset.absoluteValue.coerceIn(0F, 1F)))
                    }
            ) {}
    ```
    
    이 코드는 pager 안의 코드이다 
    가장 위의 영상처럼 작동하려면 3가지 동작이 필요하다
    
    바로 회전, 높이 변경, 알파값 변경이다
    
    - 우선 회전부터 설명하겠다
        
        pagerState.currentPageOffsetFraction 이 값은 페이지가 중심에서 얼마나 벗어났는지 표현한다
        값의 범위는 0.5 → 0.0 → -0.5 이다 0.0이 중심에 있는 상태, 0.5는 오른쪽으로 절반 이동한 상태, -0.5는 왼쪽으로 절반 이동한 상태이다
        
        그리고 (pagerState.currentPage - page)는 화면에는 3개의 페이지가 보이는데
        
        1 ~ 4까지에서의 pagerState.currentPage - page값을 나타내보면 
        
        | 현재 페이지 | `page` | `(pagerState.currentPage - page)` 값 |
        | --- | --- | --- |
        | 2 | 1 | `2 - 1 = 1` |
        | 2 | 2 | `2 - 2 = 0` |
        | 2 | 3 | `2 - 3 = -1`  |
        
        currentPageOffsetFraction 값이 0.3 이라고 가정할때 
        
        | `page` 값 | `currentPageOffsetFraction` | `(pagerState.currentPage - page)` | 최종 `pageOffset` |
        | --- | --- | --- | --- |
        | 1 | 0.3 | `2 - 1 = 1` | `1 + 0.3 = 1.3` |
        | 2 | 0.3 | `2 - 2 = 0` | `0 + 0.3 = 0.3` |
        | 3 | 0.3 | `2 - 3 = -1` | `-1 + 0.3 = -0.7` |
        
        이런것때문에 
        
        ```kotlin
        (pagerState.currentPageOffsetFraction + (pagerState.currentPage - page))
        ```
        
                 이 코드가 해당 페이지가 중심위치에서 얼마나 떨어져 있는지 나타낸다
        
        그리고 rotationZ = -rotateDegree * pageOffset
        는 얼마나 중앙에서 떨어졌는지로 목표 각도와 곱해서 오른쪽 페이지에는 +각도, 왼쪽에는-각도를 적용한다 
        
        | `page` 값 | `pageOffset`  | `rotateDegree` | `적용 각도` |
        | --- | --- | --- | --- |
        | 1 | 1.0 | `15F` | -(15f) * 1.0 = -15f |
        | 2 | 0.0 | `15F` | -(15f) * 0.0 = 0 |
        | 3 | -1.0 | `15F` | -(15f) * -1.0 = 15f |
        
    - 알파
        
        알파 값은 간단하게 `pageOffset` 의 값을 이용했다
        alpha = 0.7F + (0.3F * (1F - pageOffset.absoluteValue.coerceIn(0F, 1F)))
        알파값을 최소 0.7f 로 고정하고 중앙에서 얼마나 떨어져 있냐에 따라서 0.3을 나누도록 식을 작성함
        
    - 높이
        
        높이를 설정하지 않으면 뭔가 어색하다 
        
        [Screen_recording_20250312_204538.webm](attachment:520c292b-f12a-4ace-b728-c1e3181d7e2c:Screen_recording_20250312_204538.webm)
        
        아직은 회전이 아니라 기우는 페이저다
        
        그래서 pageOffset에 따라서 높이를 조절하도록 해야한다
        
        여기서 레전드 라디안과 sin 함수가 쓰인다
        
        라디안과 sin 함수를 쓰면 특정 각도에서 이동시에 높이를 구할수 있다
        
        ### 1. **`height` 계산**
        
        ```kotlin
        val height = sin(Math.toRadians(rotateDegree.toDouble())) * distance.toPx()
        ```
        
        - `rotateDegree`가 **15도**라면, `sin(15°)` 값을 사용하여 높이를 계산한다
        - `distance`는 화면 가로 길이의 **절반 - 페이지 간격**(즉, 중앙 기준으로 얼마나 떨어져 있나).
        - `height`는 이 `distance`를 **15° 각도로 기울였을 때의 y축 이동량**(삼각함수 개념).
        
        ---
        
        ### 2. **`translationY` 적용**
        
        ```kotlin
        translationY = (height * pageOffset.absoluteValue).toFloat()
        ```
        
        - height 변경가능한 최대 높이에 pageOffset의 절댓값을 곱해서 최대높이를 화면 이동 정도에따라 적용한다……..
