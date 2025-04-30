Android Architecture Components ViewModel

AAC ViewModel는 우선 UI의 데이터(상태)를 생명주기에서 조금 떨어져서 데이터를 보존할수 있게 도와준다
원래 구성 병경이 잃어나거나 액티비티가 재생성되면 데이터가 사라지는데 그러한 이벤트에서도 데이터가 살아있도록 해주는 클래스이다

ViewModel의 생성 흐름

1. `ViewModelProvider`로 ViewModel 요청

```kotlin
val vm = ViewModelProvider(this).get(MyViewModel::class.java)
```

여기서 `this`는 `ViewModelStoreOwner` (`Activity`, `Fragment`, `NavBackStackEntry` 등)
이 객체를 기반으로 ViewModel을 생성하거나 재사용함
  
2. 내부에서 ViewModelStore 가져오기

```kotlin
val store: ViewModelStore = viewModelStoreOwner.getViewModelStore()
```

ViewModelProvider는 내부에서 ViewModelStoreOwner로부터 ViewModelStore를 획득
ViewModelStore는 key → ViewModel 맵 구조의 저장소

3. ViewModel 재사용 or 생성

```kotlin
val existingViewModel = store.get(key)
```

store에 key로 이미 존재하면 기존 ViewModel 반환
존재하지 않으면 Factory.create()를 호출해 새 ViewModel 생성
생성된 ViewModel은 다시 store.put(key, viewModel)을 통해 저장됨

```kotlin
if (store.contains(key)) {
    return store.get(key)
} else {
    val viewModel = factory.create(MyViewModel::class.java)
    store.put(key, viewModel)
    return viewModel
}
```

```pgsql
[1] 클라이언트 코드
    └── val vm = ViewModelProvider(owner).get(MyViewModel::class.java)
                                          │
                                          ▼
[2] ViewModelProvider
    └── owner.getViewModelStore() ───────────────┐
                                          ▼      │
[3] ViewModelStore                               │
    └── has ViewModel for "MyViewModel"?         │
           ├── YES ──▶ return existing ViewModel│
           └── NO                                ▼
                        [4] ViewModelFactory
                            └── create(MyViewModel::class.java)
                                          ▼
                            new MyViewModel() ←─ 생성
                                          ▼
[5] ViewModelStore
    └── store.put(key, viewModel)  ← 저장
                                          ▼
[6] ViewModelProvider
    └── return newly created ViewModel

```
