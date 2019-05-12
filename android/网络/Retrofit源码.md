# Retrofit源码

## Retrofit的使用
### 配置build.gradle
再dependencies节点中添加
```
//retrofit
implementation 'com.squareup.retrofit2:retrofit:2.3.0'
//gson
implementation "com.squareup.retrofit2:converter-gson:2.3.0"
//rxjava 可选
implementation "com.squareup.retrofit2:adapter-rxjava2:2.3.0"
//okhttp3 log打印 可选
implementation "com.squareup.okhttp3:logging-interceptor:3.9.0"
```

### 注解
分为3大类，HTTP请求方法注解、标记类注解、参数类注解

HTTP请求方法注解：
1. GET
2. POST
3. PUT
4. DELETE
5. HEAD
6. PATCH
7. OPTIONS
8. HTTP

前7种分别对应HTTP的请求方法；HTTP可以替换以上7种，也可以拓展请求方法

标记类注解：
1. FormUrlEncoded
2. Multipart
3. Streaming：响应的数据以流的形式返回，如果不使用它，则默认会把全部数据加载到内存，所以下载大文件时需要加上这个注解


参数类注解：
1. Header
2. Headers
3. Body
4. Path
5. Field
6. FieldMap
7. Part
8. PartMap
9. Query
10. QueryMap


### 请求网络

#### GET请求

网络请求接口
```
public interface IpService{
    @GET("getIpInfo.php?ip=59.108.54.37")
    Call<IpModel> getIpMsg();
}
```
创建Retrofit:
```
  String mBaseUrl = "http://ip.taobao.com/service/"
   mRetrofit = new Retrofit.Builder()
                 //okHttpClient.Builder() 配置连接参数
                .client(mBuilder.build())
                //添加返回值GSON支持 .addConverterFactory(GsonConverterFactory.create())
                //添加rxjava支持 .addCallAdapterFactory(RxJava2CallAdapterFactory.create())
                //域名
                .baseUrl(mBaseUrl)
                .build();
  IpService ipService =    mRetrofit.create(IpService.class);
    Call<IpModel> call = ipService.getIpMsg();
```

使用call异步请求网络
```
call.enqueue(new Callback<IpModel>(){
    @Override
    public void onResponse(Call<IpModel> call, Response<IpModel> response){
        String country = response.body().getData().getCountry();
        Toast.makeText(getApplicationContext(),country,Toast.LENGTH_SHORT).show();
    }
    
    @Override
    public void onFailure(Call<IpModel> call,Throwable t){
        
    }
})；
```
#### 动态配置URL地址：@PATH
```
public interface IpServiceForPath{
    @GET("{path}/getIpInfo.php?ip=59.108.54.37")
    Call<IpModel> getIpMsg(@Path("path") String path);
}
```

#### 动态指定查询条件：@Query
```
public interface IpServiceForQuery{
    @GET("getIpInfo.php")
    Call<IpModel> getIpMsg(@Query("ip") String ip);
}
```
#### @QueryMap
```
public interface IpServiceForQueryMap{
    @GET("getIpInfo.php")
    Call<IpModel> getIpMsg(@QueryMap Map<String,String> options);
}
```

### POST请求

#### 传输数据类型为键值对：@Field
传输数据类型为键值对，这是我们最常用的POST请求数据类型
```
public interface IpServiceForPost{
    @FormUrlEncoded
    @POST("getIpInfo.php")
    Call<IpModel> getIpMsg(@Field("ip") String ip);
}
```
1. @FormUrlEncoded注解来表明这是一个表单请求
2. @Field注解标示所对应的String类型数据的键


#### 传输数据类型JSON字符串：@Body
```
public interface IpServiceForPostBody{
    @POST("getIpInfo.php")
    Call<IpModel> getIpMsg(@Body Ip ip);
}
```
用@Body这个注解标识参数对象即可，Retrofit会将Ip对象转为字符串

#### 单个文件上传：@Part
```
public interface UploadFieldForPart{
    @Multipart
    @POST("user/photo")
    Call<User> updateUser(@Part MultipartBody.Part photo,@Part("description") RequestBody description);
}
```

@Multipart注解表示允许多个@Part，updateUser方法的第一个参数是准备上传的图片文件，使用了MultipartBody.Part类型；另一个参数是RequestBody类型，它用来传递简单的键值对
```
   Retrofit retrofit = new Retrofit.Builder()
                .baseUrl("url")
                .addConverterFactory(GsonConverterFactory.create())
                .build();
        File  file = new File("file path","file name");
        RequestBody photoRequestBody = RequestBody.create(MediaType.parse("image/png"),file);
        MultipartBody.Part photo = MultipartBody.Part.createFormData("photos","file name",photoRequestBody);
        DemoService service = retrofit.create(DemoService.class);
        Call call = service.updateUser(photo,RequestBody.create(null,"photo"));
        
```
#### 多个文件上传：@PartMap
```
    @Multipart
    @POST("user/photo")
    Call<UserData> updateUser(@PartMap Map<String,RequestBody> photos,@Part("description") RequestBody description);
```

### 消息报头Header
在HTTP请求中，为了防止攻击或过滤掉不安全的访问，或者添加特殊加密的访问等，以便减轻服务器的压力和保证请求的安全，通常都会在消息报头中携带一些特殊的消息头处理

#### 静态添加Header
```
@GET("some/endpoint")
    @Headers("Accept-Encoding: application/json")
    Call get();
```
#### 动态添加Header
```
   @GET("some/endpoint")
    Call get(@Header("Location") String location);
```

## 源码解析Retrofit

### Retrofit的创建过程
使用如下方法创建Retrofit
```
Retrofit retrofit = new Retrofit.Builder()
                .baseUrl("url")
                .addConverterFactory(GsonConverterFactory.create())
                .build();
```

接着看下Builder()方法做了什么
```
  public Builder() {
      this(Platform.get());
    }
```
继续看Platform.get方法
```
 private static final Platform PLATFORM = findPlatform();

  static Platform get() {
    return PLATFORM;
  }

  private static Platform findPlatform() {
    try {
      Class.forName("android.os.Build");
      if (Build.VERSION.SDK_INT != 0) {
        return new Android();
      }
    } catch (ClassNotFoundException ignored) {
    }
    try {
      Class.forName("java.util.Optional");
      return new Java8();
    } catch (ClassNotFoundException ignored) {
    }
    return new Platform();
  }
```

可以看到最终调用的是findPlatform方法，findPlatform方法中会根据运行的平台来返回Android()、Java8()或者Platform()

接下来继续看下Android对象、Java8、Platform的作用
Android：
```
  static class Android extends Platform {
    @Override public Executor defaultCallbackExecutor() {
      return new MainThreadExecutor();
    }

    @Override CallAdapter.Factory defaultCallAdapterFactory(@Nullable Executor callbackExecutor) {
      if (callbackExecutor == null) throw new AssertionError();
      return new ExecutorCallAdapterFactory(callbackExecutor);
    }

    static class MainThreadExecutor implements Executor {
      private final Handler handler = new Handler(Looper.getMainLooper());

      @Override public void execute(Runnable r) {
        handler.post(r);
      }
    }
  }

```
Android类的实现主要是获取相应的线程池，如果设置的话，则获取默认的。默认的就是Android的Handler；自己设置的就是我们addCallAdapterFactory()添加的,如rxjava

java8:
```
  @IgnoreJRERequirement // Only classloaded and used on Java 8.
  static class Java8 extends Platform {
    @Override boolean isDefaultMethod(Method method) {
      return method.isDefault();
    }

    @Override Object invokeDefaultMethod(Method method, Class<?> declaringClass, Object object,
        @Nullable Object... args) throws Throwable {
      // Because the service interface might not be public, we need to use a MethodHandle lookup
      // that ignores the visibility of the declaringClass.
      Constructor<Lookup> constructor = Lookup.class.getDeclaredConstructor(Class.class, int.class);
      constructor.setAccessible(true);
      return constructor.newInstance(declaringClass, -1 /* trusted */)
          .unreflectSpecial(method, declaringClass)
          .bindTo(object)
          .invokeWithArguments(args);
    }
  }
```


接下来看build方法的实现
```
/**
     * Create the {@link Retrofit} instance using the configured values.
     * <p>
     * Note: If neither {@link #client} nor {@link #callFactory} is called a default {@link
     * OkHttpClient} will be created and used.
     * 创建Retrofit
     */
    public Retrofit build() {
      if (baseUrl == null) { //如果没有baseUrl则抛出异常
        throw new IllegalStateException("Base URL required.");
      }
      //设置默认的callFactory
      okhttp3.Call.Factory callFactory = this.callFactory;
      if (callFactory == null) {
        callFactory = new OkHttpClient();
      }
      //设置默认线程池
      Executor callbackExecutor = this.callbackExecutor;
      if (callbackExecutor == null) {
        callbackExecutor = platform.defaultCallbackExecutor();
      }

      // Make a defensive copy of the adapters and add the default Call adapter.
      //添加用户自定义的Adapter 如：rxjava
      List<CallAdapter.Factory> adapterFactories = new ArrayList<>(this.adapterFactories);
      adapterFactories.add(platform.defaultCallAdapterFactory(callbackExecutor));

      // Make a defensive copy of the converters.
      //添加自定义的converters 如gson
      List<Converter.Factory> converterFactories = new ArrayList<>(this.converterFactories);

      return new Retrofit(callFactory, baseUrl, converterFactories, adapterFactories,
          callbackExecutor, validateEagerly);
    }
```

### Call的创建过程
```
DemoService service = retrofit.create(DemoService.class);
```
首先生成service的接口对象，通过create方法
```
public <T> T create(final Class<T> service) {
    Utils.validateServiceInterface(service);
    if (validateEagerly) {
      eagerlyValidateMethods(service);
    }
    //动态代理对象，生成接口
    return (T) Proxy.newProxyInstance(service.getClassLoader(), new Class<?>[] { service },
        new InvocationHandler() {
          private final Platform platform = Platform.get();
          //
          @Override public Object invoke(Object proxy, Method method, @Nullable Object[] args)
              throws Throwable {
            // If the method is a method from Object then defer to normal invocation.
            if (method.getDeclaringClass() == Object.class) {
              return method.invoke(this, args);
            }
            if (platform.isDefaultMethod(method)) {
              return platform.invokeDefaultMethod(method, service, proxy, args);
            }
            //获取接口中定义的方法
            ServiceMethod<Object, Object> serviceMethod =
                (ServiceMethod<Object, Object>) loadServiceMethod(method);
            OkHttpCall<Object> okHttpCall = new OkHttpCall<>(serviceMethod, args);
            return serviceMethod.callAdapter.adapt(okHttpCall);
          }
        });
  }
```
loadServiceMethod()方法会获取我们接口中定义的方法，并进行缓存
```
  ServiceMethod<?, ?> loadServiceMethod(Method method) {
    ServiceMethod<?, ?> result = serviceMethodCache.get(method);
    if (result != null) return result;

    synchronized (serviceMethodCache) {
      result = serviceMethodCache.get(method);
      if (result == null) {
        result = new ServiceMethod.Builder<>(this, method).build();
        serviceMethodCache.put(method, result);
      }
    }
    return result;
  }
```
serviceMethodCache对方法进行缓存，loadServiceMethod会先查询缓存中是否存在，如果有就使用缓存，如果没有就创建一个，并加入缓存

使用ServiceMethod.Builder.build()方法创建一个方法
```
    public ServiceMethod build() {
      callAdapter = createCallAdapter(); //1
      responseType = callAdapter.responseType(); //2
      if (responseType == Response.class || responseType == okhttp3.Response.class) {
        throw methodError("'"
            + Utils.getRawType(responseType).getName()
            + "' is not a valid response body type. Did you mean ResponseBody?");
      }
      responseConverter = createResponseConverter(); //3

      for (Annotation annotation : methodAnnotations) { 
        parseMethodAnnotation(annotation);//4
      }

      if (httpMethod == null) {
        throw methodError("HTTP method annotation is required (e.g., @GET, @POST, etc.).");
      }

      if (!hasBody) {
        if (isMultipart) {
          throw methodError(
              "Multipart can only be specified on HTTP methods with request body (e.g., @POST).");
        }
        if (isFormEncoded) {
          throw methodError("FormUrlEncoded can only be specified on HTTP methods with "
              + "request body (e.g., @POST).");
        }
      }

      int parameterCount = parameterAnnotationsArray.length;
      parameterHandlers = new ParameterHandler<?>[parameterCount];
      for (int p = 0; p < parameterCount; p++) {
        Type parameterType = parameterTypes[p]; //5
        if (Utils.hasUnresolvableType(parameterType)) {
          throw parameterError(p, "Parameter type must not include a type variable or wildcard: %s",
              parameterType);
        }

        Annotation[] parameterAnnotations = parameterAnnotationsArray[p];
        if (parameterAnnotations == null) {
          throw parameterError(p, "No Retrofit annotation found.");
        }

        parameterHandlers[p] = parseParameter(p, parameterType, parameterAnnotations);
      }

      if (relativeUrl == null && !gotUrl) {
        throw methodError("Missing either @%s URL or @Url parameter.", httpMethod);
      }
      if (!isFormEncoded && !isMultipart && !hasBody && gotBody) {
        throw methodError("Non-body HTTP method cannot contain @Body.");
      }
      if (isFormEncoded && !gotField) {
        throw methodError("Form-encoded method must contain at least one @Field.");
      }
      if (isMultipart && !gotPart) {
        throw methodError("Multipart method must contain at least one @Part.");
      }

      return new ServiceMethod<>(this); //6
    }

```

- 注释1：调用了createCallAdapter方法，它最终会得到我们在构建Retrofit调用build方法时adapterFactories添加的对象的get方法。
- 注释2：调用CallAdapter的responseType得到的返回数据的真实类型
- 注释3：调用createResponseConverter方法来遍历converterFactories列表中存储的Converter.Factory,并返回一个合适的Converter用来转换对象
- 注释4：遍历parseMethodAnnotation方法来对请求和请求地址进行解析
- 注释5：对方法中的参数注解进行解析
- 注释6：创建ServiceMethod类并返回

create方法的返回值中传入了okhttpcall方法，我们来看看这个类的作用


- 创建请求
```
  @Override public synchronized Request request() { //1
    okhttp3.Call call = rawCall;
    if (call != null) {
      return call.request();
    }
    if (creationFailure != null) {
      if (creationFailure instanceof IOException) {
        throw new RuntimeException("Unable to create request.", creationFailure);
      } else {
        throw (RuntimeException) creationFailure;
      }
    }
    try {
      return (rawCall = createRawCall()).request();
    } catch (RuntimeException e) {
      creationFailure = e;
      throw e;
    } catch (IOException e) {
      creationFailure = e;
      throw new RuntimeException("Unable to create request.", e);
    }
  }
```

- 异步执行请求
```
 @Override public void enqueue(final Callback<T> callback) {
    checkNotNull(callback, "callback == null");

    okhttp3.Call call;
    Throwable failure;

    synchronized (this) {
      if (executed) throw new IllegalStateException("Already executed.");
      executed = true;

      call = rawCall;
      failure = creationFailure;
      if (call == null && failure == null) {
        try {
          call = rawCall = createRawCall();
        } catch (Throwable t) {
          failure = creationFailure = t;
        }
      }
    }

    if (failure != null) {
      callback.onFailure(this, failure);
      return;
    }

    if (canceled) {
      call.cancel();
    }

    call.enqueue(new okhttp3.Callback() {
      @Override public void onResponse(okhttp3.Call call, okhttp3.Response rawResponse)
          throws IOException {
        Response<T> response;
        try {
          response = parseResponse(rawResponse);
        } catch (Throwable e) {
          callFailure(e);
          return;
        }
        callSuccess(response);
      }

      @Override public void onFailure(okhttp3.Call call, IOException e) {
        try {
          callback.onFailure(OkHttpCall.this, e);
        } catch (Throwable t) {
          t.printStackTrace();
        }
      }

      private void callFailure(Throwable e) {
        try {
          callback.onFailure(OkHttpCall.this, e);
        } catch (Throwable t) {
          t.printStackTrace();
        }
      }

      private void callSuccess(Response<T> response) {
        try {
          callback.onResponse(OkHttpCall.this, response);
        } catch (Throwable t) {
          t.printStackTrace();
        }
      }
    });
  }
```

- 同步执行请求
```
@Override public Response<T> execute() throws IOException {
    okhttp3.Call call;

    synchronized (this) {
      if (executed) throw new IllegalStateException("Already executed.");
      executed = true;

      if (creationFailure != null) {
        if (creationFailure instanceof IOException) {
          throw (IOException) creationFailure;
        } else {
          throw (RuntimeException) creationFailure;
        }
      }

      call = rawCall;
      if (call == null) {
        try {
          call = rawCall = createRawCall();
        } catch (IOException | RuntimeException e) {
          creationFailure = e;
          throw e;
        }
      }
    }

    if (canceled) {
      call.cancel();
    }

    return parseResponse(call.execute());
  }
```
- 转换请求结果
```
  Response<T> parseResponse(okhttp3.Response rawResponse) throws IOException {
    ResponseBody rawBody = rawResponse.body();

    // Remove the body's source (the only stateful object) so we can pass the response along.
    rawResponse = rawResponse.newBuilder()
        .body(new NoContentResponseBody(rawBody.contentType(), rawBody.contentLength()))
        .build();

    int code = rawResponse.code();
    if (code < 200 || code >= 300) {
      try {
        // Buffer the entire body to avoid future I/O.
        ResponseBody bufferedBody = Utils.buffer(rawBody);
        return Response.error(bufferedBody, rawResponse);
      } finally {
        rawBody.close();
      }
    }

    if (code == 204 || code == 205) {
      rawBody.close();
      return Response.success(null, rawResponse);
    }

    ExceptionCatchingRequestBody catchingBody = new ExceptionCatchingRequestBody(rawBody);
    try {
      T body = serviceMethod.toResponse(catchingBody);
      return Response.success(body, rawResponse);
    } catch (RuntimeException e) {
      // If the underlying source threw an exception, propagate that rather than indicating it was
      // a runtime exception.
      catchingBody.throwIfCaught();
      throw e;
    }
  }

```

剩下的方法就是取消网络请求，处理无返回body等等方法

