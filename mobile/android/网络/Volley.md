# Volley
2013年google I/O大会上推出的一款网络通信框架,既可以访问网络取得数据，也可以加载图片，并且在性能方面也进行了大幅度的调整。

设计目标是进行数据量不大但通信频繁的网络操作。而对于大数据量的网络操作，比如说下载文件等，Volley的表现确非常糟糕

## Volley基本用法
包括StringRequest、JsonRequest、ImageRequest、ImageLoader、NetworkImageView等
其中StringRequest、JsonRequest、ImageRequest的使用非常类似如下所示：

```

    //创建队列
    RequestQueue queue = Volley.newRequestQueue(getApplicationContext());
    //发起请求
    StringRequest request = new StringRequest(
            //请求方法
            Request.Method.GET
            //请求url
            , "http://www.baidu.com"
            //响应监听
            , new Response.Listener<String>() {
        @Override
        public void onResponse(String response) {
            Log.i(TAG, "onResponse:============"+response);
        }
    }
            //错误监听
            , new Response.ErrorListener() {
        @Override
        public void onErrorResponse(VolleyError error) {
            Log.i(TAG, "onErrorResponse: ========="+ error.getMessage());
        }
    });

    //添加到队列中
    queue.add(request);
```

### ImageLoader的使用
```
 //请求队列
        RequestQueue queue = Volley.newRequestQueue(getApplicationContext());
        //获取ImageLoader实例，并设置缓存
        ImageLoader imageLoader = new ImageLoader(queue, new ImageLoader.ImageCache() {
            @Override
            public Bitmap getBitmap(String url) {
                return null;
            }

            @Override
            public void putBitmap(String url, Bitmap bitmap) {

            }
        });
        //设置图片加载监听,并设置View，默认图片等
        ImageLoader.ImageListener listener = ImageLoader.getImageListener(ImageView,R.mipmap.ic_launcher,R.mipmap.ic_launcher);
        imageLoader.get("http://img_url",listener);//设置url和监听
     }


```

设置最大宽度和最大高度的方法：
```
  /**
     * Equivalent to calling {@link #get(String, ImageListener, int, int, ScaleType)} with {@code
     * Scaletype == ScaleType.CENTER_INSIDE}.
     */
    public ImageContainer get(
            String requestUrl, ImageListener imageListener, int maxWidth, int maxHeight) {
        return get(requestUrl, imageListener, maxWidth, maxHeight, ScaleType.CENTER_INSIDE);
    }
```

### 使用NetworkImageView
xml中：
```
 <com.android.volley.toolbox.NetworkImageView
        android:id="@+id/loadimg"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Hello World!"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintRight_toRightOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

```


代码中：
```

        NetworkImageView load = findViewById(R.id.loadimg);
        
        RequestQueue queue = Volley.newRequestQueue(getApplicationContext());
        ImageLoader imageLoader = new ImageLoader(queue, new ImageLoader.ImageCache() {
            @Override
            public Bitmap getBitmap(String url) {
                return null;
            }

            @Override
            public void putBitmap(String url, Bitmap bitmap) {

            }
        });
        load.setDefaultImageResId(R.mipmap.ic_launcher);
        load.setErrorImageResId(R.mipmap.ic_launcher);
        load.setImageUrl("image url",imageLoader);
        

```

## 源码分析
