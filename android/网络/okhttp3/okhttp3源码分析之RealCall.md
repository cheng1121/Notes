# RealCall
上一篇文章分析了OkhttpClient类和Request类的作用和其内部的Builder类，主要是用来构造请求，参数、url、超时时间、拦截器等等。并没有真正开始执行请求，真正开始执行请求是在RealCall类中，下面我们就开始分析RealCall类。
```
  //OkhttpClient类中真正开始执行网络请求的方法，其中通过RealCall类实现
  @Override public Call newCall(Request request) {
    return RealCall.newRealCall(this, request, false /* for web socket */);
  }
  
  //RealCall的构造方法
  private RealCall(OkHttpClient client, Request originalRequest, boolean forWebSocket) {
    this.client = client;
    this.originalRequest = originalRequest;
    this.forWebSocket = forWebSocket;
  }
```
通过RalCall的构造方法，可以看到需要传OkHttpClient、Request和forWebSocket,三个参数，第三个参数为是否为WebSocket

接下来看下执行同步请求的方法
```
 //同步请求
  @Override public Response execute() throws IOException {
    synchronized (this) {// 1
      if (executed) throw new IllegalStateException("Already Executed");
      executed = true;
    }
    //开始计时和添加事件监听 2
    transmitter.timeoutEnter();
    transmitter.callStart();
    try {
      //通过调度器 dispatcher来执行同步请求 3
      client.dispatcher().executed(this);
      //获取响应结果，和设置拦截器 4
      return getResponseWithInterceptorChain();
    } finally {
      //通知调度器dispatcher执行完成 5
      client.dispatcher().finished(this);
    }
  }
```
里边的逻辑很简单：
1. 判断同步请求是否正在执行，执行就抛出异常，未执行就设置为执行
2. 通过transmitter 发送器，设置请求开始时间，和事件监听
3. 通过OkhttpClient中的调度器dispatcher来实际执行
4. 通过getResponseWithInterceptorChain()方法得到响应(该方法内部实现一会分析)
5. 通知调度器任务执行完毕（调度器Dispatcher类会单独分析）

接下来看下异步请求：
```
  //异步请求
  @Override public void enqueue(Callback responseCallback) {
    synchronized (this) { //判断是否正在执行
      if (executed) throw new IllegalStateException("Already Executed");
      executed = true;
    }
    //设置事件监听
    transmitter.callStart();
    //通知调度器开始执行 异步请求，（dispatcher内部会设置异步请求的线程池等操作）
    client.dispatcher().enqueue(new AsyncCall(responseCallback));
  }
```
异步请求在Dispatcher中设置一些请求策略，如：请求连接是否已存在（根据主机名host）、是否达到请求上限、每个主机同时执行的请求数、线程池等等。
之后会回调RealCall的内部类AsyncCall的executeOn方法执行任务，AsyncCall类实现了NamedRunnable类，所以ExecutorService可以直接调用this来执行任务
```
  /**
     * 执行异步请求并得到响应
     */
    void executeOn(ExecutorService executorService) {
      assert (!Thread.holdsLock(client.dispatcher()));
      boolean success = false;
      try {
        //执行任务
        executorService.execute(this);//这个this指的是execute()方法
        success = true;
      } catch (RejectedExecutionException e) {
          。。。
      } finally {
        if (!success) {
          client.dispatcher().finished(this); // This call is no longer running!
        }
      }
    }
    //执行请求
    @Override protected void execute() {
      boolean signalledCallback = false;
      transmitter.timeoutEnter();
      try {
        //得到异步请求响应 response
        Response response = getResponseWithInterceptorChain();
        signalledCallback = true;
        //返回请求结果
        responseCallback.onResponse(RealCall.this, response);
      } catch (IOException e) {
        if (signalledCallback) {
          // Do not signal the callback twice!
          Platform.get().log(INFO, "Callback failure for " + toLoggableString(), e);
        } else {
          //请求错误
          responseCallback.onFailure(RealCall.this, e);
        }
      } finally {
        //通知调度器任务执行完成
        client.dispatcher().finished(this);
      }
    }
```

### getResponseWithInterceptorChain()方法
该方法是连接服务器、设置请求缓存、连接池、重连等等一系列的拦截器的关键方法，并且获得响应结果
```
 //获取响应结果  //责任链模式
  Response getResponseWithInterceptorChain() throws IOException {
    // Build a full stack of interceptors.
    List<Interceptor> interceptors = new ArrayList<>();
    //用户自定义的拦截器
    interceptors.addAll(client.interceptors());
    //重连和重定向拦截器
    interceptors.add(new RetryAndFollowUpInterceptor(client));
    //负责把请求转换为发送到服务器的请求，把响应转换为用户友好的响应
    interceptors.add(new BridgeInterceptor(client.cookieJar()));
    //缓存拦截器
    interceptors.add(new CacheInterceptor(client.internalCache()));
    //连接拦截器
    interceptors.add(new ConnectInterceptor(client));
    if (!forWebSocket) {
      //WebSocket的拦截器
      interceptors.addAll(client.networkInterceptors());
    }
    //向服务器发送请求数据，从服务器获取响应数据的拦截器
    interceptors.add(new CallServerInterceptor(forWebSocket));

    //开启链式调用
    Interceptor.Chain chain = new RealInterceptorChain(interceptors, transmitter, null, 0,
        originalRequest, this, client.connectTimeoutMillis(),
        client.readTimeoutMillis(), client.writeTimeoutMillis());

    boolean calledNoMoreExchanges = false;
    try {
      //返回响应结果 //proceed实现链式调用
      Response response = chain.proceed(originalRequest);
      if (transmitter.isCanceled()) {
        closeQuietly(response);
        throw new IOException("Canceled");
      }
      return response;
    } catch (IOException e) {
      calledNoMoreExchanges = true;
      throw transmitter.noMoreExchanges(e);
    } finally {
      if (!calledNoMoreExchanges) {
        transmitter.noMoreExchanges(null);
      }
    }
  }
```
接下来看一下拦截器的责任链模式是怎么实现的。
从源码中可以知道具体的实现类是 RealInterceptorChain，方法为proceed方法
```
 //getResponseWithInterceptorChain方法中调用的proceed方法
 @Override public Response proceed(Request request) throws IOException {
    return proceed(request, transmitter, exchange);
  }
  //proceed的重载方法
  public Response proceed(Request request, Transmitter transmitter, @Nullable Exchange exchange)
      throws IOException {
    if (index >= interceptors.size()) throw new AssertionError();
    
    calls++;
    
    //如果我们已经有了一个流，请确认传入的请求将使用它。
    if (this.exchange != null && !this.exchange.connection().supportsUrl(request.url())) {
      throw new IllegalStateException("network interceptor " + interceptors.get(index - 1)
          + " must retain the same host and port");
    }

    //如果我们已经有了一个流，请确认这是对chain.proceed（）的唯一调用
    if (this.exchange != null && calls > 1) {
      throw new IllegalStateException("network interceptor " + interceptors.get(index - 1)
          + " must call proceed() exactly once");
    }

    //调用链中的下一个拦截器
    RealInterceptorChain next = new RealInterceptorChain(interceptors, transmitter, exchange,
        index + 1, request, call, connectTimeout, readTimeout, writeTimeout);
    Interceptor interceptor = interceptors.get(index);
    Response response = interceptor.intercept(next);

    // Confirm that the next interceptor made its required call to chain.proceed().
    //确认下一个拦截器对chain.proceed（）进行了所需的调用。
    if (exchange != null && index + 1 < interceptors.size() && next.calls != 1) {
      throw new IllegalStateException("network interceptor " + interceptor
          + " must call proceed() exactly once");
    }

    // Confirm that the intercepted response isn't null.
    //确认响应不为null
    if (response == null) {
      throw new NullPointerException("interceptor " + interceptor + " returned null");
    }

    if (response.body() == null) {
      throw new IllegalStateException(
          "interceptor " + interceptor + " returned a response with no body");
    }

    return response;
  }
```
从上方源码可以看到关键的代码就是这几行代码
```
  //1
  RealInterceptorChain next = new RealInterceptorChain(interceptors, transmitter, exchange,
        index + 1, request, call, connectTimeout, readTimeout, writeTimeout); 
    //2
    Interceptor interceptor = interceptors.get(index);
    //3
    Response response = interceptor.intercept(next);

```
逻辑是
1. 实例化下一个拦截器的RealInterceptorChain
2. 获取当前拦截器
3. 执行拦截器，并获取响应，并把下一个拦截器的RealInterceptorChain传递下去
