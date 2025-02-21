takeIf와 takeUnless는
객체 상태를 검사하고 그 결과를 기반으로 객체를 필터링할 수 있게 해주는 함수이다
이 함수는 null safe 호출과 함께 사용될 때 유용하다

### **1. takeIf**
- **목적**: 주어진 **조건**을 만족하는 경우 객체를 그대로 반환하고, 그렇지 않으면 **null**을 반환.
- **특징**: 조건을 만족할 때만 객체를 반환하는 필터링 함수.
- **사용법**: 객체에 대한 조건을 `takeIf`의 **predicate**로 넘겨주며, 조건을 만족하면 그 객체가 반환되고, 그렇지 않으면 **null**이 반환됩니다.

### **2. takeUnless**
- **목적**: 주어진 **조건**을 **만족하지 않으면** 객체를 반환하고, 만족하면 **null**을 반환.
- **특징**: `takeIf`와 반대의 논리로 동작합니다. 조건을 만족하지 않을 때 객체를 반환합니다.

### **예시 1: takeIf와 takeUnless 사용**
```kotlin
val number = Random.nextInt(100)

val evenOrNull = number.takeIf { it % 2 == 0 }  // 짝수일 경우 number 반환, 아니면 null
val oddOrNull = number.takeUnless { it % 2 == 0 }  // 홀수일 경우 number 반환, 아니면 null

println("even: $evenOrNull, odd: $oddOrNull")
```
