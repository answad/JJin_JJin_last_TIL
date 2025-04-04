Coroutine Dispatcher란

Coroutine Dispatcher는 코루틴을 어떤 정책으로 어떤 스레드, 스레드 풀에서 실행시킬지 결정한다

스레드풀은 미리 일정 개수의 스레드를 만들어 놓고, 작업이 들어올 때마다 그 스레드를 재사용하는 것이다

Coroutine Dispatcher 종류 

Dispatchers.Main
UI를 수정하는 작업을 할때 사용하는 스레드
단 하나의 스레드만 존제한다
메인스레드에 접근하기위해서 Handler와 Looper를 사용한다
다른 쓰레드에서 UI를 수정하면 crash가 난다

Dispatchers.IO
파일 읽기, 통신같은 블로킹 작업을 할때 사용한다
기본적으로 64개의 스레드 OR CPU 코어 수 중 더 큰값을 사용하고 더 많은 스레드를 생성할수 있다
CoroutineScheduler 스레드 풀을 사용한다
IO는 CPU자원을 거의 안쓰고 대기시간이 긴 작업들이여서 이렇게 많은 스레드를 사용할 수 있다
그래서 IO에서 CPU를 많이 사용하는 작업을 하면 다른 IO 작업도 느려지고 심하면 전체 앱성능 저하한다

Dispatchers.Default
CPU 자원을 많이 사용하는 작업을 할떄 사용한다
Dispatchers를 따로 지정하지 않으면 사용된다
CoroutineScheduler라는 스래드 풀을 사용하고 스레드 수는 64, CPU의 코어수 둘중 작은수만큼 사용한다
non 블로킹 코드를 넣어야하는데 작업을 하고 기다리는 시간이(CPU가 놀고있는 시간이) 없어야하고
스레드를 차지하는동안 쭉 작업을 해야한다

Dispatchers.Unconfined
특정 스레드에 지정되는것이 아니라 처음에는 호출된 스레드에서 실행하다가 중단점 이후에는 재실행하는 스레드에서 이어서 실행된다 