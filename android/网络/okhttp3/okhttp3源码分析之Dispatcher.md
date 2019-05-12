# Dispatcher
调度器，内部配置了请求队列，最大并发请求数，线程池等等配置

```
  //最大并发请求 64
  private int maxRequests = 64;
  //每个主机同时执行的最大请求
  private int maxRequestsPerHost = 5;
  private @Nullable Runnable idleCallback;

  /** 线程池 */
  private @Nullable ExecutorService executorService;

  /** 准备执行的异步请求队列 */
  private final Deque<AsyncCall> readyAsyncCalls = new ArrayDeque<>();

  /** 正在执行情的请求，包含已经取消但还未执行完的请求  异步请求队列*/
  private final Deque<AsyncCall> runningAsyncCalls = new ArrayDeque<>();

  /** 同上  同步请求队列 */
  private final Deque<RealCall> runningSyncCalls = new ArrayDeque<>();

```


核心方法有以下几种：
```
//异步请求
  void enqueue(AsyncCall call) {
    synchronized (this) {
      //添加到异步请求准备执行队列
      readyAsyncCalls.add(call);

      // Mutate the AsyncCall so that it shares the AtomicInteger of an existing running call to
      // the same host.
      if (!call.get().forWebSocket) {
        //根据主机名判断是否有已存在的调用
        AsyncCall existingCall = findExistingCallWithHost(call.host());
        if (existingCall != null) call.reuseCallsPerHostFrom(existingCall);
      }
    }
    //执行任务
    promoteAndExecute();
  }
  
  private boolean promoteAndExecute() {
    assert (!Thread.holdsLock(this));

    List<AsyncCall> executableCalls = new ArrayList<>();
    boolean isRunning;
    synchronized (this) {
      for (Iterator<AsyncCall> i = readyAsyncCalls.iterator(); i.hasNext(); ) {
        AsyncCall asyncCall = i.next();
        //如果准备队列的大小大于最大请求数   
        if (runningAsyncCalls.size() >= maxRequests) break; // Max capacity.
        //一个主机同时执行的最大请求
        if (asyncCall.callsPerHost().get() >= maxRequestsPerHost) continue; // Host max capacity.
        
        i.remove();
        asyncCall.callsPerHost().incrementAndGet();
        executableCalls.add(asyncCall);
        runningAsyncCalls.add(asyncCall);
      }
      isRunning = runningCallsCount() > 0;
    }

    for (int i = 0, size = executableCalls.size(); i < size; i++) {
      AsyncCall asyncCall = executableCalls.get(i);
      //执行异步请求（使用线程池）
      asyncCall.executeOn(executorService());
    }

    return isRunning;
  }
  
```

同步请求
```
  //把请求添加到同步请求队列中
  
  synchronized void executed(RealCall call) {
    runningSyncCalls.add(call);
  }

```

