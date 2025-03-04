Compose에서 layout을 사용하면  AlignmentLine(정렬 기준선)을 이용해서 커스텀 정렬선을 만들수 있다
이런 정렬선은 레이아웃에서 요소를 정렬하고 위치를 지정할때 사용한다

Row에서도 이런 정렬선을 사용한다

레이아웃에서 특정한 AlignmentLine 값을 전달하는 경우에는 레이아웃의 상위요소의 Placeable 인스턴스의 .get 연산자를 사용하여서 AlignmentLine 값을 측정하고 읽어올수있다
Compose의 일부 컴포저블에서는 기본적으로 AlignmentLine을 재공하는 Composable도 있다 

커스텀 Layout과  LayoutModifier를 만들떄 커스텀 AlignmentLine을 사용할수 있다

커스텀 AlignmentLine을 만드는 방법은

HorizontalAlignmentLine(merger: (Int, Int) -> Int)
VerticalAlignmentLine(merger: (Int, Int) -> Int)

이런 2가지의 함수를 
사용하는것이다 merger의 역할은 "Jetpack Compose에서 서로 다른 자식 요소들에 의해 정의된 정렬선 값들을 병합하는 방법" 이라고 하지만 이것말고는 정보가없다....
일단 나는 자식 요소들이 여러 기준선을 재공하는데 그러한 값을 병합하는 방법이라고 이해했다....

