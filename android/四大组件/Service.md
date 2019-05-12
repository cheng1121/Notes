# Service 
### 本地服务
调用者和Service在同一个进程里，运行在UI线程中，不能进行耗时操作，耗时操作超过20s后会产生ANR问题。通过在Service中创建Thread来执行耗时操作。

**任何Activity都可以控制同一Service，而系统也只会创建一个对应的Servcie的实例**

本地服务有两种启动方式：
- startService：
  1. 定义一个类继承Service
  2. manifest.xml中添加Service节点
  3. 使用startServcie(Intent)方法启动service
  4. 不使用时，调用stopService(Intent)方法停止service

生命周期：

调用 startService -> onStartCommand -> onStart(该方法已弃用)  

调用 stopService -> onDestroy 

特点：

服务启动后就和调用者没有关系了，如果不调用stopService方法，会在后台长期运行；调用者无法调用服务中的方法
    
- bindService：
  1. 定义一个类继承Service
  2. manifest.xml中添加Service节点
  3. 使用bindService(Intent,ServiceConnection,int)方法启动service
  4. 不使用时，调用unbindService(Serviceconnection)方法停止service
 
生命周期：

调用 bindService -> onBind 

调用 unbindService -> onUnbind -> onDestroy 

特点：
跟随调用者的生命周期；可以调用服务内的方法

使用方式：

```
public class MainActivity extends AppCompatActivity {

    private static final String TAG = "MainActivity";

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        Log.i(TAG, "onCreate: ");
        //startService(new Intent(MainActivity.this, MainService.class));

        bindService(new Intent(MainActivity.this, MainService.class), serviceConnection, BIND_AUTO_CREATE);
    }

    private ServiceConnection serviceConnection = new ServiceConnection() {
        @Override
        public void onServiceConnected(ComponentName name, IBinder service) {
            Log.i(TAG, "onServiceConnected: "+ name);
            MainService.MyBinder binder = (MainService.MyBinder) service;
            binder.callServiceMethod();

        }

        @Override
        public void onServiceDisconnected(ComponentName name) {
            Log.i(TAG, "onServiceDisconnected: "+ name);
        }
    };

    @Override
    protected void onStart() {
        super.onStart();
        Log.i(TAG, "onStart: ");
    }

    @Override
    protected void onResume() {
        super.onResume();
        Log.i(TAG, "onResume: ");
    }

    @Override
    protected void onRestart() {
        super.onRestart();
        Log.i(TAG, "onRestart: ");
    }

    @Override
    protected void onPause() {
        super.onPause();
        Log.i(TAG, "onPause: ");
    }

    @Override
    protected void onStop() {
        super.onStop();
        Log.i(TAG, "onStop: ");
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        Log.i(TAG, "onDestroy: ");
        //stopService(new Intent(MainActivity.this, MainService.class));
    }


}

```

```
public class MainService extends Service {
    private static final String TAG = "MainActivity";

    @Nullable
    @Override
    public IBinder onBind(Intent intent) {
        Log.d(TAG, "onBind: ");
        return new MyBinder();
    }

    @Override
    public int onStartCommand(Intent intent, int flags, int startId) {
        Log.d(TAG, "onStartCommand: ");
        return super.onStartCommand(intent, flags, startId);
    }

    @Override
    public void onStart(Intent intent, int startId) {
        super.onStart(intent, startId);
        Log.d(TAG, "onStart: ");
    }

    @Override
    public void onDestroy() {
        super.onDestroy();
        Log.d(TAG, "onDestroy: ");
    }

    public void print() {
        Log.d(TAG, "print: =================");
    }


    @Override
    public boolean onUnbind(Intent intent) {
        Log.d(TAG, "onUnbind: ");
        return super.onUnbind(intent);

    }

    public class MyBinder extends Binder {


        public void callServiceMethod() {
            print();
        }

    }
}

```

```
//清单文件中的application节点下添加
<service android:name=".MainService" />
```

### 远程服务即AIDL(这里先不了解)

### IntentService
是Service的子类,比普通的Service增加了额外的功能.

Service本身存在两个问题:
- service不回专门启动一条单独的进程，同其所在应用处于相同进程
- 不回开启一条新的线程，所以不能直接处理耗时任务

IntentService就是为了处理耗时任务：
1. 会创建独立的线程来处理所有的Intent请求
2. 会创建独立的worker线程来处理onHandleIntent方法实现的代码，无需处理多线程问题
3. 所有请求处理完成后，IntentService会自动停止，无需调用stopSelf方法停止Service
4. 为Service的onBind提供默认实现，返回null
5. 为Service的onStartCommand提供默认实现，将请求Intent添加到队列中
  