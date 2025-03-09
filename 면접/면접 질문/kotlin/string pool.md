String Pool은 Java와 Kotlin(jvm 기반 언어)에서 문자열(String) 객체를 효율적으로 관리하는 메모리 관리 기법이다
간단히 말해, 동일한 값을 가진 문자열을 하나의 객체로 재사용해서 메모리 낭비를 줄이는 역할을 한다
또한 하나의 객체로 사용해서 같은 문자열이면 같은 주소를 가리켜서 문자열 비교에서도 이점이 있다

string pool이 적용되는 경우는 
컴파일타임에 확정된 문자열 리터럴에만 적용된다

예시로는 경우는 컴파일 타임에 결합한 문자열, ""으로 생성한 문자열에 적용된다

```kotlin
const val str2 = "He" + "llo"

fun main() {
    val str1 = "Hello"
    val str3 = "Hello"

    println(str1 === str2)  // true (컴파일타임 결합으로 같은 객체를 가리킨다)
    println(str1 === str3)  // true 
}
```

그러나 StringBuilder, String()같은 새로운 객체를 생성하는 경우, 런타임에 string을 결합하거나, 통신으로 받아오는 경우에는 적용이 안된다

```kotlin
fun main() {
    val str1 = "Hello"

    println(str1 === String("Hello".toCharArray()))  // false (다른 객체를 참조)
    println(str1 === "He" + "llo")  // false (런타임 결합으로 다른 객체가 생성됨)
    println(str1 === StringBuilder("Hello").toString())  // false (StringBuilder로 생성된 문자열은 String Pool에 없음)
}
```
