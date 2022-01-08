# BridgeInterceptor
桥梁拦截器
主要作用的是把应用层的请求转换为网络层的请求，把网络层的响应转换为应用层的响应


```
/**
   * 拦截器的核心方法
    * @param chain
   * @return
   * @throws IOException
   */
  @Override public Response intercept(Chain chain) throws IOException {
    Request userRequest = chain.request();
    Request.Builder requestBuilder = userRequest.newBuilder();
    //请求正文
    RequestBody body = userRequest.body();
    //构建请求头
    if (body != null) {
      MediaType contentType = body.contentType();
      if (contentType != null) {
        requestBuilder.header("Content-Type", contentType.toString());
      }

      long contentLength = body.contentLength();
      if (contentLength != -1) {
        requestBuilder.header("Content-Length", Long.toString(contentLength));
        requestBuilder.removeHeader("Transfer-Encoding");
      } else {
        requestBuilder.header("Transfer-Encoding", "chunked");
        requestBuilder.removeHeader("Content-Length");
      }
    }
    //添加请求url
    if (userRequest.header("Host") == null) {
      requestBuilder.header("Host", hostHeader(userRequest.url(), false));
    }
    //保持客户端与服务端的连接
    if (userRequest.header("Connection") == null) {
      requestBuilder.header("Connection", "Keep-Alive");
    }

    // If we add an "Accept-Encoding: gzip" header field we're responsible for also decompressing
    // the transfer stream.
    //gzip压缩
    boolean transparentGzip = false;
    if (userRequest.header("Accept-Encoding") == null && userRequest.header("Range") == null) {
      transparentGzip = true;
      requestBuilder.header("Accept-Encoding", "gzip");
    }
    //添加cookies
    List<Cookie> cookies = cookieJar.loadForRequest(userRequest.url());
    if (!cookies.isEmpty()) {
      requestBuilder.header("Cookie", cookieHeader(cookies));
    }
    //用户代理（用于让服务端识别操作系统，版本等等一些信息）
    if (userRequest.header("User-Agent") == null) {
      requestBuilder.header("User-Agent", Version.userAgent());
    }
    //调用下一个拦截器并返回响应
    Response networkResponse = chain.proceed(requestBuilder.build());//1
    //判断是否有cookie，如果有就保存cookie
    HttpHeaders.receiveHeaders(cookieJar, userRequest.url(), networkResponse.headers());
    //构建返回给应用层的响应
    Response.Builder responseBuilder = networkResponse.newBuilder()
        .request(userRequest);
    //如果使用了gzip则先解压缩
    if (transparentGzip
        && "gzip".equalsIgnoreCase(networkResponse.header("Content-Encoding"))
        && HttpHeaders.hasBody(networkResponse)) {
      GzipSource responseBody = new GzipSource(networkResponse.body().source());
      Headers strippedHeaders = networkResponse.headers().newBuilder()
          .removeAll("Content-Encoding")
          .removeAll("Content-Length")
          .build();
      responseBuilder.headers(strippedHeaders);
      String contentType = networkResponse.header("Content-Type");
      responseBuilder.body(new RealResponseBody(contentType, -1L, Okio.buffer(responseBody)));
    }
    //返回响应
    return responseBuilder.build();
  }
```

1. 调用下一个拦截器并返回响应，此处为核心代码。上方的代码的作用是构建http请求报文，并进行gzip压缩；下方代码的作用是把http响应报文转换为Response对象