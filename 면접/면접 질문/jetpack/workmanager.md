백그라운드작업을 위한 WorkManager 

WorkManager는 jetpack이 재공하는 효율적인 백그라운드 태스크 관리 라이브러리입니다

WorkManager는 작업에 제약조건을 걸어서 작업을 처리하기 좋은 특정 상황에 작동하게 할수 있다 예시로 와이파이가 연경됬을때, 충전중일떄가 있다
또한 WorkManager의 작업은 SqlLite에 저장돼어서 앱 종료시나 기기 재부팅시에도 예약된 작업이 유지된다
그리고 작업 실패시에 자동으로 재시도를 할수 있고
작업들을 순차적으로 실행하도록 작업 체이닝을 할수 있다

WorkManager의 요소

Worker, WorkRequest, Constraints, WorkManager 가 있다
Worker는 실제로 작업이 어떤 일을 할지 정의하고 
WorkRequest는 OneTimeWorkRequestBuilder, PeriodicWorkRequestBuilder을 사용해서 만들고 얼마나자주, 몇번 요청 보낼지를 정한다
Constraints 는 작업의 실핼조건을 정의한다
WorkManager은 작업을 예약하거나 실행하고, 취소 등을 관리한다


Worker (실제 작업을 수행하는 클래스)  
- doWork() 메서드를 오버라이드하여 작업을 수행한다  
- 작업이 성공, 실패, 재시도 여부를 `Result.success()`, `Result.failure()`, `Result.retry()`로 반환해야 한다

```kotlin
class SyncWorker(context: Context, workerParams: WorkerParameters) : Worker(context, workerParams) {
    override fun doWork(): Result {
        return try {
            syncData() // 백그라운드에서 데이터 동기화 수행
            Result.success() // 작업 성공
        } catch (e: Exception) {
            Result.retry() // 실패 시 재시도 요청
        }
    }
}
```

WorkRequest (작업 요청)  
실제 작업을 실행하기 위해 요청을 만드는 객체
- OneTimeWorkRequest → 한 번 실행  
- PeriodicWorkRequest → 일정 시간마다 반복 실행  

```kotlin
val workRequest = OneTimeWorkRequestBuilder<SyncWorker>().build()
WorkManager.getInstance(context).enqueue(workRequest)
```

```kotlin
val periodicWorkRequest = PeriodicWorkRequestBuilder<SyncWorker>(15, TimeUnit.MINUTES).build()
WorkManager.getInstance(context).enqueue(periodicWorkRequest)
```

---

Constraints (작업 실행 조건)  
작업을 실행할 조건(네트워크 연결, 충전 중 여부 등)을 설정할 수 있다

```kotlin
val constraints = Constraints.Builder()
    .setRequiredNetworkType(NetworkType.CONNECTED) // Wi-Fi 또는 모바일 네트워크 필요
    .setRequiresCharging(true) // 충전 중일 때만 실행
    .build()

val workRequest = OneTimeWorkRequestBuilder<SyncWorker>()
    .setConstraints(constraints)
    .build()
```

---

WorkManager (작업을 관리하는 API)  
- enqueue(workRequest): 작업을 예약  
- cancelWorkById(id): 특정 작업 취소  
- cancelAllWork(): 모든 작업 취소  

```kotlin
val workManager = WorkManager.getInstance(context)

// 작업 예약
workManager.enqueue(workRequest)

// 작업 취소
workManager.cancelWorkById(workRequest.id)
```
