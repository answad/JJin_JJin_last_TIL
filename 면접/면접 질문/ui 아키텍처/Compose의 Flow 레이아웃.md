Compose에서는 디자인 요구사항을 해결하기 위한 복잡한 UI를 Compose 사용자가 구현하지 않을수 있도록 많은 UI를 재공한다

Flow 레이아웃도 그중 하나이다
Flow레이아웃에는 FlowRow와 FlowColumn이 있다
각각 Row과 Column과 비슷한 UI 이지만 각각 넓이, 높이가 부족하면 아래도 흘러내리게 한다

https://developer.android.com/static/develop/ui/compose/images/layouts/flow/flow_row_simple.png?hl=ko
위 예시는 FlowRow이다 

Flow 레이아웃은 기본적으로 아이템들의 순서를 변경하지 않는다 
또한 FLow 레이아웃은 maxItemsInEachRow또는 maxItemsInEachColumn을 이용해서  
한 줄에 배치할 아이템의 숫자를 지정할수 있다  

ContextualFlowRow, ContextualFlowColumn일반적인 Flow 레이아웃과 비슷하지만 
지연로딩이 가능하고 maxline를 넘엇을때 실행할 동작을 넣을수 있는 overflow 파라미터도 있다
또한 ContextualFlow는 그리기 단계가 아니더라도 index, 줄 번호 사용가능한 크기정보를 제공한다

또 Row, Column 안에서의 .fillMaxWidth 와 FlowRow, FlowColumn 안에서의 .fillMaxWidth 의 작동방식이 다르다

만약 Flow레이아웃안의 아이템에서 fillMaxWidth(0.7f)를 사용하면
Flow 레이아웃에서의 fillMaxWidth는 **전체 레이아웃의 넓이**의 70%를 차지하고 
일반적인 Row, Column레이아웃안에서 사용하면 **남은**넓이의 70%를 차지한다

fillMaxColumnWidth(), fillMaxRowHeight()
이 수정자팩토리들은
Flow 레이아웃에서 아이템중에서 가장 넓은,가장 높은 아이템의 넓이, 높이만큼 차지하도록 하는 수정자 팩토리이다
