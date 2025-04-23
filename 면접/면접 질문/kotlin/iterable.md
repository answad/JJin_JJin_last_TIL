우선 Iterable은 여러 개의 아이템을 순차적으로 하나씩 꺼내서 처리할수있는 "객체"를 말한다.
현재 읽고있는 위치를 저정하고 다음페이지로 넘어가면서 내용을 확인할 수 있게 해준다.

```kotlin
interface Iterator<out T> {
    /**
     * 다음요소가 있는지 Boolean 값을 반환하는 함수
     */
    operator fun hasNext(): Boolean

    /**
     * 다음 요소를 반환하는 함수
     */
    operator fun next(): T
}
```

이터레이터는 한번의 하나의 요소에 접근하고
리스트같은 자료형을 순회할때는 이터레이터를 통해 순회한다

 이터레이터 구현 예시
```kotlin
class MyNumberIterator(private val count: Int) : Iterator<Int> {
    private var current = 0

    override fun hasNext(): Boolean = current < count

    override fun next(): Int {
        if (!hasNext()) {
            throw NoSuchElementException()
        }
        return current++
    }
}

class MyNumbers(private val size: Int) : Iterable<Int> {
    override fun iterator(): Iterator<Int> = MyNumberIterator(size)
}

fun main() {
    val myNumbers = MyNumbers(5)
    for (number in myNumbers) {
        println("MyNumber: $number") // 출력: MyNumber: 0, 1, 2, 3, 4
    }
}
```

이터레이터를 사용하는 자료형은 Iterable 인터페이스를 구현하고
Iterable안의 iterator 함수는 Iterator를 구현한(이터레이터) 클래스를 반환한다.

간단하게 요약하자면 이터레이터, 이터레이터블은 특정 자료형을 순회할수있도록 하는 인터페이스이다
