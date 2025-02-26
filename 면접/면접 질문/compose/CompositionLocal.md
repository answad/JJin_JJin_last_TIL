개요

CompositionLocal은 Jetpack Compose에서 데이터를 암시적으로 전달할 수 있도록 도와주는 API 이다
일반적으로 Compose에서는 데이터를 UI 트리를 따라 매개변수로 전달하지만
색상이나 글꼴 스타일처럼 여러 곳에서 자주 사용하는 데이터의 경우
매번 명시적으로 전달하는 것이 번거로울 수 있다
이를 해결하기 위해 CompositionLocal을 사용하면 UI 트리를 따라 데이터를 암시적으로 전달할수 있다

CompositionLocal 특징  

CompositionLocal은 UI 트리의 특정 노드에서 값을 설정하고
그 아래에 있는 모든 컴포저블이 해당 값을 암시적으로 사용할 수 있도록 값을 재공한다
예를 들어 `MaterialTheme`은 `colorScheme`, `typography`, `shapes` 같은 데이터를 CompositionLocal을 통해 전달한다
CompositionLocal 인스턴스는 `MaterialTheme` 내부에서 정의된 `LocalColorScheme`, `LocalShapes`, `LocalTypography` 객체로 Provide하고
이를 통해 UI의 하위 요소에서도 명시적인 매개변수 전달 없이 해당 값에 접근할수 있다

```kotlin
@Composable
fun MaterialTheme(
    colorScheme: ColorScheme = MaterialTheme.colorScheme,
    shapes: Shapes = MaterialTheme.shapes,
    typography: Typography = MaterialTheme.typography,
    content: @Composable () -> Unit
) {
    val rippleIndication = rippleOrFallbackImplementation()
    val selectionColors = rememberTextSelectionColors(colorScheme)
    @Suppress("DEPRECATION_ERROR")
    CompositionLocalProvider(
        LocalColorScheme provides colorScheme,
        LocalIndication provides rippleIndication,
        // TODO: b/304985887 - remove after one stable release
        androidx.compose.material.ripple.LocalRippleTheme provides CompatRippleTheme,
        LocalShapes provides shapes,
        LocalTextSelectionColors provides selectionColors,
        LocalTypography provides typography,
    ) {
        ProvideTextStyle(value = typography.bodyLarge, content = content)
    }
}
```

CompositionLocalProvider

CompositionLocal 값을 제공하려면 `CompositionLocalProvider`를 사용해야 한다. `provides` 중위 함수를 이용해 CompositionLocal 인스턴스에 새로운 값을 설정할 수 있다
이 값은 CompositionLocal을 사용하는 하위 요소에서 자동으로 적용된다

예시로 `LocalContentColor`는 Compose의 Text 컴포저블함수의 기본적인 텍스트 색상을 정의하는 CompositionLocal이다
`CompositionLocalProvider`를 사용하여 특정 영역에서 `LocalContentColor`의 값을 변경할 수 있다

![image](https://github.com/user-attachments/assets/52579589-aa25-4049-a397-b1afaafc6e4e)

```kotlin
CompositionLocalProvider(LocalContentColor provides MaterialTheme.colorScheme.primary) {
    Text("Primary color provided by LocalContentColor")
}
```
CompositionLocalProvider가 LocalContentColor를 MaterialTheme.colorScheme.primary색으로 변경하고 
그 값을 범위내의 Text 컴포넌트에서 사용한다 

#### CompositionLocal 값 소비  
CompositionLocal에 설정된 값을 가져오려면 `current` 속성을 사용하면 된다. 예를 들어, `LocalContext` CompositionLocal을 이용해 현재 `Context` 객체를 가져올 수 있다

```kotlin  
@Composable
fun SomeExampl(fruitSize: Int) {
    val context = LocalContext.current
}
```  

이 코드에서는 `setContent` 에서 제공하는 `LocalContext.current`를 사용하여 `Context` 객체를 가져온다

커스텀 CompositionLocal 만들기  
기본 제공되는 CompositionLocal객체 외에도 사용자가 직접 CompositionLocal을 정의할 수도 있다.
`compositionLocalOf`나 `staticCompositionLocalOf`를 사용하여 새로운 CompositionLocal을 만들 수 있다
compositionLocalOf는 Compose가 상태`읽기`를 추적하지만
staticCompositionLocalOf는 Compose가 상태 읽기를 추적하지 않아서 staticCompositionLocalOf의 content 람다의 전체를 리컴포지션한다
val LocalElevations = compositionLocalOf { Elevations() }

여기서 `LocalElevations`는 기본값을 갖는 CompositionLocal이다. 앱의 테마에 따라 다르게 동작하도록 하려면 `CompositionLocalProvider`를 이용해 새로운 값을 제공할 수도 있다

```kotlin
CompositionLocalProvider(LocalElevations provides Elevations(card = 1.dp, default = 1.dp)) {
    // 해당 범위 내에서 LocalElevations.current를 통해 Elevations 값 사용 가능
}
```  

CompositionLocal은 필요하고 유용한 APi 이지만 무조건 코드의 가독성과 데이터의 흐름을 잘 파악할수 있는지 생각하고 사용해야한다
CompositionLocal을 무분별하게 사용하면 코드의 가독성이 떨어지고, 데이터의 흐름이 명확하지 않아 디버깅이 어려워진다

CompositionLocal을 사용하는 것이 적절한 경우
트리 전체 또는 하위 계층 구조에서 동일한 데이터를 공유해야 하는 경우  
데이터가 특정 UI 요소와 강하게 연결된 경우 (예: 테마 정보)  
UI 구성 요소 간 데이터 공유가 필요하지만, 매개변수로 전달하면 코드가 복잡해지는 경우  (예: context)

반대로 CompositionLocal이 적절하지 않은 경우는 다음과 같다.  
특정 화면의 `ViewModel`을 CompositionLocal을 통해 공유하려는 경우  
특정 UI 요소에서만 필요한 데이터를 CompositionLocal로 전달하려는 경우
적절하지 않은 경우는 너무 일반적이지 않은거같다

용어정리

CompositionLocal은 클래스이고, CompositionLocalProvider는 값을 공급하는 컴포저블이다
current는 CompositionLocal에서 값을 읽기 위한 CompositionLocal의 속성
compositionLocalOf이나 staticCompositionLocalOf은 프로바이드 가능한 CompositionLocal인스턴스를 생성하는함수이다
LocalColorScheme과 LocalShapes는 CompositionLocal의 인스턴스로, 각각 색상 및 모양 관련 값을 제공한다
