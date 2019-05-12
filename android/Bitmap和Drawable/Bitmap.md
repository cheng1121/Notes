# Bitmap

### Bitmap的使用

```
 String path = Environment.getExternalStorageDirectory().getAbsolutePath() + "/crop.jpg";
//加载本地图片，需要存储权限
Bitmap bitmap = BitmapFactory.decodeFile(path);

//加载资源文件（res目录下的图片）
Bitmap resourceBitmap = BitmapFactory.decodeResource(getResources(), R.mipmap.ic_launcher);

```

### Bitmap占用内存大小

**bitmap内存占用 约等于 像素总数据大小 = 横向像素数量 * 纵向像素数量 * 每个像素的字节大小**


##### 单个像素的字节大小

Config | 占用字节大小(byte) | 说明
---|---|---
ALPHA_8 | 1 | 单透明通道
RGB_565 | 2 | 简易RGB色调
ARGB_4444 | 4 | 已废弃
ARGB_8888 | 4 | 24位真彩
RGBA_F16 | 8 | 更丰富的色彩表现HDR（8.0新增）
HARDWARE | Special | Bitmap直接存储在graphic memory(8.0新增)

```
    图片原始大小：1190 * 1190  252kB


    int size = bitmap.getRowBytes() * bitmap.getHeight();
    Log.i(TAG, "decodeImage: 占用内存大小：" + size);
    Log.i(TAG, "decodeImage: 宽度："+ bitmap.getWidth());
    Log.i(TAG, "decodeImage: 高度：" + bitmap.getHeight());
    Log.i(TAG, "decodeImage: config: "+ bitmap.getConfig());
    Log.i(TAG, "decodeImage: density "+ getResources().getDisplayMetrics().density);
    
    结果：
    decodeImage: 占用内存大小：5664400
    decodeImage: 宽度：1190
    decodeImage: 高度：1190
    decodeImage: config: ARGB_8888
    decodeImage: densityDPI 640
```

从以上可以看出来，占用内存 5664400 = 1190 * 1190 * 4(ARGB_8888 占用4字节)


### Bitmap压缩

##### 采样率压缩
```
 String path = Environment.getExternalStorageDirectory().getAbsolutePath() + "/crop.jpg";

        BitmapFactory.Options options = new BitmapFactory.Options();
        options.inPreferredConfig = Bitmap.Config.RGB_565; //设置图片格式565
        options.inJustDecodeBounds = true;  //设置只获取边框大小
        BitmapFactory.decodeFile(path,options); //读取图片边框
        options.inJustDecodeBounds= false; //false为读取整个图片
        options.inSampleSize = 5; //设置采样率为原来的 1/5   也就是宽高除以5
        Bitmap bitmap = BitmapFactory.decodeFile(path,options);

       // Bitmap resourceBitmap = BitmapFactory.decodeResource(getResources(), R.mipmap.ic_launcher);


        int size = bitmap.getRowBytes() * bitmap.getHeight();
        Log.i(TAG, "decodeImage: 占用内存大小：" + size);
        Log.i(TAG, "decodeImage: 宽度："+ bitmap.getWidth());
        Log.i(TAG, "decodeImage: 高度：" + bitmap.getHeight());
        Log.i(TAG, "decodeImage: config: "+ bitmap.getConfig());
        Log.i(TAG, "decodeImage: density "+ getResources().getDisplayMetrics().density);

        imageView.setImageBitmap(bitmap);
        
        运行结果：
        decodeImage: 占用内存大小：113288
        decodeImage: 宽度：238
        decodeImage: 高度：238
        decodeImage: config: RGB_565
        
```

##### 质量压缩
质量压缩  不改变图片的宽高和像素 所以占用内存是不变的

质量压缩是通过改变图片的位深及透明度等来达到压缩的目的，改变的是保存后的存储体积

```
  Bitmap bitmap = BitmapFactory.decodeFile(path);

    ByteArrayOutputStream baos = new ByteArrayOutputStream();
    //图片大小
    //压缩比率为原来的80%  并把byte数据存入 baos中
    bitmap.compress(Bitmap.CompressFormat.JPEG, 80, baos);
    byte[] bitmapBytes = baos.toByteArray();
    //从baos中解析出压缩后的图片
    bitmap = BitmapFactory.decodeByteArray(bitmapBytes, 0, bitmapBytes.length);

    return bitmap;
```

通常在使用的时候会混合使用 质量压缩和采样率压缩


