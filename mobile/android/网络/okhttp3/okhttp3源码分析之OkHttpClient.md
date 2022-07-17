# OkHttpClient

OkHttpClient类中设置了一些基本配置
```
 // 默认协议，http2 和http1.1
  static final List<Protocol> DEFAULT_PROTOCOLS = Util.immutableList(
      Protocol.HTTP_2, Protocol.HTTP_1_1);

  //默认的密钥算法套件
  static final List<ConnectionSpec> DEFAULT_CONNECTION_SPECS = Util.immutableList(
      ConnectionSpec.MODERN_TLS, ConnectionSpec.CLEARTEXT);
      
  //调度器
  final Dispatcher dispatcher;
  //代理
  final @Nullable Proxy proxy;
  //协议
  final List<Protocol> protocols;
  //连接规格--- 密钥算法
  final List<ConnectionSpec> connectionSpecs;
  //拦截器
  final List<Interceptor> interceptors;
  //网络拦截器
  final List<Interceptor> networkInterceptors;
  final EventListener.Factory eventListenerFactory;
  final ProxySelector proxySelector;
  //cookie
  final CookieJar cookieJar;
  //缓存
  final @Nullable Cache cache;
  //内部缓存
  final @Nullable InternalCache internalCache;
  final SocketFactory socketFactory;
  final SSLSocketFactory sslSocketFactory;
  final CertificateChainCleaner certificateChainCleaner;
  //域名验证
  final HostnameVerifier hostnameVerifier;
  final CertificatePinner certificatePinner;
  final Authenticator proxyAuthenticator;
  final Authenticator authenticator;
  //连接池
  final ConnectionPool connectionPool;
  //域名解析
  final Dns dns;
  final boolean followSslRedirects;
  final boolean followRedirects;
  final boolean retryOnConnectionFailure;
  final int callTimeout;
  final int connectTimeout;
  final int readTimeout;
  final int writeTimeout;
  final int pingInterval;
```

### OkHttpClient的内部类Builder
通过OkHttpClient的默认构造方法，我们可以看到需要传递一个Builder实例
```
 public OkHttpClient() {
    //构造 默认创建Builder()方便使用
    this(new Builder());
  }
OkHttpClient(Builder builder) {
    this.dispatcher = builder.dispatcher;
    this.proxy = builder.proxy;
    this.protocols = builder.protocols;
    this.connectionSpecs = builder.connectionSpecs;
    this.interceptors = Util.immutableList(builder.interceptors);
    this.networkInterceptors = Util.immutableList(builder.networkInterceptors);
    this.eventListenerFactory = builder.eventListenerFactory;
    this.proxySelector = builder.proxySelector;
    this.cookieJar = builder.cookieJar;
    this.cache = builder.cache;
    this.internalCache = builder.internalCache;
    this.socketFactory = builder.socketFactory;

    boolean isTLS = false;
    for (ConnectionSpec spec : connectionSpecs) {
      isTLS = isTLS || spec.isTls();
    }

    if (builder.sslSocketFactory != null || !isTLS) {
      this.sslSocketFactory = builder.sslSocketFactory;
      this.certificateChainCleaner = builder.certificateChainCleaner;
    } else {
      X509TrustManager trustManager = Util.platformTrustManager();
      this.sslSocketFactory = newSslSocketFactory(trustManager);
      this.certificateChainCleaner = CertificateChainCleaner.get(trustManager);
    }

    if (sslSocketFactory != null) {
      Platform.get().configureSslSocketFactory(sslSocketFactory);
    }

    this.hostnameVerifier = builder.hostnameVerifier;
    this.certificatePinner = builder.certificatePinner.withCertificateChainCleaner(
        certificateChainCleaner);
    this.proxyAuthenticator = builder.proxyAuthenticator;
    this.authenticator = builder.authenticator;
    this.connectionPool = builder.connectionPool;
    this.dns = builder.dns;
    this.followSslRedirects = builder.followSslRedirects;
    this.followRedirects = builder.followRedirects;
    this.retryOnConnectionFailure = builder.retryOnConnectionFailure;
    this.callTimeout = builder.callTimeout;
    this.connectTimeout = builder.connectTimeout;
    this.readTimeout = builder.readTimeout;
    this.writeTimeout = builder.writeTimeout;
    this.pingInterval = builder.pingInterval;

    if (interceptors.contains(null)) {
      throw new IllegalStateException("Null interceptor: " + interceptors);
    }
    if (networkInterceptors.contains(null)) {
      throw new IllegalStateException("Null network interceptor: " + networkInterceptors);
    }
  }

```
从构造方法可以看到OkHttpClient中的各种配置（属性），需要通过Builder来赋值。那么我们就看一下Builder中的代码
```
  public Builder() {
      //调度器
      dispatcher = new Dispatcher();
      //默认协议
      protocols = DEFAULT_PROTOCOLS;
      //默认的
      connectionSpecs = DEFAULT_CONNECTION_SPECS;
      eventListenerFactory = EventListener.factory(EventListener.NONE);
      //代理
      proxySelector = ProxySelector.getDefault();
      if (proxySelector == null) {
        proxySelector = new NullProxySelector();
      }
      //cookie
      cookieJar = CookieJar.NO_COOKIES;
      //socket工厂
      socketFactory = SocketFactory.getDefault();
      //域名验证
      hostnameVerifier = OkHostnameVerifier.INSTANCE;
      certificatePinner = CertificatePinner.DEFAULT;
      proxyAuthenticator = Authenticator.NONE;
      authenticator = Authenticator.NONE;
      //连接池
      connectionPool = new ConnectionPool();
      //域名解析
      dns = Dns.SYSTEM;
      followSslRedirects = true;
      followRedirects = true;
      retryOnConnectionFailure = true;
      callTimeout = 0;
      connectTimeout = 10_000;
      readTimeout = 10_000;
      writeTimeout = 10_000;
      pingInterval = 0;
    }
```
从上方Builder的构造方法可以看出来，Builder给这些属性设置默认配置。当然也可以在OkHttpClient中先给这些属性赋值，然后再传给Builder
```
 Builder(OkHttpClient okHttpClient) {
      this.dispatcher = okHttpClient.dispatcher;
      this.proxy = okHttpClient.proxy;
      this.protocols = okHttpClient.protocols;
      this.connectionSpecs = okHttpClient.connectionSpecs;
      this.interceptors.addAll(okHttpClient.interceptors);
      this.networkInterceptors.addAll(okHttpClient.networkInterceptors);
      this.eventListenerFactory = okHttpClient.eventListenerFactory;
      this.proxySelector = okHttpClient.proxySelector;
      this.cookieJar = okHttpClient.cookieJar;
      this.internalCache = okHttpClient.internalCache;
      this.cache = okHttpClient.cache;
      this.socketFactory = okHttpClient.socketFactory;
      this.sslSocketFactory = okHttpClient.sslSocketFactory;
      this.certificateChainCleaner = okHttpClient.certificateChainCleaner;
      this.hostnameVerifier = okHttpClient.hostnameVerifier;
      this.certificatePinner = okHttpClient.certificatePinner;
      this.proxyAuthenticator = okHttpClient.proxyAuthenticator;
      this.authenticator = okHttpClient.authenticator;
      this.connectionPool = okHttpClient.connectionPool;
      this.dns = okHttpClient.dns;
      this.followSslRedirects = okHttpClient.followSslRedirects;
      this.followRedirects = okHttpClient.followRedirects;
      this.retryOnConnectionFailure = okHttpClient.retryOnConnectionFailure;
      this.callTimeout = okHttpClient.callTimeout;
      this.connectTimeout = okHttpClient.connectTimeout;
      this.readTimeout = okHttpClient.readTimeout;
      this.writeTimeout = okHttpClient.writeTimeout;
      this.pingInterval = okHttpClient.pingInterval;
    }
```
Builder中各属性的配置是通过链式调用的方式来配置的（每个方法的返回值都是Builder本身）

设置完这些属性后通过build()返回OkHttpClient实例
```
public OkHttpClient build() {
      return new OkHttpClient(this);
    }
```

okHttpClient调用newCall()方法来设置请求参数和url（在Request中配置）
```
 @Override public Call newCall(Request request) {
    return RealCall.newRealCall(this, request, false /* for web socket */);
  }

//webSocket 服务器和客户端进行双向通信，服务端可以主动向客户端推送数据
 @Override public WebSocket newWebSocket(Request request, WebSocketListener listener) {
    RealWebSocket webSocket = new RealWebSocket(request, listener, new Random(), pingInterval);
    webSocket.connect(this);
    return webSocket;
  }
```
通过调用返回值Call接口中的execute()执行同步请求；enqueue(Callback responseCallback)方法执行异步请求
```
public interface Call extends Cloneable {
  //请求url和参数等配置
  Request request();

 //同步请求
  Response execute() throws IOException;

 //异步请求
  void enqueue(Callback responseCallback);
  //取消请求
  void cancel();

  //是否已执行
  boolean isExecuted();
  //是否已取消
  boolean isCanceled();

  //超时
  Timeout timeout();

  //克隆一个新的Call
  Call clone();
  //工厂接口
  interface Factory {
    //创建一个新的Call
    Call newCall(Request request);
  }
}
```
接下来看一下Request类的内部实现
```
public final class Request {
  //请求url
  final HttpUrl url;
  //请求方式get、post、delete等
  final String method;
  //请求头
  final Headers headers;
  //请求body
  final @Nullable RequestBody body;
  //tag
  final Map<Class<?>, Object> tags;
  //缓存控制
  private volatile @Nullable CacheControl cacheControl; // Lazily initialized.
  
  public Request(Builder builder) {
    this.url = builder.url;
    this.method = builder.method;
    this.headers = builder.headers.build();
    this.body = builder.body;
    this.tags = Util.immutableMap(builder.tags);
  }
  public HttpUrl url() {
    return url;
  }
  public String method() {
    return method;
  }
  public Headers headers() {
    return headers;
  }
  public @Nullable String header(String name) {
    return headers.get(name);
  }
  public List<String> headers(String name) {
    return headers.values(name);
  }
  public @Nullable RequestBody body() {
    return body;
  }
  public @Nullable Object tag() {
    return tag(Object.class);
  }
  public @Nullable <T> T tag(Class<? extends T> type) {
    return type.cast(tags.get(type));
  }
  public Builder newBuilder() {
    return new Builder(this);
  }
  public CacheControl cacheControl() {
    CacheControl result = cacheControl;
    return result != null ? result : (cacheControl = CacheControl.parse(headers));
  }
  public boolean isHttps() {
    return url.isHttps();
  }

  @Override public String toString() {
    return "Request{method="
        + method
        + ", url="
        + url
        + ", tags="
        + tags
        + '}';
  }
  //给属性赋值的内部类
  public static class Builder {
    @Nullable HttpUrl url;
    String method;
    Headers.Builder headers;
    @Nullable RequestBody body;

    Map<Class<?>, Object> tags = Collections.emptyMap();
    //默认为GET请求
    public Builder() {
      this.method = "GET";
      this.headers = new Headers.Builder();
    }

    Builder(Request request) {
      this.url = request.url;
      this.method = request.method;
      this.body = request.body;
      this.tags = request.tags.isEmpty()
          ? Collections.emptyMap()
          : new LinkedHashMap<>(request.tags);
      this.headers = request.headers.newBuilder();
    }

    public Builder url(HttpUrl url) {
      if (url == null) throw new NullPointerException("url == null");
      this.url = url;
      return this;
    }
    public Builder url(String url) {
      if (url == null) throw new NullPointerException("url == null");

      // Silently replace web socket URLs with HTTP URLs.
      if (url.regionMatches(true, 0, "ws:", 0, 3)) {
        url = "http:" + url.substring(3);
      } else if (url.regionMatches(true, 0, "wss:", 0, 4)) {
        url = "https:" + url.substring(4);
      }

      return url(HttpUrl.get(url));
    }
    public Builder url(URL url) {
      if (url == null) throw new NullPointerException("url == null");
      return url(HttpUrl.get(url.toString()));
    }
    public Builder header(String name, String value) {
      headers.set(name, value);
      return this;
    }
    public Builder addHeader(String name, String value) {
      headers.add(name, value);
      return this;
    }

    /** Removes all headers named {@code name} on this builder. */
    public Builder removeHeader(String name) {
      headers.removeAll(name);
      return this;
    }

    /** Removes all headers on this builder and adds {@code headers}. */
    public Builder headers(Headers headers) {
      this.headers = headers.newBuilder();
      return this;
    }

    public Builder cacheControl(CacheControl cacheControl) {
      String value = cacheControl.toString();
      if (value.isEmpty()) return removeHeader("Cache-Control");
      return header("Cache-Control", value);
    }

    public Builder get() {
      return method("GET", null);
    }

    public Builder head() {
      return method("HEAD", null);
    }

    public Builder post(RequestBody body) {
      return method("POST", body);
    }

    public Builder delete(@Nullable RequestBody body) {
      return method("DELETE", body);
    }

    public Builder delete() {
      return delete(Util.EMPTY_REQUEST);
    }

    public Builder put(RequestBody body) {
      return method("PUT", body);
    }

    public Builder patch(RequestBody body) {
      return method("PATCH", body);
    }

    public Builder method(String method, @Nullable RequestBody body) {
      if (method == null) throw new NullPointerException("method == null");
      if (method.length() == 0) throw new IllegalArgumentException("method.length() == 0");
      if (body != null && !HttpMethod.permitsRequestBody(method)) {
        throw new IllegalArgumentException("method " + method + " must not have a request body.");
      }
      if (body == null && HttpMethod.requiresRequestBody(method)) {
        throw new IllegalArgumentException("method " + method + " must have a request body.");
      }
      this.method = method;
      this.body = body;
      return this;
    }

    public Builder tag(@Nullable Object tag) {
      return tag(Object.class, tag);
    }

    public <T> Builder tag(Class<? super T> type, @Nullable T tag) {
      if (type == null) throw new NullPointerException("type == null");

      if (tag == null) {
        tags.remove(type);
      } else {
        if (tags.isEmpty()) tags = new LinkedHashMap<>();
        tags.put(type, type.cast(tag));
      }

      return this;
    }
    //build返回Request
    public Request build() {
      if (url == null) throw new IllegalStateException("url == null");
      return new Request(this);
    }
  }
}
```
从Request类中的代码可以看到，其内部通过内部类Builder来给url，请求方式、缓存、header等属性赋值，最后通过Builder类中的build()方法来返回Request实例

### OkHttp的使用
从以上分析可以看出来，OkHttp的使用方式有两种
```
//第一种（执行的同步请求）
 OkHttpClient okHttpClient = new OkHttpClient();
 Request request = new Request.Builder()
                .url("www.baidu.com")
                .build();
 try {
    //同步执行请求
    Response response = okHttpClient.newCall(request).execute();
    //请求结果
    String str = response.body().toString();
} catch (IOException e) {
    e.printStackTrace();
    }

//第二种（执行的异步请求）
OkHttpClient.Builder builder = new OkHttpClient.Builder();
builder.connectTimeout(20,TimeUnit.SECONDS);
OkHttpClient client = builder.build();
client.newCall(new Request.Builder().url("sdfs").build()).enqueue(new Callback() {
        @Override
        public void onFailure(Call call, IOException e) {}

        @Override
        public void onResponse(Call call, Response response) throws IOException {}
});

```
以上两种方法的区别是：第一种使用okhttp的默认配置执行请求；第二种是自己配置超时时间等连接参数后执行请求





