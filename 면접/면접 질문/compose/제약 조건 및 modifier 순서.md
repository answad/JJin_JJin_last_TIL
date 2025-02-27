제약 조건 및 modifier 순서 

Compose에서는 modifier로 컴포저블의 디자인과 기능을 추가할 수 있다
이러한 modifier는 크기를 정하는 컴포저블에 전달된 제약 조건에 영향을 줄 수 있다

UI 트리에서의 modifier
(modifier ui tree)[https://developer.android.com/static/develop/ui/compose/images/layouts/constraints-modifiers/modifier-wrapping.png?hl=ko]
google은 modifier를 이해시키기 위해 ui tree에서의 modifeir를 각 노드의 래퍼노드로 표시한다

컴포저블에 modifier를 2개 이상 추가하면 modifier 체인이 생성되고 이걸 modifeir 체이닝이라고 부른다
여러개의 modifeir를 추가하면 각각의 수정자 노드는 다른 수정자 노드와 내부 레이아웃 노드를 래핑한다

modifier가 추가돼어도 layout 단계에서 트리를 탐색하는 알고리즘은 같지만 각각의 modifeir 노드도 방문한다
이렇게 modifeir가 레이아웃을 변경할수 있다

레이아웃 단계의 제약조건

레이아웃 단계는 
하위 요소 측정: 노드가 하위 요소(있는 경우)를 측정합니다.
자체 크기 결정: 노드는 이러한 측정치를 기반으로 자체 크기를 결정합니다.
하위 요소 배치: 각 하위 노드는 노드의 자체 위치를 기준으로 배치됩니다.
이 3단계로 레이아웃의 높이 너비 좌표를 결정한다

제약(Constraints)은 레이아웃의 알고리즘중 1, 2단계에서 적절한 크기를 찾는데에 영향을 준다
제약은 너비와 높이의 최소, 최대를 정한다

 제약조건의 유형

제한이 없음, 최대넓이 혹은 최소넓이가있거나 둘다 있다, 정확하게 크기를 지정함, 다른 조건들의 조합 (예: 최대넓이가 있으면서 높이를 지정함)

제약 조건이 상위 요소에서 하위 요소로 전달되는 방식

레이아웃 단계의 1단계처럼  루트노드는 하위노드의 크기를 측정하고 루트노드의 제약조건을 하위노드에 전달한다
하위요소가 제약조건을 변경하지 않는 modifier 이면 하위 modifeir 에 제약조건을 전달하고 제약조건을 변경하는 modifier면 제약조건을 변경하고 하위 modifier에 제약조간을 전달한다
리프노드에 도달하면 전달받은 제약조건에 따라서 크기를 결정하고 크기를 상위노드에 전달한다
상위요소는 하위 요소에게 전달받은 크기로 제약조건을 수정하고 수정된 제약조건을 또다른 하위요소에 전달한다

제약조건에 영향을 미치는 modifeir

size, requiredSize 
requiredSize는 제약조건을 무시하고 크기를 결정한다
width, height 
sizeIn
sizeIn은 높이나 넓이의 최대, 최소를 지정한다
