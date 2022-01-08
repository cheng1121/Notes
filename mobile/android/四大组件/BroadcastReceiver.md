# 广播(BroadcastReceiver)
分为两种：有序广播和普通广播

### 普通广播
完全异步的，可以在同一时刻被所有接收者接收到，消息传递的效率比较高，缺点是不能将广播处理结果传递给下一个接收者，并且无法终止广播Intent的传播

### 有序广播
按照接收者声明的优先级别，被接受者依次接收广播

### 广播的生命周期
如果一个广播处理完onReceive，那么系统将认定此对象将不再是一个活动的对象，也就会finished掉它。所以其生命周期很短，大约10s，所以不应该在其内部创建线程执行耗时操作，因为有可能线程内部任务未执行完毕，广播就已经被销毁了

### 示例
##### 静态注册

```
//自定义广播 继承 BroadcastReceiver类
public class MainReceiver extends BroadcastReceiver {


    private static final String TAG = "MainActivity";

    @Override
    public void onReceive(Context context, Intent intent) {
        Log.i(TAG, "onReceive: ");
        //接收广播
    }
}

```

```
清单文件中添加 receiver 并配置 intent filter
<!--静态注册广播-->
        <receiver android:name=".MainReceiver"

            >
            <intent-filter>
                <action android:name="com.cheng.chapter.receiver"/>

            </intent-filter>

        </receiver>


```

```
//发送广播
  Intent intent = new Intent("com.cheng.chapter.receiver");

  intent.setPackage(getPackageName());
  //发送广播
  sendBroadcast(intent);

```
注：Android8.0以后删除了静态注册的方式，需要在发送广播的时候添加包名，也就是这句代码  intent.setPackage(getPackageName()); 静态注册的广播才能接收到；并且官方推荐使用动态注册广播

### 动态注册广播
```
//自定义广播 继承 BroadcastReceiver类
public class MainReceiver extends BroadcastReceiver {


    private static final String TAG = "MainActivity";

    @Override
    public void onReceive(Context context, Intent intent) {
        Log.i(TAG, "onReceive: ");
        //接收广播
    }
}

```

```
//动态 注册广播
IntentFilter intentFilter = new IntentFilter();
intentFilter.addAction("com.cheng.chapter.receiver");
```
```
//发送广播
  Intent intent = new Intent("com.cheng.chapter.receiver");
  //发送广播
  sendBroadcast(intent);
  
//发送有序广播  可以添加权限也可以传 null代表不需要权限
sendOrderedBroadcast(intent, permission);
```

```
 //取消注册广播
 unregisterReceiver(new MainReceiver());
```

### 动态和静态注册的区别
- 动态注册广播不是常驻型广播，也就是说广播跟随Activity的生命周期，注意:在activity结束前，移除广播接收器
- 静态注册广播是常驻型广播，也就是说当应用程序关闭后，如果有消息广播，程序也会被系统调用自动运行
- 当广播为有序广播时：
  1. 优先级高的先接收
  2. 同优先级的广播接收器，动态优先于静态
  3. 同优先级的同类广播接收器，静态是先扫描的优先于后扫描的；动态是先注册的优先于后注册的
- 当广播为普通广播时：
  1. 无视优先级，动态优先于静态
  2. 同优先级的同类广播，，静态是先扫描的优先于后扫描的；动态是先注册的优先于后注册的



