http 연결은

클라이언트(웹 브라우저)  ->  연결 요청  ->  서버
클라이언트(웹 브라우저)  <-  TCP 연결 설정  ->  서버
클라이언트(웹 브라우저)  ->  HTTP 요청 메시지  ->  서버
클라이언트(웹 브라우저)  <-  HTTP 응답 메시지  <-  서버
클라이언트(웹 브라우저)  ->  연결 종료  ->  서버

이런 흐름으로 이루어진다 
그런데 우리에게 중요한것은 HTTP 요청 메시지 HTTP 응답 메시지 인데 연결을 따로 하는것이 너무 아깝지 않나?
그래서 같은 서버(baseUrl)에 보내는요청에 대해서 tcp 연결을 끊지 않고 재사용하는 connaction pool 이 있다!!!

connaction pool은 tcp 연결을 끊지 않고 connaction pool 에 저장했다가 요청을 보낼때 이미 있는 tcp 연결로 http 요청을 보내는식이다
connaction pool은 검색을 해봐도 서버와 db 와의 커넥션 풍 말고는 자료가 잘 안나와서 

직접 okhttp 라이브러리를 찾아봤다 

okhttp 사용 예시는 다음과 같다

OkHttpClient 객체 생성, Request(요청)객체 생성, 두 객체를 이용해서 요청 보내기

```kotlin
    // 커넥션 풀 설정 (최대 5개 연결, 5분 후 연결 종료)
    val connectionPool = ConnectionPool(5, 5, TimeUnit.MINUTES)

    // OkHttpClient에 커넥션 풀 적용
    val client = OkHttpClient.Builder()
        .connectionPool(connectionPool)
        .build()

    val request = Request.Builder()
        .url("https://someapi/rich")
        .build()

    // 요청 실행하고 응답 받기
    client.newCall(request).execute().use { response ->
      
    }
```

우선 OkHttpClient 객체 생성, connaction pool 설정부터 찾아봤다

OkHttpClient class 는 우선 builder 패턴을 사용한다 
```kotlin
open class OkHttpClient internal constructor(
  builder: Builder
) : Cloneable, Call.Factory, WebSocket.Factory {
 class Builder constructor() {
    internal var connectionPool: ConnectionPool = ConnectionPool()
    internal val interceptors: MutableList<Interceptor> = mutableListOf()
    ....

    internal constructor(okHttpClient: OkHttpClient) : this() {
      this.connectionPool = okHttpClient.connectionPool
      this.interceptors += okHttpClient.interceptors
      .... 
    }
     fun addInterceptor(interceptor: Interceptor) = apply {
      interceptors += interceptor
    }
    fun connectionPool(connectionPool: ConnectionPool) = apply {
      this.connectionPool = connectionPool
    }
    ...
    fun build(): OkHttpClient = OkHttpClient(this)
  }
}
```

 그래서 connectionPool속성에 기본값을 재공하는데 
 ```kotlin
class ConnectionPool internal constructor(
  internal val delegate: RealConnectionPool
) {
  constructor(
    maxIdleConnections: Int,
    keepAliveDuration: Long,
    timeUnit: TimeUnit
  ) : this(RealConnectionPool(
      taskRunner = TaskRunner.INSTANCE,
      maxIdleConnections = maxIdleConnections,
      keepAliveDuration = keepAliveDuration,
      timeUnit = timeUnit
  ))

  constructor() : this(5, 5, TimeUnit.MINUTES)
}
```

3번째 생성자가 2번째 생성자를 사용하고 2번째 생성자가 1번째 생성자를 사용한는식으로 구현되어있고 
기본값은 위쪽 예제의 값과 동일한 (5, 5, TimeUnit.MINUTES) 으로 들어가 있다
그리고 이렇게 하면 하나의 OkHttpClient 당 하나의 ConnectionPool, RealConnectionPool을 가질수 있게 된다

그다음은 RealConnectionPool을 설명할건데 
RealConnectionPool은 실제로 connection들을 ConcurrentLinkedQueue형태로 저장하고 
요청시에 connaction 을 connectionPool에 저장할수 있도록 함수를 재공하고
설정된 시간이 지나면 connection을 connectionPool에서 삭제하고
connection을 제사ㅏ용할수 있도록 함수를 재공한다 
```kotlin

class RealConnectionPool(
  taskRunner: TaskRunner,
  private val maxIdleConnections: Int,
  keepAliveDuration: Long,
  timeUnit: TimeUnit
) {
  // connectionPool
  private val connections = ConcurrentLinkedQueue<RealConnection>()

  // connection pool 사용할수 있게 해주는 함수
  fun callAcquirePooledConnection(
    address: Address,
    call: RealCall,
    routes: List<Route>?,
    requireMultiplexed: Boolean
  ): Boolean {
    for (connection in connections) {
      synchronized(connection) {
        if (requireMultiplexed && !connection.isMultiplexed) return@synchronized
        if (!connection.isEligible(address, routes)) return@synchronized
        call.acquireConnectionNoEvents(connection)
        return true
      }
    }
    return false
  }

  // connection 추가함수
  fun put(connection: RealConnection) {
    connection.assertThreadHoldsLock()

    connections.add(connection)
    cleanupQueue.schedule(cleanupTask)
  }
  ....

  companion object {
    fun get(connectionPool: ConnectionPool): RealConnectionPool = connectionPool.delegate
  }
}
```

다음 차레는 요청객체 생성 부분이지만 connection pool과는 연관이 없어서 넘어갔다

이제 다음은 요청 보내기 인데 
```kotlin
client.newCall(request).execute().use {}
```

OkHttpClient의 newCall 함수를 사용해서 Call 객체를 만들고 
Call의 execute 함수를 사용해서 실제로 요청을 보내고
use함수로 요청의 결과를 사용하고 response 객체를 처리한다
이중에서 execute에서 connection pool 을 사용한다 

OkHttpClient
```kotlin
  override fun newCall(request: Request): Call = RealCall(this, request, forWebSocket = false)
```

RealCall.execute()
```kotlin
 override fun execute(): Response {
    check(executed.compareAndSet(false, true)) { "Already Executed" }

    timeout.enter()
    callStart()
    try {
      client.dispatcher.executed(this)
      return getResponseWithInterceptorChain()
    } finally {
      client.dispatcher.finished(this)
    }
  }
```
함수 내부에서 RealCall.getResponseWithInterceptorChain를 호출하는데 
```kotlin

  @Throws(IOException::class)
  internal fun getResponseWithInterceptorChain(): Response {
    // Build a full stack of interceptors.
    val interceptors = mutableListOf<Interceptor>()
    interceptors += client.interceptors
    interceptors += RetryAndFollowUpInterceptor(client)   // 중요
    interceptors += BridgeInterceptor(client.cookieJar)
    interceptors += CacheInterceptor(client.cache)
    interceptors += ConnectInterceptor
    if (!forWebSocket) {
      interceptors += client.networkInterceptors
    }
    interceptors += CallServerInterceptor(forWebSocket)

    val chain = RealInterceptorChain(
        call = this,
        interceptors = interceptors,
        index = 0,
        exchange = null,
        request = originalRequest,
        connectTimeoutMillis = client.connectTimeoutMillis,
        readTimeoutMillis = client.readTimeoutMillis,
        writeTimeoutMillis = client.writeTimeoutMillis
    )

    var calledNoMoreExchanges = false
    try {
      val response = chain.proceed(originalRequest)
      if (isCanceled()) {
        response.closeQuietly()
        throw IOException("Canceled")
      }
      return response
    } catch (e: IOException) {
      calledNoMoreExchanges = true
      throw noMoreExchanges(e) as Throwable
    } finally {
      if (!calledNoMoreExchanges) {
        noMoreExchanges(null)
      }
    }
  }
```

```kotlin
class RetryAndFollowUpInterceptor(private val client: OkHttpClient) : Interceptor {

  @Throws(IOException::class)
  override fun intercept(chain: Interceptor.Chain): Response {
    val realChain = chain as RealInterceptorChain
    var request = chain.request
    val call = realChain.call
    var followUpCount = 0
    var priorResponse: Response? = null
    var newExchangeFinder = true
    var recoveredFailures = listOf<IOException>()
    while (true) {
      call.enterNetworkInterceptorExchange(request, newExchangeFinder) //  중요

      var response: Response
      var closeActiveExchange = true
      try {
        if (call.isCanceled()) {
          throw IOException("Canceled")
        }

        ....
}

```

 RealCall 의
```kotlin
  fun enterNetworkInterceptorExchange(request: Request, newExchangeFinder: Boolean) {
    check(interceptorScopedExchange == null)

    synchronized(this) {
      check(!responseBodyOpen) {
        "cannot make a new request because the previous response is still open: " +
            "please call response.close()"
      }
      check(!requestBodyOpen)
    }

    if (newExchangeFinder) {
      this.exchangeFinder = ExchangeFinder( // 중요
          connectionPool,
          createAddress(request.url),
          this,
          eventListener
      )
    }
  }
```


```kotlin
class ExchangeFinder(
  private val connectionPool: RealConnectionPool,
  internal val address: Address,
  private val call: RealCall,
  private val eventListener: EventListener
) {
  private var routeSelection: RouteSelector.Selection? = null
  private var routeSelector: RouteSelector? = null
  private var refusedStreamCount = 0
  private var connectionShutdownCount = 0
  private var otherFailureCount = 0
  private var nextRouteToTry: Route? = null

  fun find(
    client: OkHttpClient,
    chain: RealInterceptorChain
  ): ExchangeCodec {
    try {
      val resultConnection = findHealthyConnection(
          connectTimeout = chain.connectTimeoutMillis,
          readTimeout = chain.readTimeoutMillis,
          writeTimeout = chain.writeTimeoutMillis,
          pingIntervalMillis = client.pingIntervalMillis,
          connectionRetryEnabled = client.retryOnConnectionFailure,
          doExtensiveHealthChecks = chain.request.method != "GET"
      )
      ...
    }
  }
  private fun findHealthyConnection(
    connectTimeout: Int,
    readTimeout: Int,
    writeTimeout: Int,
    pingIntervalMillis: Int,
    connectionRetryEnabled: Boolean,
    doExtensiveHealthChecks: Boolean
  ): RealConnection {
      val candidate = findConnection(
          connectTimeout = connectTimeout,
          readTimeout = readTimeout,
          writeTimeout = writeTimeout,
          pingIntervalMillis = pingIntervalMillis,
          connectionRetryEnabled = connectionRetryEnabled
      )
      .... 
    }
  }
@Throws(IOException::class)
  private fun findConnection(
    connectTimeout: Int,
    readTimeout: Int,
    writeTimeout: Int,
    pingIntervalMillis: Int,
    connectionRetryEnabled: Boolean
  ): RealConnection {
    ...
    // Attempt to get a connection from the pool.
    if (connectionPool.callAcquirePooledConnection(address, call, null, false)) { // 중요
      val result = call.connection!!
      eventListener.connectionAcquired(call, result)
      return result
    }
    ...
  }
}
```

```kotlin
      if (connectionPool.callAcquirePooledConnection(address, call, routes, false)) {
        val result = call.connection!!
        eventListener.connectionAcquired(call, result)
        return result
      }
```
이코드를 한줄한줄 봐보면 
RealConnectionPool.callAcquirePooledConnection

```kotlin
  fun callAcquirePooledConnection(
    address: Address,
    call: RealCall,
    routes: List<Route>?,
    requireMultiplexed: Boolean
  ): Boolean {
    for (connection in connections) {
      synchronized(connection) {
        if (requireMultiplexed && !connection.isMultiplexed) return@synchronized
        if (!connection.isEligible(address, routes)) return@synchronized
        call.acquireConnectionNoEvents(connection)
        return true
      }
    }
    return false
  }
```
이 함수는 connections을 순회하면서 조건에 맞는 connection이 있으면 RealCall.acquireConnectionNoEvents함수를 실행시키고  true를 반환하고 없으면 false 를 함환한다


RealCall.acquireConnectionNoEvents
```kotlin
  fun acquireConnectionNoEvents(connection: RealConnection) {
    connection.assertThreadHoldsLock()

    check(this.connection == null)
    this.connection = connection
    connection.calls.add(CallReference(this, callStackTrace))
  }
```

이 함수는
Realcall 객체의 connection 변수를 파라미터로 받은 connection으로 바꾸고 
CallReference(this, callStackTrace) 객체를 생성해서 현재 연결(connection)의 calls 리스트에 추가한다

그리고 바뀐 connection 을 
val result = call.connection!!
변숭에 저장하고 
```kotlin
   eventListener.connectionAcquired(call, result)
```
connectionAcquired 함수는 eventListener에 연결을 알리는 콜백이다

또 find함수 안에서 resultConnection.newCodec(client, chain)라는 코드가 있는데 RealConnection.newCodec 함수는 
적절한 http 설정이 안되어있으면 http 버전을 설정하고 소켓 타임아웃 설정후에 ExchangeCodec객체를 반환한다

이렇게ExchangeFinder.find 함수는 connection을 재홍룔한 ExchangeCodec을 반환한다 

그 후 RealCall.initExchange
함수가 find함수로 받아온 ExchangeFinder객체로 Exchange객체를 생성한다

이 클래스는 네트웨크 요청의 헤더, 바디를 작성하거나 읽는 역할이다 

그리고 Exchange객체를 interceptorScopedExchange, exchange변수에 각각 저장하는데 
RetryAndFollowUpInterceptor에서 interceptorScopedExchange 변수를 참조해서 연결을 재사용한다


이게 끝이다 그런데 뭔가 중간에 길을잘못 찾은거같기도 하고 내가봐도 이해가 안되고..

다음번에는 okhttp 전체를 이해하고 connection pool 애 대해서 설명을 해야겠다
라이브러리 뜯어보는게 재미 있어서 그냥 막 시작했는데 조금 급했던거같다
