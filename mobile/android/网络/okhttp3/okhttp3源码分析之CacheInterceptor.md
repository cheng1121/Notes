# CacheInterceptor
缓存拦截器，作用是执行用户的缓存策略


```
 /**
   * 拦截器核心代码
   * @param chain
   * @return
   * @throws IOException
   */

  @Override public Response intercept(Interceptor.Chain chain) throws IOException {
    //检查内部缓存中是否有缓存，有则查看是否有当前请求的缓存，有则返回
    Response cacheCandidate = cache != null
        ? cache.get(chain.request())
        : null;
     //当前时间的时间戳
    long now = System.currentTimeMillis();
    //缓存设置，包括存放位置，有效期等
    CacheStrategy strategy = new CacheStrategy.Factory(now, chain.request(), cacheCandidate).get();
    Request networkRequest = strategy.networkRequest;
    Response cacheResponse = strategy.cacheResponse;

    if (cache != null) {
      //追踪缓存是否命中？（cache内部会判断networkRequest和cacheResponse是否为null，并记录命中次数）
      cache.trackResponse(strategy);
    }

    if (cacheCandidate != null && cacheResponse == null) {
      closeQuietly(cacheCandidate.body()); // The cache candidate wasn't applicable. Close it. 缓存候选不适用。 关闭它
    }

    // If we're forbidden from using the network and the cache is insufficient, fail.
    //如果我们被禁止使用网络并且没有缓存，则失败。
    if (networkRequest == null && cacheResponse == null) { //1
      return new Response.Builder()
          .request(chain.request())
          .protocol(Protocol.HTTP_1_1)
          .code(504)
          .message("Unsatisfiable Request (only-if-cached)")
          .body(Util.EMPTY_RESPONSE)
          .sentRequestAtMillis(-1L)
          .receivedResponseAtMillis(System.currentTimeMillis())
          .build();
    }

    // If we don't need the network, we're done.
    //如果不需要使用网络，则直接返回缓存中的响应
    if (networkRequest == null) { // 2
      return cacheResponse.newBuilder()
          .cacheResponse(stripBody(cacheResponse))
          .build();
    }
    //使用网络，直接执行下一个拦截器
    Response networkResponse = null;
    try {
      networkResponse = chain.proceed(networkRequest); //3
    } finally {
      // If we're crashing on I/O or otherwise, don't leak the cache body.
      //如果我们在I / O或其他方面崩溃，请不要泄漏缓存主体。
      if (networkResponse == null && cacheCandidate != null) {
        closeQuietly(cacheCandidate.body());
      }
    }

    // If we have a cache response too, then we're doing a conditional get.
    //如果我们也有缓存响应，那么我们正在进行条件获取。
    if (cacheResponse != null) {  //4
      if (networkResponse.code() == HTTP_NOT_MODIFIED) { //判断响应码，如果已修改则更新缓存
        Response response = cacheResponse.newBuilder() 
            .headers(combine(cacheResponse.headers(), networkResponse.headers()))
            .sentRequestAtMillis(networkResponse.sentRequestAtMillis())
            .receivedResponseAtMillis(networkResponse.receivedResponseAtMillis())
            .cacheResponse(stripBody(cacheResponse))
            .networkResponse(stripBody(networkResponse))
            .build();
        networkResponse.body().close();

        // Update the cache after combining headers but before stripping the
        // Content-Encoding header (as performed by initContentStream()).
        //在合并headers之后剥离之前更新缓存
        cache.trackConditionalCacheHit();
        cache.update(cacheResponse, response);
        return response;
      } else {
        closeQuietly(cacheResponse.body());
      }
    }
    //cacheResponse为null时
    Response response = networkResponse.newBuilder() //6
        .cacheResponse(stripBody(cacheResponse))
        .networkResponse(stripBody(networkResponse))
        .build();

    if (cache != null) { 
      //判断是是否有body      &&  请求和响应是否可以被缓存
      if (HttpHeaders.hasBody(response) && CacheStrategy.isCacheable(response, networkRequest)) {
        // Offer this request to the cache.
        //将此请求提供给缓存。
        CacheRequest cacheRequest = cache.put(response);
        //把响应写入缓存并返回响应 使用okio
        return cacheWritingResponse(cacheRequest, response);
      }
      //验证请求方式
      if (HttpMethod.invalidatesCache(networkRequest.method())) {
        try {
          //移除该请求的所有缓存
          cache.remove(networkRequest);
        } catch (IOException ignored) {
          // The cache cannot be written.
        }
      }
    }

    return response;
  }
```

整体逻辑是检查是否有缓存，如果有则根据缓存策略执行：
1. 没有网络并且不使用缓存，则直接返回错误
2. 没有网络且使用缓存，则如果有缓存则返回缓存
3. 有网络时，先执行下一个拦截器（责任链模式的链式调用）,获取响应
4. 得到响应后，判断是否有对应的缓存，并且判断数据是否改变，已改变则更新缓存
5. 没有缓存，则把请求对应的数据存入缓存