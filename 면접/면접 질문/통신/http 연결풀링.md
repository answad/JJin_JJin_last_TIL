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
함수 내부에서 getResponseWithInterceptorChain를 호출하는데 
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
    if (call.isCanceled()) throw IOException("Canceled")

    // Attempt to reuse the connection from the call.
    val callConnection = call.connection // This may be mutated by releaseConnectionNoEvents()!
    if (callConnection != null) {
      var toClose: Socket? = null
      synchronized(callConnection) {
        if (callConnection.noNewExchanges || !sameHostAndPort(callConnection.route().address.url)) {
          toClose = call.releaseConnectionNoEvents()
        }
      }

      // If the call's connection wasn't released, reuse it. We don't call connectionAcquired() here
      // because we already acquired it.
      if (call.connection != null) {
        check(toClose == null)
        return callConnection
      }

      // The call's connection was released.
      toClose?.closeQuietly()
      eventListener.connectionReleased(call, callConnection)
    }

    // We need a new connection. Give it fresh stats.
    refusedStreamCount = 0
    connectionShutdownCount = 0
    otherFailureCount = 0

    // Attempt to get a connection from the pool.
    if (connectionPool.callAcquirePooledConnection(address, call, null, false)) {
      val result = call.connection!!
      eventListener.connectionAcquired(call, result)
      return result
    }

    // Nothing in the pool. Figure out what route we'll try next.
    val routes: List<Route>?
    val route: Route
    if (nextRouteToTry != null) {
      // Use a route from a preceding coalesced connection.
      routes = null
      route = nextRouteToTry!!
      nextRouteToTry = null
    } else if (routeSelection != null && routeSelection!!.hasNext()) {
      // Use a route from an existing route selection.
      routes = null
      route = routeSelection!!.next()
    } else {
      // Compute a new route selection. This is a blocking operation!
      var localRouteSelector = routeSelector
      if (localRouteSelector == null) {
        localRouteSelector = RouteSelector(address, call.client.routeDatabase, call, eventListener)
        this.routeSelector = localRouteSelector
      }
      val localRouteSelection = localRouteSelector.next()
      routeSelection = localRouteSelection
      routes = localRouteSelection.routes

      if (call.isCanceled()) throw IOException("Canceled")

      // Now that we have a set of IP addresses, make another attempt at getting a connection from
      // the pool. We have a better chance of matching thanks to connection coalescing.
      if (connectionPool.callAcquirePooledConnection(address, call, routes, false)) {
        val result = call.connection!!
        eventListener.connectionAcquired(call, result)
        return result
      }

      route = localRouteSelection.next()
    }

    // Connect. Tell the call about the connecting call so async cancels work.
    val newConnection = RealConnection(connectionPool, route)
    call.connectionToCancel = newConnection
    try {
      newConnection.connect(
          connectTimeout,
          readTimeout,
          writeTimeout,
          pingIntervalMillis,
          connectionRetryEnabled,
          call,
          eventListener
      )
    } finally {
      call.connectionToCancel = null
    }
    call.client.routeDatabase.connected(newConnection.route())

    // If we raced another call connecting to this host, coalesce the connections. This makes for 3
    // different lookups in the connection pool!
    if (connectionPool.callAcquirePooledConnection(address, call, routes, true)) {
      val result = call.connection!!
      nextRouteToTry = route
      newConnection.socket().closeQuietly()
      eventListener.connectionAcquired(call, result)
      return result
    }

    synchronized(newConnection) {
      connectionPool.put(newConnection)
      call.acquireConnectionNoEvents(newConnection)
    }

    eventListener.connectionAcquired(call, newConnection)
    return newConnection
  }
}
```
.... 오늘은 여기까지..
