IntArray, List<Int>의 차이

IntArray 

크기만 고정된 배열 
실제 int 형태로 값을 저장함
연속된 메모리 공간을 사용해서 값을 찾고, 수정하는것이 빠르다
초기화시에 0으로 자동 초기화된다

List<Int>

크기와 값 변경이 불가능한 읽기 전용 불변리스트
값을 저장할떄 Intiger로 저장되어서 박싱이 생긴다
int값을 Intiger객체로 감싸서 저장한다
그래서 메모리를 더 많이 쓰고 접근속도도 느리다

Intiger객체로 감싸는 이유는 
Kotlin에서 제네릭(List<T>)은 JVM 제네릭을 그대로 사용하고
JVM 제네릭은 primitive type(int 등)을 직접 다룰 수 없기 때문에, Int는 Integer로 박싱된다

그래서 박싱하고 언박싱하는데에 오버해드가 발생한다 
