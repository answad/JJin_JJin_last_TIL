jetpack compose가 ui를 그리는 과정은 컴포지션, 레이아웃(측정, 배치), 그리기 주요한 3가지 단계가 있다 
컴포지션 단계에서는 composable 함수를 실행하고 composeable 함수가 의미하는 ui 애 대한 자세한 설명을 만듭니다

![image](https://github.com/user-attachments/assets/131857c3-62f5-4579-989a-a9978cc9ded2)

composable 함수 -> ui tree
레이아웃 단계에서는 ui tree 의 각 노드가 실제 핸드폰에서 어떤 크기를 가지고 어떤 위치를 차지할지 결정한다

![image](https://github.com/user-attachments/assets/97b9bde8-800b-47ec-ad5b-74a87b8efba3)

레이아웃 단계는 측정과 배치 2단계로 나눠지는데 
컴포지션 단계에서 생성된 ui tree 를 처리할때 
하위 요소 측정: 노드의 하위요소가 있는경우에 노드가 하위 요소를 측정합니다.
자체 크기 결정: 이러한 하위 요소의 측정치를 기반으로 노드가 자체 크기를 결정합니다.
하위 요소 배치: 각 하위 노드는 노드의 자체 위치를 기준으로 배치됩니다.
이러한 측정과 배치 2단계가 끝나면 ui tree의 각 노드에는 width와 height, x,y 좌표값이 있다
그리기 단계에서는 트리를 위에서 아래로 탐색하고 각 노드가 순서대로 화면에 그려진다
