Flow는 Kotlin에서 비동기 데이터 스트림을 처리하기 위해 제공되는 API이다.

Flow는 코루틴과 함께 작동하며, 데이터 스트림을 선언적으로 생성하고 처리할 수 있도록 도와준다.

RxJava와 같은 반응형 Programming 라이브러리와 비슷하지만,

Kotlin 코루틴의 강력한 기능을 활용하여 간단하고 직관적으로 사용할 수 있도록 설계되었습니다.

Kotlin의 Flow는 비동기 데이터를 처리하는 데 유용하며, 이를 통해 간결하고 효율적인 코드 작성이 가능하다.

이 글에서는 Flow의 주요 특징과 사용법을 살펴보고, 이를 통해 Kotlin에서 비동기 프로그래밍을 어떻게 다룰 수 있는지 알아보겠다.

Flow의 주요 특징
1. Cold Stream
Flow는 cold stream이다.



즉, Flow는 구독(subscribe, collect)되기 전까지 실제로 데이터를 방출하지 않는다.

데이터 요청이 있을 때만 데이터를 생성하며, 리소스를 효율적으로 사용한다.

이 특성 덕분에 Flow는 데이터 스트림을 처리할 때 불필요한 리소스 낭비를 방지할 수 있다.

자세히 말하면, flow 빌더로 정의된 Flow는 collect가 호출되기 전까지 데이터 방출을 시작하지 않으므로, 데이터 소비자가 없을 경우에는 불필요한 작업을 수행하지 않습니다.

예시:

val numbers = flow {
    println("Flow started!")
    emit(1) // 데이터 방출
    emit(2) // 데이터 방출
}
numbers를 선언만 하면데이터가 방출되지 않고, collect가 호출될 때까지 기다린다.

2. Suspension 지원
Flow는 suspend 함수처럼 비동기적으로 데이터를 처리할 수 있다.



Flow의 코드도 suspend 함수처럼 작동하며, 이를 통해 코루틴 내에서 중단 가능한 함수처럼 데이터를 처리할 수 있다. emit() 함수가 suspend함수이고 데이터를 방출하는 동안 코루틴은 일시 중단(suspend) 상태에 들어가고, 비동기적으로 실행된다. 이 덕분에 UI 스레드를 차단하지 않고 데이터를 처리할 수 있어, 앱의 성능을 최적화하는 데 유리하다.

예시:

suspend fun fetchData(): String {
    delay(1000) // 비동기적으로 1초 대기
    return "Data"
}

val flowData: Flow<String> = flow {
    emit(fetchData()) // suspend 함수 호출
}
3. Backpressure 처리
Flow는 Backpressure를 처리할 수 있습니다. Backpressure란 데이터 생산자가 데이터를 너무 빠르게 생성할 때, 이를 처리하는 소비자가 데이터를 처리할 수 없는 상황을 말합니다. Flow는 데이터가 소비될 때까지 방출을 대기하거나, 소비자가 데이터를 처리할 수 있을 때만 다음 데이터를 방출합니다. 이를 통해 과도한 데이터 처리 문제를 자동으로 해결할 수 있습니다. 이 특성은 데이터 스트림을 안정적으로 처리할 수 있게 해 줍니다.



4. 다양한 연산자
Flow는 데이터 변환(map, filter), 병합(merge, zip), 수집(collect) 등 다양한 연산자를 제공하여, 데이터를 선언적이고 직관적으로 처리할 수 있게 해 준다. 이 연산자들은 데이터 스트림을 변형하고 처리하는 데 매우 유용하며, 코드가 간결하고 가독성이 좋게 만들어 줍니다. 예를 들어, Flow에서는 map, filter, reduce 등의 연산자를 사용하여 데이터를 직관적으로 처리할 수 있다.

예시:

val numberFlow = flowOf(1, 2, 3, 4, 5)
val transformedFlow = numberFlow
    .filter { it % 2 == 0 } // 짝수만 필터링
    .map { it * it } // 값을 제곱
이 코드는 짝수만 필터링하고, 그 값들을 제곱하여 새로운 Flow를 반환합니다. Flow의 연산자들을 사용하면 데이터 변환이나 처리 과정을 쉽게 구현할 수 있습니다.

Flow의 기본 사용법
1. Flow 생성
Flow는 flow 빌더를 사용하여 생성할 수 있습니다. 데이터는 emit() 함수를 통해 방출됩니다.

import kotlinx.coroutines.flow.Flow
import kotlinx.coroutines.flow.flow

fun getNumbers(): Flow<Int> = flow {
    for (i in 1..5) {
        emit(i) // 데이터 방출
        kotlinx.coroutines.delay(1000) // 1초 대기
    }
}
2. Flow 수집
Flow는 collect 연산자를 사용하여 데이터를 처리합니다. collect는 Flow를 구독하며, 데이터가 방출될 때마다 이를 처리할 수 있게 해 줍니다.

import kotlinx.coroutines.flow.collect
import kotlinx.coroutines.runBlocking

fun main() = runBlocking {
    getNumbers().collect { number ->
        println("Received: $number")
    }
}
3. Flow 연산자 사용
Flow는 여러 연산자를 사용하여 데이터를 변환하거나 필터링할 수 있습니다. 예를 들어, map, filter 등의 연산자를 사용하여 데이터의 흐름을 제어할 수 있습니다.

import kotlinx.coroutines.flow.map
import kotlinx.coroutines.flow.filter

fun getFilteredNumbers(): Flow<Int> = getNumbers()
    .filter { it % 2 == 0 } // 짝수만 필터링
    .map { it * 2 } // 짝수 값을 두 배로 증가시킴
Flow 활용 시 주의할 점
Flow는 비차단적 방식으로 동작하지만, collect가 호출되기 전까지는 실제로 데이터가 방출되지 않으므로, collect 호출을 적절히 제어해야 합니다.
Flow를 사용할 때 코루틴 컨텍스트에 주의해야 합니다. 예를 들어, UI 스레드에서 Flow를 직접 처리하는 대신, 백그라운드 스레드에서 처리하도록 하여 UI의 성능을 유지할 수 있습니다.
