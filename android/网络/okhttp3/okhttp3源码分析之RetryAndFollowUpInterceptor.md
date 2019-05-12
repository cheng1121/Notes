# RetryAndFollowUpInterceptor
该拦截器负责失败重试和重定向
核心方法为：
```
 /**
   * 核心方法
   * @param chain  RealInterceptorChain
   * @return response 响应
   * @throws IOException io异常
   */
  @Override public Response intercept(Interceptor.Chain chain) throws IOException {
    //拿到请求
    Request request = chain.request();
    //拦截器链
    RealInterceptorChain realChain = (RealInterceptorChain) chain;
    //发射器
    Transmitter transmitter = realChain.transmitter();
    //计数
    int followUpCount = 0;
    //响应
    Response priorResponse = null;
    while (true) { //无限循环，直到调用return  或者抛出异常
      //准备连接服务端 
      transmitter.prepareToConnect(request); //1
      //判断连接是否被取消
      if (transmitter.isCanceled()) {
        throw new IOException("Canceled");
      }
      //响应
      Response response;
      //是否成功
      boolean success = false;
      try {
        //调用下一个拦截器  并返回响应（责任链模式）
        response = realChain.proceed(request, transmitter, null); //2
        success = true;
      } catch (RouteException e) {
        // The attempt to connect via a route failed. The request will not have been sent.
        //尝试通过路由连接失败。 请求将不会被发送。
        if (!recover(e.getLastConnectException(), transmitter, false, request)) {
          throw e.getFirstConnectException();
        }
        continue;
      } catch (IOException e) {
        // An attempt to communicate with a server failed. The request may have been sent.
        //尝试与服务器通信失败。 请求可能已发送。
        boolean requestSendStarted = !(e instanceof ConnectionShutdownException);
        if (!recover(e, transmitter, requestSendStarted, request)) throw e;
        continue;
      } finally {
        // The network call threw an exception. Release any resources.
        //网络调用发生异常，释放一些资源
        if (!success) {
          transmitter.exchangeDoneDueToException();
        }
      }

      // Attach the prior response if it exists. Such responses never have a body.
      //如果priorResponse不为空，则重新构造和一个body为null的响应，并给当前response赋值
      if (priorResponse != null) {
        response = response.newBuilder()
            .priorResponse(priorResponse.newBuilder()
                    .body(null)
                    .build())
            .build();
      }
      //获取要传输的数据
      Exchange exchange = Internal.instance.exchange(response);
      //获取传输路径
      Route route = exchange != null ? exchange.connection().route() : null;
      //后续操作 根据返回的状态码进行处理（重定向和重试）
      Request followUp = followUpRequest(response, route); //3
      //followUp为null 则返回响应response
      if (followUp == null) {
        if (exchange != null && exchange.isDuplex()) {
          transmitter.timeoutEarlyExit();
        }
        return response;
      }
      //followUp不为null 则判断body是否为null
      RequestBody followUpBody = followUp.body();
      if (followUpBody != null && followUpBody.isOneShot()) {
        return response;
      }

      closeQuietly(response.body());
      if (transmitter.hasExchange()) {
        exchange.detachWithViolence();
      }
      //判断请求次数 大于20则抛出异常跳出循环
      if (++followUpCount > MAX_FOLLOW_UPS) {  //4
        throw new ProtocolException("Too many follow-up requests: " + followUpCount);
      }
      //赋值，继续循环
      request = followUp;
      priorResponse = response;
    }
  }
```
主要做以下几件事：
1. 准备连接服务端
2. 调用下一个拦截器（实现责任链模式的核心代码）
3. 对响应response进行后续处理。主要是通过响应码判断是进行重定向还是重试
4. 对followUpCount加1，并判断是否要跳出循环并抛出异常

下方代码为第三步中，对响应码进行的判断
```
  * Figures out the HTTP request to make in response to receiving {@code userResponse}. This will
   * either add authentication headers, follow redirects or handle a client request timeout. If a
   * follow-up is either unnecessary or not applicable, this returns null.
   * 计算出响应接收{@code userResponse}的HTTP请求。 这将添加身份验证标头，遵循重定向或处理客户端请求超时。 如果后续操作不必要或不适用，则返回null。
   */
  private Request followUpRequest(Response userResponse, @Nullable Route route) throws IOException {
    if (userResponse == null) throw new IllegalStateException();
    //得到响应码
    int responseCode = userResponse.code();
    //得到请求方式
    final String method = userResponse.request().method();
    switch (responseCode) {
       //代理验证
      case HTTP_PROXY_AUTH:
        Proxy selectedProxy = route != null
            ? route.proxy()
            : client.proxy();
        if (selectedProxy.type() != Proxy.Type.HTTP) {
          throw new ProtocolException("Received HTTP_PROXY_AUTH (407) code while not using proxy");
        }
        return client.proxyAuthenticator().authenticate(route, userResponse);
      //无需验证
      case HTTP_UNAUTHORIZED:
        return client.authenticator().authenticate(route, userResponse);
      //重定向
      case HTTP_PERM_REDIRECT:
      case HTTP_TEMP_REDIRECT:
        // "If the 307 or 308 status code is received in response to a request other than GET
        // or HEAD, the user agent MUST NOT automatically redirect the request"
        //“如果收到307或308状态代码以响应GET或HEAD以外的请求，则用户代理不得自动重定向请求”
        if (!method.equals("GET") && !method.equals("HEAD")) {
          return null;
        }
        // fall-through
      case HTTP_MULT_CHOICE:
      case HTTP_MOVED_PERM:
      case HTTP_MOVED_TEMP:
      case HTTP_SEE_OTHER:
        // Does the client allow redirects?
        //客户端是否允许重定向？
        if (!client.followRedirects()) return null;

        String location = userResponse.header("Location");
        if (location == null) return null;
        HttpUrl url = userResponse.request().url().resolve(location);

        // Don't follow redirects to unsupported protocols.
        //不要遵循不支持的协议的重定向。
        if (url == null) return null;

        // If configured, don't follow redirects between SSL and non-SSL.
        //如果已配置，请不要遵循SSL和非SSL之间的重定向。
        boolean sameScheme = url.scheme().equals(userResponse.request().url().scheme());
        if (!sameScheme && !client.followSslRedirects()) return null;

        // Most redirects don't include a request body.
        //大多数重定向不包括请求正文
        Request.Builder requestBuilder = userResponse.request().newBuilder();
        if (HttpMethod.permitsRequestBody(method)) {
          //判断是否包括请求正文
          final boolean maintainBody = HttpMethod.redirectsWithBody(method);
          //PROPFIND方式的请求会重定向到GET请求
          if (HttpMethod.redirectsToGet(method)) {
            requestBuilder.method("GET", null);
          } else {
            //添加body
            RequestBody requestBody = maintainBody ? userResponse.request().body() : null;
            requestBuilder.method(method, requestBody);
          }
          if (!maintainBody) {
            //移除请求头
            requestBuilder.removeHeader("Transfer-Encoding");
            requestBuilder.removeHeader("Content-Length");
            requestBuilder.removeHeader("Content-Type");
          }
        }

        // When redirecting across hosts, drop all authentication headers. This
        // is potentially annoying to the application layer since they have no
        // way to retain them.
        //在主机间重定向时，请删除所有身份验证标头。 这对应用程序层来说可能很烦人，因为它们无法保留它们。
        if (!sameConnection(userResponse.request().url(), url)) {
          requestBuilder.removeHeader("Authorization");
        }

        return requestBuilder.url(url).build();
      //连接超时
      case HTTP_CLIENT_TIMEOUT:
        // 408's are rare in practice, but some servers like HAProxy use this response code. The
        // spec says that we may repeat the request without modifications. Modern browsers also
        // repeat the request (even non-idempotent ones.)
        if (!client.retryOnConnectionFailure()) {
          // The application layer has directed us not to retry the request.
          return null;
        }

        RequestBody requestBody = userResponse.request().body();
        if (requestBody != null && requestBody.isOneShot()) {
          return null;
        }

        if (userResponse.priorResponse() != null
            && userResponse.priorResponse().code() == HTTP_CLIENT_TIMEOUT) {
          // We attempted to retry and got another timeout. Give up.
          return null;
        }

        if (retryAfter(userResponse, 0) > 0) {
          return null;
        }

        return userResponse.request();
      //Service Unavailable.  暂停服务
      case HTTP_UNAVAILABLE:
        if (userResponse.priorResponse() != null
            && userResponse.priorResponse().code() == HTTP_UNAVAILABLE) {
          // We attempted to retry and got another timeout. Give up.
          //重试后还是超时，则放弃
          return null;
        }

        if (retryAfter(userResponse, Integer.MAX_VALUE) == 0) {
          // specifically received an instruction to retry without delay
          return userResponse.request();
        }

        return null;

      default:
        return null;
    }
  }
```