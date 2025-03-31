launch vs async

launch와 async 둘다 CoroutineScope 안에서 비동기적인 작업을 실행할때 사용하지만 중요한 차이점이 있다

|  | `launch` | `async` |
|---|---|---|
| **반환값** | `Job` (결과 없음) | `Deferred<T>` (결과 반환) |
| **사용 목적** | 단순한 비동기 실행 | 결과를 받아야 하는 비동기 실행 |
| **결과 받기** | `join()` 가능하지만 보통 필요 없음 | `await()` 호출해야 결과 받음 |
| **예외 처리** | 부모 코루틴에 전파됨 | `await()` 호출 시 예외 발생 |

결과 필요 없음 → `launch`  
결과 필요함 → `async` + `await()`

```kotlin
fun CoroutineScope.launch(
    context: CoroutineContext = EmptyCoroutineContext,
    start: CoroutineStart = CoroutineStart.DEFAULT,
    block: suspend CoroutineScope.() -> Unit
): Job {
    val newContext = newCoroutineContext(context)
    val coroutine = if (start.isLazy)
        LazyStandaloneCoroutine(newContext, block) else
        StandaloneCoroutine(newContext, active = true)
    coroutine.start(start, coroutine, block)
    return coroutine
}
```

```kotlin
fun <T> CoroutineScope.async(
    context: CoroutineContext = EmptyCoroutineContext,
    start: CoroutineStart = CoroutineStart.DEFAULT,
    block: suspend CoroutineScope.() -> T
): Deferred<T> {
    val newContext = newCoroutineContext(context)
    val coroutine = if (start.isLazy)
        LazyDeferredCoroutine(newContext, block) else
        DeferredCoroutine<T>(newContext, active = true)
    coroutine.start(start, coroutine, block)
    return coroutine
}
```