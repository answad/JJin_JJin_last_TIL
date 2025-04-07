Java 에서는 

타입이 원시(Primitive) 타입과 참조 타입(reference)
둘로 나눠져있다

예시 원시 타입은 int, boolean, float 등이 있고
참조 타입은 Integer, Boolean, Float 등이 있다

int는 실제 4바이트의 숫자이고 Integer는 실제 4바이트 숫자를 감싸주는 객체이다

Primitive는 래퍼 클래스를 사용하는것보다 빠르고 메모리를 적게 쓰지만
Wrapper는 느리고 메모리를 더 사용하지만 toString, equals, MIN_VALUE, MAX_VALUE 같은 메서드, 속성이 더 있고 null 값을 받을수 있다

그리고 객체타입뿐만 null값을 받을수 있기때문에 java 개발자들은 객체타입인지 원시 타입인지, 객체타입을 원시타입으로 언박싱할때 null값을 가졌는지 아닌지를 신경써야하는등
불편한점이 많다

그래서 이러한 동작(함수, 속성), null 가능한지 같은지가 달라서 개발자가 신경써야할게 많아서 더 신경쓰지 않게 하기 위해서 원시타입을 개발자가 신경쓰지 않도록 개발자 수준에서 원시 타입을 사용하지 못하도록 하였다

그래서 Kotlin에는 원시 타입이 없는것처럼 보인다

int, boolean, float 가 아닌 Int, Boolean, Float 같은 클래스만이 있다
nullable한 타입은 Int?, Boolean?, Float? 처럼 "?" 를 사용해서 nullable을 표현한다

그러나 실제로 kotlin 을 컴파일 하면 jvm에서 Int는 원시타입 int로 Int? 는 참조타입 Integer로 바뀐다