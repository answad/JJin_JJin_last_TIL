공식문서에 뜬금없게 한장이 pager를 설명하는 문서가 있다

Compose의 페이저 
pager는 팩의 페이지를 넘기듯이 한장한장 넘길수있는 Composable 함수이다 
종류는 VerticalPager와 HorizontalPager가 있고
VerticalPager는 위아래로 스크롤할수있고 좌우넓이를 최대로 차지한다
HorizontalPager는 좌우로 스크롤할수있고 상하넓이를 최대로 차지한다

Pager의 각 페이지는 실제로 화면에 표시될때 지연 컴포지션하고 레이아웃된다
화면에 보이지 않는 페이지도 로딩해야할때는 beyondBoundsPageCount를 1이상으로 설정하면 된다

페이저에서 특정 index로 스크롤할수있는데 

```kotlin
  coroutineScope.launch {
        pagerState.scrollToPage(5)
    }
```
coroutineScope와 scrollToPage를 이용하는 방법이다

하지만 이 방법은 애니메이션이 없어서 애니메이션을 넣고싶으면 animateScrollToPage를 사용해라 

PagerState는 페이지에 관항 정보를 가진 3개의 속성이있다

currentPage: 현제 중심적으로 보여주고있는 페이지의 index + 1
settledPage: 애니메이션이나 스크롤이 실행되지 않는 안정된 페이지 index + 1
targetPage: 스크롤 동작이 끝났을떄 pager내부에서 스스로 이동하고있는 page index + 1

 PagerState.currentPageOffsetFraction 

 0f 부터 1f 까지의 숫자이고 현재 패이지가 중심에서 얼마나 멀어졌는지를 반환한다
1f는 페이지가 완전한상태 중심에 있는상태
0f는 페이지가 완전히 사라졌을떄

 HorizontalPager와 VerticalPager는 상하또는 좌우 넓이를 최대로 차지한다
 그렇게 하고싶지 않은경우에는 pageSize 값을 Fixed로 직접 설정해줄수 있다 

 contentPadding 

 pager 내부에서 각 content를 감싸는 padding을 설정할수 있다


 pagerSnapDistance  기본적으로 pager는 한번의 스크롤로 한 페이지씩만 넘길수 있게 하는게 변경하려면 
    pagerSnapDistance = PagerSnapDistance.atMost(10)
이런식으로 설정해주면 된다ㅏ
