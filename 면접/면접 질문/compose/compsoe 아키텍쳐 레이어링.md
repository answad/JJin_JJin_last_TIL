Jetpack Compose는 단순한 하나의 프로젝트가 아니다 
하나의 기능을 가진 스택을 만들기 위한 4개의 모듈로 만들어졌다 
이 모듈의 역할과 동작을 이해하면 
적당한 단계의 모듈을 사용하여 앱 빌딩 
세부적인 제어와 커스텀을 위해 낮은 수준으로 드롭다운 할 수 있는 경우 이해를 할 수 있다

Jetpack Compose 를 이루는 레이어는 
![image](https://developer.android.com/static/develop/ui/compose/images/layering-major-layers.svg?hl=ko)
Materal -> Foundation -> UI -> Runtime 이다

각 레이어는 하위 레이어들에 기반하고
높은 수준의 구성요소를 만들기 위해 기능을 결합해서 사용한다
각 레이어는 하위 레이어의 API를 기반으로 API를 구현 하여
특정레이어의 기능을 하위 레이어의 기능들로 구현할수 있다


런타임 모듈에는 
![image](https://github.com/user-attachments/assets/1c2d419d-69c3-4ff0-a9cd-2524cd8c0741)
상태 API : remember, mutableStateOf, collectAsState.
사이드 이펙트 API : LaunchedEffect, SideEffect.
코루틴 관련 API : rememberCoroutineScope, snapshotFlow.
CompositionLocal API : compositionLocalOf.
컴포지션관련 API : Composition, Recomposer, ComposeNode, and RecomposeScope.
시간 관련 API : MonotonicFrameClock, withFrameMillis,
Composable, Stable 같은 주석

이 외에도 많은 APi 들이 있다

컴포즈의 가장 기초를 재공하고 Compose의 ui tree 생성기능을 사용하려면 이 모듈을 사용한다


UI 모듈에서는 

ui-text, ui-graphics, ui-tooling같은 모듈로 구성되고 
실제로 UI 를 그리는 API 들이 있다
UI의 기본을 그리는 APi 만 필요하면 이 모듈을 사용한다

Foundation(기초)모듈에서는

UI를 더 쉽게 빌드할수 있도록 
Row, Column, LazyColumn같은 API 를 재공한다

Material 모듈은 

기초 모듈보다 복잡한 
	Switch, BottomSheet, Textfield	
같은 더욱 고도화된 API를 재공한다

--------------------------

JetpackCompose의 디자인 원칙

Compose의 디자인원칙중 하나는 모눌리스식 으로 Compose를 만드는것보다 
여러 레이어를 함깨 조립할수있게 각각의 레이어에 목적에 집중된 기능을 제공하는것이였다

이런식으로 기능을 구현하면 다양한 장점이 있다

1. 컨트롤 가능성
2. 커스텀 가능성

상위의 API는 내부적으로 더 많은 기능을 실행한다 그러나 개발자가 컨트롤할수 있는정도를 제한한다
개발자가 컨트롤할수있는 정도를 늘리려면 하위수준의 API를 할수 있다 

예시로 Button 이 있다 Button 은 Materal의 구성 요소이다 만약 개발자가 클릭시 효과를 커스텀하고싶으면 
그 하위의  모듈인 Foundation모듈의 Row와 런타임 모듈의 modifier를 이용해서 클릭 효과를 커스텀할수있다

개발자는 적당한 추상화정도를 택해서 Compose를 사용해야한다

3. 정확한 추상화 선택

그러나 이런 예시가 꼭 하위 요소를 사용하는게 좋다고 말하는것은 아니다 
예시로 Modifier.pointerInput 은 Modifier.draggable, Modifier.scrollable, Modifier.swipeable 에 자동으로 적용되어있다
그래서 적당한 추상화 정도를 선택해야한다
