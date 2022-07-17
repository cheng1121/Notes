# Handler原理
![Handler运行流程图](https://upload-images.jianshu.io/upload_images/11142016-11ebd236e49b8aed.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### ThreadLocal的工作原理
ThreadLocal是一个线程内部的数据存储类，通过它可以在指定线程中存储数据，只有指定的线程可以读取存储的数据，其他线程无法读取

##### ThreadLocal的使用
```
public class AActivity extends AppCompatActivity {
 private static ThreadLocal<Boolean> threadLocal = new ThreadLocal<>();


    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_a);
    
        threadLocal.set(true);

        new Thread("Thread1"){
          @Override
          public void run() {
              super.run();

              threadLocal.set(false);
              Log.i(TAG, getName()+" run: threadLocal value " + threadLocal.get());

          }
      }.start();


        new Thread("Thread2"){
          @Override
          public void run() {
              super.run();
              Log.i(TAG, getName()+" run: threadLocal value "  + threadLocal.get());
          }
      }.start();

        Log.i(TAG, Thread.currentThread().getName()+" onCreate: threadLocal value " + threadLocal.get());
    }
}
```
运行结果：![image.png](https://upload-images.jianshu.io/upload_images/11142016-c559bc065d1249a2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##### ThreadLocalMap
ThreadLocalMap是ThreadLocal中用来存储数据的静态内部类
其中的数据结构为Entry数组,Entry是一个弱引用类
数组的长度默认为16，空间不足时会自动扩容，每次扩容为当前数组长度 * 2 
通过 int i = 当前ThreadLocal的HashCode & (数组长度 - 1)计算每个元素存储的位置
```
static class Entry extends WeakReference<ThreadLocal<?>> {
            /** The value associated with this ThreadLocal. */
            Object value;

            Entry(ThreadLocal<?> k, Object v) {
                super(k);
                value = v;
            }
        }
```



##### ThreadLocal的set方法
```
public void set(T value) {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null)
            map.set(this, value);
        else
            createMap(t, value);
    }

 void createMap(Thread t, T firstValue) {
        t.threadLocals = new ThreadLocalMap(this, firstValue);
    }
```
拿到当前线程中的ThreadLocalMap对象，如果map对象不为空，则把数据存入map中；否则就创建一个map对象存入线程中

##### ThreadLocal的get方法
```
    public T get() {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null) {
            ThreadLocalMap.Entry e = map.getEntry(this);
            if (e != null) {
                @SuppressWarnings("unchecked")
                T result = (T)e.value;
                return result;
            }
        }
        return setInitialValue();
    }
  //该方法为 ThreadLocalMap中的获取Entry数组中某一元素的方法
  private Entry getEntry(ThreadLocal<?> key) {
           //通过当前线程中的ThreadLocal的hash code和数组长度来获取 数据在数组中的索引
            int i = key.threadLocalHashCode & (table.length - 1);
            Entry e = table[i];
            if (e != null && e.get() == key)
                return e;
            else
                return getEntryAfterMiss(key, i, e);
        }
//返回默认值，并且创建ThreadLocalMap对象
private T setInitialValue() {
        T value = initialValue();
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null)
            map.set(this, value);
        else
            createMap(t, value);
        return value;
    }
//默认值就是null
 protected T initialValue() {
        return null;
    }
```
获取当前线程中的ThreadLocalMap 对象，获取map中和当前线程的数据；如果数据为null的话，则返回默认值

### MessageQueue的工作原理
主要包含两个操作插入(enqueueMessage())和读取（next()），其中读取操作伴随着删除操作

##### 插入(enqueueMessage)方法
```
  boolean enqueueMessage(Message msg, long when) {
        if (msg.target == null) {
            throw new IllegalArgumentException("Message must have a target.");
        }
        if (msg.isInUse()) {
            throw new IllegalStateException(msg + " This message is already in use.");
        }

        synchronized (this) {
           //根据退出标记判断是否可以退出
            if (mQuitting) {
                IllegalStateException e = new IllegalStateException(
                        msg.target + " sending message to a Handler on a dead thread");
                Log.w(TAG, e.getMessage(), e);
                msg.recycle();
                return false;
            }
             //设置当前消息已在使用(排队)
            msg.markInUse();
            //当前消息的传递时间 毫秒值
            msg.when = when;
            Message p = mMessages;
            boolean needWake;
            if (p == null || when == 0 || when < p.when) {
                // 被阻止就唤醒事件队列
                msg.next = p;
                mMessages = msg;
                needWake = mBlocked;
            } else {
             // 插入队列中间。 通常我们不必唤醒事件队列，
             //除非队列头部有障碍，并且消息是队列中最早的异步消息。
                needWake = mBlocked && p.target == null && msg.isAsynchronous();
                Message prev;
                for (;;) {
                    prev = p;
                    p = p.next;
                    if (p == null || when < p.when) {
                        break;
                    }
                    if (needWake && p.isAsynchronous()) {
                        needWake = false;
                    }
                }
                msg.next = p; // invariant: p == prev.next
                prev.next = msg;
            }

            // We can assume mPtr != 0 because mQuitting is false.
            if (needWake) {
                nativeWake(mPtr);
            }
        }
        return true;
    }
```

##### 读取(next)的方法
```
  Message next() {
        // 如果loop退出则不再读取消息
        final long ptr = mPtr;
        if (ptr == 0) {
            return null;
        }

        int pendingIdleHandlerCount = -1; // -1 only during first iteration
        int nextPollTimeoutMillis = 0;
        for (;;) {
            if (nextPollTimeoutMillis != 0) {
                Binder.flushPendingCommands();
            }
             //根据nextPollTimeoutMillis判断是否 阻塞
            nativePollOnce(ptr, nextPollTimeoutMillis);

            synchronized (this) {
                // 尝试查找下一条消息，如果找到就返回
                final long now = SystemClock.uptimeMillis();
                Message prevMsg = null;
                Message msg = mMessages;
                if (msg != null && msg.target == null) {
                    // 如果没有消息发送过来，则阻塞，以等待消息
                    do {
                        prevMsg = msg;
                        msg = msg.next;
                    } while (msg != null && !msg.isAsynchronous());
                }
                if (msg != null) {
                    if (now < msg.when) {
                        // 当前消息未准备就绪，设置唤醒超时时间以等待准备好后唤醒
                        nextPollTimeoutMillis = (int) Math.min(msg.when - now, Integer.MAX_VALUE);
                    } else {
                        // 获取一个消息
                        mBlocked = false;
                        if (prevMsg != null) {
                            prevMsg.next = msg.next;
                        } else {
                            mMessages = msg.next;
                        }
                        msg.next = null;
                        if (DEBUG) Log.v(TAG, "Returning message: " + msg);
                        msg.markInUse();
                        return msg;
                    }
                } else {
                    // 没有消息了
                    nextPollTimeoutMillis = -1;
                }

                // 退出
                if (mQuitting) {
                    dispose();
                    return null;
                }
             。。。
            }
           。。。
            }
         。。。
        }
    }
```
整体逻辑就是循环读取下一条消息并返回，如果没有新消息则阻塞，等待新消息。


### Looper的工作原理
1. 通过Looper.prepare方法来创建一个Looper
2. 通过Looper.loop方法来循环读取消息

##### prepare方法
```
 public static void prepare() {
        prepare(true);
    }

  private static void prepare(boolean quitAllowed) {
        if (sThreadLocal.get() != null) {
            throw new RuntimeException("Only one Looper may be created per thread");
        }
        sThreadLocal.set(new Looper(quitAllowed));
    }

  private Looper(boolean quitAllowed) {
        mQueue = new MessageQueue(quitAllowed);
        mThread = Thread.currentThread();
    }
   //UI线程中创建Looper的方法
  public static void prepareMainLooper() {
        prepare(false);
        synchronized (Looper.class) {
            if (sMainLooper != null) {
                throw new IllegalStateException("The main Looper has already been prepared.");
            }
            sMainLooper = myLooper();
        }
    }
```
1. prepare内部会调用prepare(boolean quitAllowed)方法
2. prepare(boolean quitAllowed)中会创建一个Looper实例并存储到ThreadLocal中
3. Looper的构造方法中会创建一个MessageQueue和获取Looper所在线程
4. 创建UI线程中的Looper是调用prepareMainLooper方法，其内部也是调用prepare方法来实现的

##### loop方法
```
 public static void loop() {
       //获取Looper
        final Looper me = myLooper();
        if (me == null) {
            throw new RuntimeException("No Looper; Looper.prepare() wasn't called on this thread.");
        }
        final MessageQueue queue = me.mQueue;

        //确保此线程的标识是本地进程的标识，并跟踪该标识标记的实际内容。
        Binder.clearCallingIdentity();
        final long ident = Binder.clearCallingIdentity();

        。。。

        boolean slowDeliveryDetected = false;
       //循环读取消息
        for (;;) {
            Message msg = queue.next(); //可能阻塞
            if (msg == null) {
                // 没有消息
                return;
            }
           。。。
     
            final long dispatchStart = needStartTime ? SystemClock.uptimeMillis() : 0;
            final long dispatchEnd;
            try {
               //发送消息到Handler的dispatchMessage方法中
                msg.target.dispatchMessage(msg);
                dispatchEnd = needEndTime ? SystemClock.uptimeMillis() : 0;
            } finally {
                if (traceTag != 0) {
                    Trace.traceEnd(traceTag);
                }
            }
            。。。
         
            msg.recycleUnchecked();
        }
    }
```
1. 获取looper
2. 调用messageQueue的next方法循环读取消息，当没有消息时 next方法被阻塞，loop方法也会被阻塞
3. 得到消息后，调用Handler的dispatchMessage方法把消息传递给Handler

### UI线程中的消息循环
1.在ActivityThread的main方法中调用Looper. prepareMainLooper来创建Looper和MessageQueue,调用Looper.loop方法来开启循环
2. 通过ActivityThread.H发送消息到messageQueue中，ActivityThread.H继承了Handler

##### Looper 为什么不会阻塞UI线程

```
public static void main(String[] args) {
        。。。
        //创建Looper 和MessageQueue
        Looper.prepareMainLooper();
        。。。
       //创建ActivityThread
        ActivityThread thread = new ActivityThread();
       //创建Binder通道（Binder线程）
        thread.attach(false, startSeq);
        //获取ActivityThread中的Hanlder
        if (sMainThreadHandler == null) {
            sMainThreadHandler = thread.getHandler();
        }
        。。。
        //开启循环
        Looper.loop();
        throw new RuntimeException("Main thread loop unexpectedly exited");
    }
```
1. 为了保证UI线程一直运行下去，需要一个无限循环，而Looper就是这个无限循环
2. 创建了Binder线程，在Binder线程中通过Handler发送消息到UI线程取执行具体的操作。loop 方法中的queue.next()中调用了nativePollOnce(ptr, nextPollTimeoutMillis)方法，当没有消息时next会阻塞在nativePollOnce方法中，其实现原理涉及了linux中的pipe/epoll机制(这里不过多了解)。阻塞后，会释放资源，UI线程处于休眠状态，所以并不会占用大量cpu资源
3. Activity的生命周期方法的执行都是通过Handler发送消息到UI线程中执行的

最后贴一下UI线程Handler需要执行的操作
```
 class H extends Handler {
        public static final int BIND_APPLICATION        = 110;
        public static final int EXIT_APPLICATION        = 111;
        public static final int RECEIVER                = 113;
        public static final int CREATE_SERVICE          = 114;
        public static final int SERVICE_ARGS            = 115;
        public static final int STOP_SERVICE            = 116;

        public static final int CONFIGURATION_CHANGED   = 118;
        public static final int CLEAN_UP_CONTEXT        = 119;
        public static final int GC_WHEN_IDLE            = 120;
        public static final int BIND_SERVICE            = 121;
        public static final int UNBIND_SERVICE          = 122;
        public static final int DUMP_SERVICE            = 123;
        public static final int LOW_MEMORY              = 124;
        public static final int PROFILER_CONTROL        = 127;
        public static final int CREATE_BACKUP_AGENT     = 128;
        public static final int DESTROY_BACKUP_AGENT    = 129;
        public static final int SUICIDE                 = 130;
        public static final int REMOVE_PROVIDER         = 131;
        public static final int ENABLE_JIT              = 132;
        public static final int DISPATCH_PACKAGE_BROADCAST = 133;
        public static final int SCHEDULE_CRASH          = 134;
        public static final int DUMP_HEAP               = 135;
        public static final int DUMP_ACTIVITY           = 136;
        public static final int SLEEPING                = 137;
        public static final int SET_CORE_SETTINGS       = 138;
        public static final int UPDATE_PACKAGE_COMPATIBILITY_INFO = 139;
        public static final int DUMP_PROVIDER           = 141;
        public static final int UNSTABLE_PROVIDER_DIED  = 142;
        public static final int REQUEST_ASSIST_CONTEXT_EXTRAS = 143;
        public static final int TRANSLUCENT_CONVERSION_COMPLETE = 144;
        public static final int INSTALL_PROVIDER        = 145;
        public static final int ON_NEW_ACTIVITY_OPTIONS = 146;
        public static final int ENTER_ANIMATION_COMPLETE = 149;
        public static final int START_BINDER_TRACKING = 150;
        public static final int STOP_BINDER_TRACKING_AND_DUMP = 151;
        public static final int LOCAL_VOICE_INTERACTION_STARTED = 154;
        public static final int ATTACH_AGENT = 155;
        public static final int APPLICATION_INFO_CHANGED = 156;
        public static final int RUN_ISOLATED_ENTRY_POINT = 158;
        public static final int EXECUTE_TRANSACTION = 159;
        public static final int RELAUNCH_ACTIVITY = 160;

        String codeToString(int code) {
            if (DEBUG_MESSAGES) {
                switch (code) {
                    case BIND_APPLICATION: return "BIND_APPLICATION";
                    case EXIT_APPLICATION: return "EXIT_APPLICATION";
                    case RECEIVER: return "RECEIVER";
                    case CREATE_SERVICE: return "CREATE_SERVICE";
                    case SERVICE_ARGS: return "SERVICE_ARGS";
                    case STOP_SERVICE: return "STOP_SERVICE";
                    case CONFIGURATION_CHANGED: return "CONFIGURATION_CHANGED";
                    case CLEAN_UP_CONTEXT: return "CLEAN_UP_CONTEXT";
                    case GC_WHEN_IDLE: return "GC_WHEN_IDLE";
                    case BIND_SERVICE: return "BIND_SERVICE";
                    case UNBIND_SERVICE: return "UNBIND_SERVICE";
                    case DUMP_SERVICE: return "DUMP_SERVICE";
                    case LOW_MEMORY: return "LOW_MEMORY";
                    case PROFILER_CONTROL: return "PROFILER_CONTROL";
                    case CREATE_BACKUP_AGENT: return "CREATE_BACKUP_AGENT";
                    case DESTROY_BACKUP_AGENT: return "DESTROY_BACKUP_AGENT";
                    case SUICIDE: return "SUICIDE";
                    case REMOVE_PROVIDER: return "REMOVE_PROVIDER";
                    case ENABLE_JIT: return "ENABLE_JIT";
                    case DISPATCH_PACKAGE_BROADCAST: return "DISPATCH_PACKAGE_BROADCAST";
                    case SCHEDULE_CRASH: return "SCHEDULE_CRASH";
                    case DUMP_HEAP: return "DUMP_HEAP";
                    case DUMP_ACTIVITY: return "DUMP_ACTIVITY";
                    case SLEEPING: return "SLEEPING";
                    case SET_CORE_SETTINGS: return "SET_CORE_SETTINGS";
                    case UPDATE_PACKAGE_COMPATIBILITY_INFO: return "UPDATE_PACKAGE_COMPATIBILITY_INFO";
                    case DUMP_PROVIDER: return "DUMP_PROVIDER";
                    case UNSTABLE_PROVIDER_DIED: return "UNSTABLE_PROVIDER_DIED";
                    case REQUEST_ASSIST_CONTEXT_EXTRAS: return "REQUEST_ASSIST_CONTEXT_EXTRAS";
                    case TRANSLUCENT_CONVERSION_COMPLETE: return "TRANSLUCENT_CONVERSION_COMPLETE";
                    case INSTALL_PROVIDER: return "INSTALL_PROVIDER";
                    case ON_NEW_ACTIVITY_OPTIONS: return "ON_NEW_ACTIVITY_OPTIONS";
                    case ENTER_ANIMATION_COMPLETE: return "ENTER_ANIMATION_COMPLETE";
                    case LOCAL_VOICE_INTERACTION_STARTED: return "LOCAL_VOICE_INTERACTION_STARTED";
                    case ATTACH_AGENT: return "ATTACH_AGENT";
                    case APPLICATION_INFO_CHANGED: return "APPLICATION_INFO_CHANGED";
                    case RUN_ISOLATED_ENTRY_POINT: return "RUN_ISOLATED_ENTRY_POINT";
                    case EXECUTE_TRANSACTION: return "EXECUTE_TRANSACTION";
                    case RELAUNCH_ACTIVITY: return "RELAUNCH_ACTIVITY";
                }
            }
            return Integer.toString(code);
        }
        public void handleMessage(Message msg) {
            if (DEBUG_MESSAGES) Slog.v(TAG, ">>> handling: " + codeToString(msg.what));
            switch (msg.what) {
                case BIND_APPLICATION:
                    Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "bindApplication");
                    AppBindData data = (AppBindData)msg.obj;
                    handleBindApplication(data);
                    Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
                    break;
                case EXIT_APPLICATION:
                    if (mInitialApplication != null) {
                        mInitialApplication.onTerminate();
                    }
                    Looper.myLooper().quit();
                    break;
                case RECEIVER:
                    Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "broadcastReceiveComp");
                    handleReceiver((ReceiverData)msg.obj);
                    Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
                    break;
                case CREATE_SERVICE:
                    Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, ("serviceCreate: " + String.valueOf(msg.obj)));
                    handleCreateService((CreateServiceData)msg.obj);
                    Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
                    break;
                case BIND_SERVICE:
                    Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "serviceBind");
                    handleBindService((BindServiceData)msg.obj);
                    Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
                    break;
                case UNBIND_SERVICE:
                    Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "serviceUnbind");
                    handleUnbindService((BindServiceData)msg.obj);
                    Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
                    break;
                case SERVICE_ARGS:
                    Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, ("serviceStart: " + String.valueOf(msg.obj)));
                    handleServiceArgs((ServiceArgsData)msg.obj);
                    Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
                    break;
                case STOP_SERVICE:
                    Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "serviceStop");
                    handleStopService((IBinder)msg.obj);
                    Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
                    break;
                case CONFIGURATION_CHANGED:
                    handleConfigurationChanged((Configuration) msg.obj);
                    break;
                case CLEAN_UP_CONTEXT:
                    ContextCleanupInfo cci = (ContextCleanupInfo)msg.obj;
                    cci.context.performFinalCleanup(cci.who, cci.what);
                    break;
                case GC_WHEN_IDLE:
                    scheduleGcIdler();
                    break;
                case DUMP_SERVICE:
                    handleDumpService((DumpComponentInfo)msg.obj);
                    break;
                case LOW_MEMORY:
                    Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "lowMemory");
                    handleLowMemory();
                    Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
                    break;
                case PROFILER_CONTROL:
                    handleProfilerControl(msg.arg1 != 0, (ProfilerInfo)msg.obj, msg.arg2);
                    break;
                case CREATE_BACKUP_AGENT:
                    Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "backupCreateAgent");
                    handleCreateBackupAgent((CreateBackupAgentData)msg.obj);
                    Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
                    break;
                case DESTROY_BACKUP_AGENT:
                    Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "backupDestroyAgent");
                    handleDestroyBackupAgent((CreateBackupAgentData)msg.obj);
                    Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
                    break;
                case SUICIDE:
                    Process.killProcess(Process.myPid());
                    break;
                case REMOVE_PROVIDER:
                    Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "providerRemove");
                    completeRemoveProvider((ProviderRefCount)msg.obj);
                    Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
                    break;
                case ENABLE_JIT:
                    ensureJitEnabled();
                    break;
                case DISPATCH_PACKAGE_BROADCAST:
                    Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "broadcastPackage");
                    handleDispatchPackageBroadcast(msg.arg1, (String[])msg.obj);
                    Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
                    break;
                case SCHEDULE_CRASH:
                    throw new RemoteServiceException((String)msg.obj);
                case DUMP_HEAP:
                    handleDumpHeap((DumpHeapData) msg.obj);
                    break;
                case DUMP_ACTIVITY:
                    handleDumpActivity((DumpComponentInfo)msg.obj);
                    break;
                case DUMP_PROVIDER:
                    handleDumpProvider((DumpComponentInfo)msg.obj);
                    break;
                case SLEEPING:
                    Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "sleeping");
                    handleSleeping((IBinder)msg.obj, msg.arg1 != 0);
                    Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
                    break;
                case SET_CORE_SETTINGS:
                    Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "setCoreSettings");
                    handleSetCoreSettings((Bundle) msg.obj);
                    Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
                    break;
                case UPDATE_PACKAGE_COMPATIBILITY_INFO:
                    handleUpdatePackageCompatibilityInfo((UpdateCompatibilityData)msg.obj);
                    break;
                case UNSTABLE_PROVIDER_DIED:
                    handleUnstableProviderDied((IBinder)msg.obj, false);
                    break;
                case REQUEST_ASSIST_CONTEXT_EXTRAS:
                    handleRequestAssistContextExtras((RequestAssistContextExtras)msg.obj);
                    break;
                case TRANSLUCENT_CONVERSION_COMPLETE:
                    handleTranslucentConversionComplete((IBinder)msg.obj, msg.arg1 == 1);
                    break;
                case INSTALL_PROVIDER:
                    handleInstallProvider((ProviderInfo) msg.obj);
                    break;
                case ON_NEW_ACTIVITY_OPTIONS:
                    Pair<IBinder, ActivityOptions> pair = (Pair<IBinder, ActivityOptions>) msg.obj;
                    onNewActivityOptions(pair.first, pair.second);
                    break;
                case ENTER_ANIMATION_COMPLETE:
                    handleEnterAnimationComplete((IBinder) msg.obj);
                    break;
                case START_BINDER_TRACKING:
                    handleStartBinderTracking();
                    break;
                case STOP_BINDER_TRACKING_AND_DUMP:
                    handleStopBinderTrackingAndDump((ParcelFileDescriptor) msg.obj);
                    break;
                case LOCAL_VOICE_INTERACTION_STARTED:
                    handleLocalVoiceInteractionStarted((IBinder) ((SomeArgs) msg.obj).arg1,
                            (IVoiceInteractor) ((SomeArgs) msg.obj).arg2);
                    break;
                case ATTACH_AGENT: {
                    Application app = getApplication();
                    handleAttachAgent((String) msg.obj, app != null ? app.mLoadedApk : null);
                    break;
                }
                case APPLICATION_INFO_CHANGED:
                    mUpdatingSystemConfig = true;
                    try {
                        handleApplicationInfoChanged((ApplicationInfo) msg.obj);
                    } finally {
                        mUpdatingSystemConfig = false;
                    }
                    break;
                case RUN_ISOLATED_ENTRY_POINT:
                    handleRunIsolatedEntryPoint((String) ((SomeArgs) msg.obj).arg1,
                            (String[]) ((SomeArgs) msg.obj).arg2);
                    break;
                case EXECUTE_TRANSACTION:
                    final ClientTransaction transaction = (ClientTransaction) msg.obj;
                    mTransactionExecutor.execute(transaction);
                    if (isSystem()) {
                        // Client transactions inside system process are recycled on the client side
                        // instead of ClientLifecycleManager to avoid being cleared before this
                        // message is handled.
                        transaction.recycle();
                    }
                    // TODO(lifecycler): Recycle locally scheduled transactions.
                    break;
                case RELAUNCH_ACTIVITY:
                    handleRelaunchActivityLocally((IBinder) msg.obj);
                    break;
            }
            Object obj = msg.obj;
            if (obj instanceof SomeArgs) {
                ((SomeArgs) obj).recycle();
            }
            if (DEBUG_MESSAGES) Slog.v(TAG, "<<< done: " + codeToString(msg.what));
        }
    }

```