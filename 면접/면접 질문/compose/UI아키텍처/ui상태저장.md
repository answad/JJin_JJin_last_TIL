compose 에서 개발자가 의도하지않게 ui 상태가 사라지는 경우는 구성변경(Configuration changes) 이 일어나거나 사용자가 앱을 완전히 Distory 하거나 앱이 백그라운드에서 onStop 일때 안드로이드 시스템(os) 가 리소스를 다른프로세스에서 쓸수 있게 앱을 종료했을때가 있다
사용자가 입력중에 이런 이벤트가 발생하면 ux 를 해칠수도 있다 그래서 이런 이벤트가 일어났을때 ui상태를 보존하는것은 ux를 긍정적이게 만들수 있다
이런경우에 상태를 보존하는것이 가능하도록 재공하는 여러 api 가 있다

1. rememberSaveable 

rememberSaveable 상태를 Bundle 객체에 담아서 저장한다
rememberSaveable은 Bundle객체를 사용함으로 여러 activity 에서 공유할수도 있다 
화면 회전 등의 구성 변경 후에도 상태를 복원할 수 있게 한다
기본적인 데이터 type은 저장할수 있지만 복잡한 상태는 복잡한 상태는 커스텀 Saver(예: `listSaver`, `mapSaver` 등)를 구현하여 저장할 수 잇다
Bundle은 크기제한이 있어서 큰 객체를 저장하는것은 추천하지 않는다 
더 복잡한 상태는 다른 영구저장소를 사용하는것을 추천함

SavedStateHandle은 viewModel 에서 쓰이는데 

viewModel은 SavedStateHandle를 사용한다
SavedStateHandle은 ViewModel에서 UI 상태를 저장하고 복원하는 데 사용됩니다.  
ViewModel은 구성 변경 시 상태를 자동으로 유지하지만, 시스템에 의한 프로세스 종료 시에는 상태를 유지하지 못한다
SavedStateHandle을 사용하면 Bundle 기반으로 UI 상태를 저장할 수 있어, 프로세스 종료 후에도 상태를 복원할 수 있다
SavedStateHandle또한 bundle 을 사용함으로 크거나 복잡한 객체를 저장하는것은 추천하지 않는다

SavedStateHandle은 사용하는 api 가 2가지가 있는데
하나는 Compose State에서 사용하는 MutableState로 읽을 수 있는 saveable()이고, 다른 하나는 StateFlow를 이용한 getStateFlow()이다
saveable()은 UI 상태를 MutableState로 읽고 쓸 수 있게 해준다
반면에 getStateFlow()는 SavedStateHandle에 저장된 상태를 읽기 전용 StateFlow로 제공한다
