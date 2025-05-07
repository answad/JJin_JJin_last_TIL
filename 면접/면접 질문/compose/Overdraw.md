Overdraw 란

Overdraw는 UI를 렌더링할 때 한 프레임 내에서 같은 픽셀을 두 번 이상 그리는 현상
이렇게 같은 픽셀을 여러번 그리면 사용자가 볼수 없는 픽셀을 랜더링 함으로서 GPU 자원을 낭비한다

예시로는 

```kotlin
Box(modifier = Modifier.fillMaxSize().background(Color.White)) {
    Box(
        modifier = Modifier
            .fillMaxSize()
            .background(Color.Gray)
    )
    Text("Hello", modifier = Modifier.align(Alignment.Center))
}
```
이런 코드가 있다
이 코드는 배경을 2번이상 지정해서 오버드로우가 발생한다

개발자 입장에서는 가려지는 부분이니까 상관 없지 않을까 생각할수도 있지만 시스템은 Painter's Algorithm을 사용한다
이 알고리즘은 뒤에 있는(z축 값이 낮은) 요소부터 그리고 그 앞의(z축 값이 높은) 요소들이 뒤에 있는(z축 값이 낮은) 요소를 가리는식으로 UI를 그린다

안드로이드 시스템의 개발자 옵션에서는 각 픽셀을 그리는 횟수를 색으로 표시한다

개발자 옵션
![image](https://github.com/user-attachments/assets/3154d5cc-a176-46bc-a131-825f76fb799a)

오버드로우 시각화
![image](https://github.com/user-attachments/assets/853f4229-528c-48e4-93db-f4b9e50341a6)

트루 컬러: 오버드로 없음
파란색: 오버드로 1회
녹색: 오버드로 2회
분홍색: 오버드로 3회
빨간색: 오버드로 4회 이상

이런 오버드로우를 일으키는 원인은 불필요한 배경, 투명도가 있다

레이아웃(column, row)에는 배경이 없다 
그래서 레이아웃 자체는 어떤것도 랜더링하지 않는다 그러나 배경이 있으면 불필요한 오버드로가 발생할수도 있다

투명도를 적용하면 이전 요소의 색과 새로운 요소의 색을 조합해서 다시 그려야하기떄문에 필수적인것이 아니면 투명도를 조절하지 않고 색을 변경하는것이 낫다
