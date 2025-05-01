hiltViewModel()은 Jetpack Compose에서 Hilt 기반의 ViewModel 의존성 주입, 관리를 간편하게 사용할 수 있도록 제공되는 헬퍼 함수이다

이 함수는 내부적으로 ViewModelProvider를 사용하여 현재 Composable이 속한 ViewModelStoreOwner로부터 ViewModel을 가져오는데 
Compose + Jetpack Navigation 환경에서는 이 ViewModelStoreOwner가 NavBackStackEntry에 해당된다
이말은 hiltViewModel() 가 호출된 화면이 NabStack에서 삭제(pop)되면 ViewModel의 인스턴스도 삭제된다
이렇게 함으로써 화면 전환 시 ViewModel이 재사용되지 않고 각 화면마다 고유한 ViewModel 인스턴스를 유지할 수 있게 된다

그리고 Hilt에서는 @HiltViewModel과 @Inject constructor(...)를 함께 사용해서 ViewModel 인스턴스 생성 시 필요한 의존성을 자동으로 주입해줄 수 있게 한다

예시로 ViewModel에서 Repository나 UseCase 등을 생성자 파라미터로 요구할 경우, 기존 방식처럼 ViewModelFactory를 일일이 작성하고 ViewModelProvider에 전달할 필요 없이, Hilt가 자동으로 컴파일 타임에 생성자 기반으로 팩토리를 만들어준다
이 구조 덕분에 개발자는 의존성 주입에 대한 번거로운 설정 없이 깔끔하게 ViewModel을 구성할 수 있고, 테스트할 때도 같은 구조로 mocking된 의존성을 주입할 수 있다
또한 SavedStateHandle도 자동으로 받아올수있다
