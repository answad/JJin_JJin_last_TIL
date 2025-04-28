Sequence는 List와 비슷하다
List와 Sequence 모두 요소에 하나씩 접근하는 객체이고, Sequence가 내부적으로 List가 사용하는 Iterator를 사용하기 때문이다

둘의 차이점은 연산 방식에 있다

List는 연산(filter, map 등)을 할때마다 새로운 리스트를 반환하고 다음 연산을 하는 방식이지만

Sequence는 연산들을 연결만 해두고 최종 연산(터미널 연산)(toList, forEach 등)이 호출되기 전까지 실제로 아무것도 수행하지 않으며
최종 연산이 호출될 때 요소를 하나씩 꺼내면서 지연(lazy)적으로 filter, map 등을 순차적으로 적용하여 필요한 만큼만 처리하기 때문에
전체를 모두 순회하는 List보다 불필요한 연산 수가 줄고 메모리 사용량 또한 줄어든다

예제


List
```kotlin
val list = listOf(1, 2, 3, 4, 5, 6)

val result = list
    .filter {
        println("filter: $it")
        it % 2 == 0
    }
    .map {
        println("map: $it")
        it * 10
    }
    .take(2)

println(result)
```

```less
filter: 1
filter: 2
filter: 3
filter: 4
filter: 5
filter: 6
map: 2
map: 4
map: 6
결과[20, 40]
```
리스트는 모든 요소에 대해서 filter를 하고 filter 연산이 반환한 list에 대해서
map연산으로 새로운 리스트로 변환 하고 그 리스트에서 2개의 요소를 가져온다 

연산이 총 9번 있었다

Sequence

```kotlin
val sequence = listOf(1, 2, 3, 4, 5, 6)
    .asSequence()

val result = sequence
    .filter {
        println("filter: $it")
        it % 2 == 0
    }
    .map {
        println("map: $it")
        it * 10
    }
    .take(2)
    .toList()

println(result)
```

```less
filter: 1
filter: 2
map: 2
filter: 3
filter: 4
map: 4
결과[20, 40]
```

시퀀스는 filter를 통과하는 요소가 나오면 다음 연산이 map을 실행하고 그 요소의 변환된 값을 take함수로 저장한다
그리고 다시 filter로 돌아가서 요소를 불러온다 그렇게 take함수까지 값이 2개 전달되면 연산을 멈춘다

시퀀스는 연산을 6번만 했다
