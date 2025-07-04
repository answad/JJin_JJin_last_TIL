Mutex란 공유 자원에 한 번에 하나의 스레드 또는 코루틴만 접근하도록 도와주는 클래스이다
Mutex의 어원은 **Mutual Exclusion (상호 배제)**로, "서로를 배제한다"는 의미를 가진다

여러 스레드나 코루틴이 동시에 공유 자원에 접근하면 레이스 컨디션이 발생하여 데이터 꼬임, 잘못된 결과가 나타날 수 있다
이러한 문제를 방지하기 위해 Mutex를 사용한다

Mutex는 락을 관리하는 역할을 하며, mutex.withLock { } 블록 안에 있는 코드를 실행하기 위해서는 반드시 Mutex로부터 락을 획득해야 한다
만약 다른 스레드나 코루틴이 이미 락을 가지고 있다면, 현재 코루틴은 락이 해제될 때까지 suspend 상태로 대기하게 된다

withLock { }은 락의 획득과 해제를 자동으로 관리해주기 때문에, 별도로 lock() 및 unlock()을 호출할 필요가 없으며, 에러가 발생하더라도 락이 자동으로 해제되어 안정성이 높다

또한, 락을 무조건 기다리는 것이 아니라, tryLock()을 사용하면 락을 즉시 시도해볼 수 있으며, 실패 시 다른 작업을 선택적으로 처리할 수 있다. 이를 통해 불필요한 대기를 줄이고 빠른 분기 처리가 가능하다


Mutex는 과도하게 사용하면 성능 저하가 발생할 수 있으므로, 꼭 필요한 최소 범위에서 사용해야 한다

잘못 사용하면 데드락(락을 서로 기다리면서 프로그램이 멈추는 현상)이 발생할 수 있으니 락의 순서를 일관되게 관리해야 한다

Mutex는 코틀린 코루틴 전용으로 설계된 비동기 락이기 때문에, 일반 synchronized 블록과 달리 코루틴을 **블로킹하지** 않는다
