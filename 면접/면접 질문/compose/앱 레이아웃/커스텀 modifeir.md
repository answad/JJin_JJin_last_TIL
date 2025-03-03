수정자는 여러 부분으로 구성된다

수정자 팩토리

Modifier의 **확장** 함수로, 수정자에 관용적인 API를 제공하고 수정자를 쉽게 체이닝할 수 있도록 한다
수정자 팩토리는 Compose에서 UI를 수정하는 데 사용하는 수정자 요소를 내부적으로 생성하고 사용한다

수정자 요소(Modifier.Element)

여기에서 실제로 수정자의 동작을 구현할 수 있다


필요한 기능에 따라 맞춤 수정자를 구현하는 방법에는 여러 가지가 있다
맞춤 수정자를 구현하는 가장 쉬운 방법은 이미 정의된 다른 수정자 팩토리를 결합하는 맞춤 수정자 팩토리를 구현하는 것이다 modifeir 체이닝
더 많은 커스텀이 필요하면 더 많은 유연성을 제공하는 Modifier.Node API를 사용하여 수정자 요소를 구현한다

맞춤 Modifeir 를 만드는가장 쉬운 방법은 
기본재공하는 수정자 팩토리를 체이닝 하는것이다

```kotlin
fun Modifier.myBackground(color: Color) = padding(16.dp)
    .clip(RoundedCornerShape(8.dp))
    .background(color)
```
이런식으로 clip과 background, padding을 같이 사용하는 커스텀 Modifeir를 만들수 있다 

Modifer 확장함수(수정자 팩토리)를 구현하는것 

```kotlin
@Composable
fun Modifier.fade(enable: Boolean): Modifier {
    val alpha by animateFloatAsState(if (enable) 0.5f else 1.0f)
    return this then Modifier.graphicsLayer { this.alpha = alpha }
}
```

주의할게 있다

Compose는 return 값이 있는 Composable은 skip 하지 않고 리컴포징한다

Modifier.Node를 이용한 커스텀 Modifeir

Modifier.Node는 Compose에서 Modfier를 만드는 가장 하위 수준의 APi이다 Compose 에서 Modifeir를 만들때 Modifier.Node를 이용한다
그리고 성능이 가장 좋은 Modifeir 생성 방법이다

Modifier.Node로 커스텀 modfier를 구현하는 작업은 2단계로 나뉜다

Modifier의 **로직**과 **상태**를 보유하는 Modifier.Node 구현
Modifier.Node 인스턴스를 만들고 업데이트하는 ModifierNodeElement구현


ModifierNodeElement 클래스는 스테이트리스(Stateless)이며 각 리컴포지션마다 새 인스턴스가 할당되고
Modifier.Node 클래스는 스테이트풀(Stateful)일 수 있으며 여러 리컴포지션에서 유지되며 재사용할 수도 있다
ModifierNode는 아주 다양한 종류가 있다

1 Modifier.Node 구현 (이 예에서는 CircleNode)은 맞춤 수정자의 기능을 구현함
```kotlin
// Modifier.Node
private class CircleNode(var color: Color) : DrawModifierNode, Modifier.Node() {
    override fun ContentDrawScope.draw() {
        drawCircle(color)
    }
}
```

2. ModifierNodeElement, 맞춤 수정자를 만들거나 업데이트하기 위한 데이터를 보유하는 변경 불가능한 클래스 구현

```kotlin
// ModifierNodeElement
private data class CircleElement(val color: Color) : ModifierNodeElement<CircleNode>() {
    override fun create() = CircleNode(color)

    override fun update(node: CircleNode) {
        node.color = color
    }
}
```

ModifierNodeElement는 create와 update 메서드를 구현해야한다

create는 Modifier.Node의 인스턴스를 생성하는 함수이고
Modifer가 첫 호출때 Modifer노드를 만들기 위해 호출된다

update는 수정자 노드가 존재하는동안에 속성이 변경되었을때 호출된다
성능 최적화를 위해서 파라미터로 이전에 생성한 Modifier.Node의 인스턴스를 받고 변경된 속성을 적용한다

그리고 ModifierNodeElement 는 equals 및 hashCode도 구현해야한다 update에서 update는 이전 파라미터와 달라졌을때만 호출한다

```kotlin
// Modifier factory
fun Modifier.circle(color: Color) = this then CircleElement(color)
```

실제로 CircleElement를 사용하는 수정자 팩토리


또한 Modifier.Node에서는 coroutineScope를 사용할수 있다

