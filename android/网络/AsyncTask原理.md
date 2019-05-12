AsyncTask旨在成为Thread和Handler的辅助类，并不构成通用的线程框架。AsyncTask被设计为执行端操作，耗时上线为几秒钟，如果需要保持长时间运行，建议使用线程或者线程池，或者其他工具

### AsyncTask的使用

 ##### 三个泛型的含义
1. 第一个泛型，为请求参数的类型 ，对应下方代码的String
2. 第二个泛型，为返回的进度类型，对应下方代码的Integer
3. 第三个泛型，为返回的结果，对应下方代码的Long
4. 如果不需要传入参数或者返回值，可以把这三个泛型设置为Void

```
  //运行在ui线程
  new MyAsyncTask().execute("name","age","gender");

   private static class MyAsyncTask extends AsyncTask<String, Integer, Long> {
        //运行在UI线程
        @Override
        protected void onPostExecute(Long aLong) {
            super.onPostExecute(aLong);
        }
        //运行在子线程
        @Override
        protected Long doInBackground(String... strings) {
             int count = strings.length;
             String name = strings[0];
             String age = strings[1];
             String gender = strings[2];
             //do request
            return null;
        }
        //运行在UI线程
        @Override
        protected void onPreExecute() {
            super.onPreExecute();
        }
        //运行在UI线程
        @Override
        protected void onProgressUpdate(Integer... values) {
            super.onProgressUpdate(values);
        }
    }
```
##### AsyncTask执行顺序
1. 在UI线程调用execute方法，并传入参数
2. UI线程中执行onPreExecute方法用于设置任务，比如显示进度条等
3. 在工作线程执行doInBackground方法，进行网路请求等等耗时任务；如果在此方法中调用publishProgress方法,会把当前任务的执行进度发送到onProgressUpdate方法内
4. 如果在doInBackground方法中调用了publishProgress方法，则此方法(onProgressUpdate)会执行。运行在UI线程，用于显示当前任务的进度
5. 当doInBackground中的任务执行完成后，会调用onPostExecute方法并把返回值传入。运行在UI线程，处理任务执行结果
6. 可以通过调用cancel()取消任务，调用此方法将导致isCancelled()返回true。cancel方法被调用后，doInBackground方法执行完毕后会调用onCancelled方法，而不是onPostExecute方法。

### AsyncTask源码
```
public abstract class AsyncTask<Params, Progress, Result> {
    private static final String LOG_TAG = "AsyncTask";
     //获取当前设备的cpu数
    private static final int CPU_COUNT = Runtime.getRuntime().availableProcessors();
    //设置核心线程数量，最少2个，最多4个，如果cpu数量-1在2～4之内，则选择cpu数量-1
    private static final int CORE_POOL_SIZE = Math.max(2, Math.min(CPU_COUNT - 1, 4));
    //最大线程数量 cpu数量* 2 +1
    private static final int MAXIMUM_POOL_SIZE = CPU_COUNT * 2 + 1;
    //空闲线程存活时间 30秒
    private static final int KEEP_ALIVE_SECONDS = 30;
    //线程工厂
    private static final ThreadFactory sThreadFactory = new ThreadFactory() {
           
     //原子操作类，线程池编号 在并发执行时不会阻塞其他线程
     private final AtomicInteger mCount = new AtomicInteger(1);
        //创建线程
        public Thread newThread(Runnable r) {
            return new Thread(r, "AsyncTask #" + mCount.getAndIncrement());
        }
    };
    //阻塞队列    LinkedBlockingQueue一个由链表结构组成的有界队列，如果不指定长度的画，
    //此队列的长度为Integer.MAX_VALUE。此队列按照先进先出的顺序进行排序。
    private static final BlockingQueue<Runnable> sPoolWorkQueue =
            new LinkedBlockingQueue<Runnable>(128);

    /**
     * 执行任务的线程池
     */
    public static final Executor THREAD_POOL_EXECUTOR;

    static {
        //线程池 核心线程数CORE_POOL_SIZE ，最大线程数MAXIMUM_POOL_SIZE
       //KEEP_ALIVE_SECONDS 空闲线程存活时间   
        ThreadPoolExecutor threadPoolExecutor = new ThreadPoolExecutor(
                CORE_POOL_SIZE, MAXIMUM_POOL_SIZE, KEEP_ALIVE_SECONDS, TimeUnit.SECONDS,
                sPoolWorkQueue, sThreadFactory);
        //允许核心线程空闲退出
        threadPoolExecutor.allowCoreThreadTimeOut(true);
        THREAD_POOL_EXECUTOR = threadPoolExecutor;
    }

    /**
     * 串行任务队列
     */
    public static final Executor SERIAL_EXECUTOR = new SerialExecutor();
    //handler消息     结果code
    private static final int MESSAGE_POST_RESULT = 0x1;
    //handler消息     进度code
    private static final int MESSAGE_POST_PROGRESS = 0x2;
    //默认执行任务的为 串行执行
    private static volatile Executor sDefaultExecutor = SERIAL_EXECUTOR;
    //handler 获取主线程Looper 并向主线程发送消息
    private static InternalHandler sHandler;
    //实现了Callable接口，使线程具备返回值
    private final WorkerRunnable<Params, Result> mWorker;
    //执行的任务可取消，和监听任务是否完成
    private final FutureTask<Result> mFuture;
    //设置AsyncTask当前的状态，默认为任务未开始执行
    private volatile Status mStatus = Status.PENDING;
    //原子操作 布尔类型，任务是否取消
    private final AtomicBoolean mCancelled = new AtomicBoolean();
    //原子操作  任务是否被执行过
    private final AtomicBoolean mTaskInvoked = new AtomicBoolean();
    //getHandler方法返回给调用者的Handler，如果传入了UI线程的Looper
   //则返回的是InternalHandler，否则是Handler
    private final Handler mHandler;
    //串行任务队列
    private static class SerialExecutor implements Executor {
        //任务队列 实现于Deque，拥有队列或者栈特性的接口
       // 实现于Cloneable，拥有克隆对象的特性
       //实现于Serializable，拥有序列化的能力 
       //线性双向队列，用于存储所有任务
        final ArrayDeque<Runnable> mTasks = new ArrayDeque<Runnable>();
        //正在执行的任务  
        Runnable mActive;
 
        public synchronized void execute(final Runnable r) {
         //将任务加入到双向队列中
            mTasks.offer(new Runnable() {
                public void run() {
                    try {
                        //执行任务
                        r.run();
                    } finally {
                       //等待当前执行的任务执行完毕后，进入执行逻辑
                        scheduleNext();
                    }
                }
            });
            //如果当前没有执行的任务，则直接进入执行逻辑
            if (mActive == null) {
                scheduleNext();
            }
        }
       //从队列中取出队列头的任务交给线程池去执行
        protected synchronized void scheduleNext() {
            if ((mActive = mTasks.poll()) != null) {
                THREAD_POOL_EXECUTOR.execute(mActive);
            }
        }
    }

    /**
     * AsyncTask的状态
     */
    public enum Status {
        /**
         *AsyncTask还未开始执行任务
         */
        PENDING,
        /**
         * AsyncTask正在执行任务
         */
        RUNNING,
        /**
         * AsyncTask任务执行完毕
         */
        FINISHED,
    }
     //获取主线程中的Handler
    private static Handler getMainHandler() {
        synchronized (AsyncTask.class) {
            if (sHandler == null) {
                sHandler = new InternalHandler(Looper.getMainLooper());
            }
            return sHandler;
        }
    }
    //获取AsyncTask中的Handler
    private Handler getHandler() {
        return mHandler;
    }
     
    //隐藏API 设置默认执行器
    public static void setDefaultExecutor(Executor exec) {
        sDefaultExecutor = exec;
    }

    /**
     * 默认构造方法
     */
    public AsyncTask() {
        this((Looper) null);
    }

    /**
     *构造方法，必须在UI线程中调用
     */
    public AsyncTask(@Nullable Handler handler) {
        this(handler != null ? handler.getLooper() : null);
    }

    /**
     * 构造方法，必须在UI线程中调用
     */
    public AsyncTask(@Nullable Looper callbackLooper) {
         //得到主线程的Handler
        mHandler = callbackLooper == null || callbackLooper == Looper.getMainLooper()
            ? getMainHandler()
            : new Handler(callbackLooper);
         //WorkerRunnable实现了Callable接口，使线程具备了返回数据的能力
        mWorker = new WorkerRunnable<Params, Result>() {
            public Result call() throws Exception {
                //原子操作，设置任务已被执行
                mTaskInvoked.set(true);
                 //任务执行结果
                Result result = null;
                try {
                   //设置当前线程优先级为标准后台线程
                 Process.setThreadPriority(Process.THREAD_PRIORITY_BACKGROUND);
                    //执行用户写好的逻辑
                    result = doInBackground(mParams);
                    //将进程中未执行的命令，一并送入CPU进行处理
                    Binder.flushPendingCommands();
                } catch (Throwable tr) {
                   //产生异常，任务取消执行
                    mCancelled.set(true);
                    throw tr;
                } finally {
                    //把任务处理结果发送到主线程
                    postResult(result);
                }
                //返回任务处理结果
                return result;
            }
        };
       //包装任务的包装类
        mFuture = new FutureTask<Result>(mWorker) {
            @Override
            protected void done() {
                try {
                    //发送所有未被调用的结果一并发出
                    postResultIfNotInvoked(get());
                } catch (InterruptedException e) {
                    android.util.Log.w(LOG_TAG, e);
                } catch (ExecutionException e) {
                    throw new RuntimeException("An error occurred while executing doInBackground()",
                            e.getCause());
                } catch (CancellationException e) {
                    postResultIfNotInvoked(null);
                }
            }
        };
    }

    private void postResultIfNotInvoked(Result result) {
         //判断任务是否被执行过
        final boolean wasTaskInvoked = mTaskInvoked.get();
        //如果未被执行，则把结果发送出去
        if (!wasTaskInvoked) {
            postResult(result);
        }
    }
    //handler发送结果
    private Result postResult(Result result) {
        @SuppressWarnings("unchecked")
        Message message = getHandler().obtainMessage(MESSAGE_POST_RESULT,
                new AsyncTaskResult<Result>(this, result));
        message.sendToTarget();
        return result;
    }

    /**
     * 返回当前AsyncTask的状态
     */
    public final Status getStatus() {
        return mStatus;
    }

    /**
     *   抽象方法，需要重写
     *   在工作线程中执行
     */
    @WorkerThread
    protected abstract Result doInBackground(Params... params);

    /**
     * 运行在UI线程，在doInBackground方法之前执行
     */
    @MainThread
    protected void onPreExecute() {
    }

    /**
     * 运行在UI线程，doInBackground方法执行完成后执行，返回任务的处理结果
     */
    @SuppressWarnings({"UnusedDeclaration"})
    @MainThread
    protected void onPostExecute(Result result) {
    }

    /**
     * 在doInBackground方法中执行publishProgress方法后，执行该方法，返回任务执行进度
     */
    @SuppressWarnings({"UnusedDeclaration"})
    @MainThread
    protected void onProgressUpdate(Progress... values) {
    }

    /**
     * 运行在UI线程，cancel(boolean)方法被调用后，执行该方法
     */
    @SuppressWarnings({"UnusedParameters"})
    @MainThread
    protected void onCancelled(Result result) {
        onCancelled();
    }    
    
    /**
     * 任务被取消的回调
     */
    @MainThread
    protected void onCancelled() {
    }

    /**
     *    任务是否取消
     */
    public final boolean isCancelled() {
        return mCancelled.get();
    }

    /**
     * 取消任务
     */
    public final boolean cancel(boolean mayInterruptIfRunning) {
        mCancelled.set(true);
        return mFuture.cancel(mayInterruptIfRunning);
    }

    /**
     * 获取任务结果
     */
    public final Result get() throws InterruptedException, ExecutionException {
        return mFuture.get();
    }

    /**
     *  在指定时间内获取结果
     */
    public final Result get(long timeout, TimeUnit unit) throws InterruptedException,
            ExecutionException, TimeoutException {
        return mFuture.get(timeout, unit);
    }

    /**
     *  执行任务，在UI线程中执行
     */
    @MainThread
    public final AsyncTask<Params, Progress, Result> execute(Params... params) {      
        return executeOnExecutor(sDefaultExecutor, params);
    }

    /**
     *  执行任务
     */
    @MainThread
    public final AsyncTask<Params, Progress, Result> executeOnExecutor(Executor exec,
            Params... params) {
       //判断是否为运行状态或者结束状态，如果是则抛出异常
        if (mStatus != Status.PENDING) {
            switch (mStatus) {
                case RUNNING:
                    throw new IllegalStateException("Cannot execute task:"
                            + " the task is already running.");
                case FINISHED:
                    throw new IllegalStateException("Cannot execute task:"
                            + " the task has already been executed "
                            + "(a task can be executed only once)");
            }
        }
        //设置当前AsyncTask的状态为运行状态
        mStatus = Status.RUNNING;
       //进行准备工作
        onPreExecute();
         //将参数加入到任务中
        mWorker.mParams = params;
        //执行任务
        exec.execute(mFuture);

        return this;
    }

    /**
     *直接执行一个runnable
     */
    @MainThread
    public static void execute(Runnable runnable) {
        sDefaultExecutor.execute(runnable);
    }

    /**
     *  发送当前任务的进度到onProgressUpdate方法中
     */
    @WorkerThread
    protected final void publishProgress(Progress... values) {
        if (!isCancelled()) {
            getHandler().obtainMessage(MESSAGE_POST_PROGRESS,
                    new AsyncTaskResult<Progress>(this, values)).sendToTarget();
        }
    }
     //任务执行完毕，判断如果任务未完成则执行onPostExecute
    private void finish(Result result) {
        if (isCancelled()) {
            onCancelled(result);
        } else {
            onPostExecute(result);
        }
        mStatus = Status.FINISHED;
    }
    //Handler  处理接收到的消息 和UI交互
    private static class InternalHandler extends Handler {
        public InternalHandler(Looper looper) {
            super(looper);
        }

        @SuppressWarnings({"unchecked", "RawUseOfParameterizedType"})
        @Override
        public void handleMessage(Message msg) {
            AsyncTaskResult<?> result = (AsyncTaskResult<?>) msg.obj;
            switch (msg.what) {
                case MESSAGE_POST_RESULT:
                    //将结果发送到UI线程，一次只发送一个结果过去
                    // There is only one result
                    result.mTask.finish(result.mData[0]);
                    break;
                case MESSAGE_POST_PROGRESS:
                   //更新进度
                    result.mTask.onProgressUpdate(result.mData);
                    break;
            }
        }
    }

    private static abstract class WorkerRunnable<Params, Result> implements Callable<Result> {
        Params[] mParams;
    }

    @SuppressWarnings({"RawUseOfParameterizedType"})
    private static class AsyncTaskResult<Data> {
        final AsyncTask mTask;
        final Data[] mData;

        AsyncTaskResult(AsyncTask task, Data... data) {
            mTask = task;
            mData = data;
        }
    }
}

```

##### 并行和串行
AsyncTask默认是串行执行任务，不过在android3.0之前是并行执行任务；现在向要并行执行任务的话可以使用如下方法：
```
//并行执行任务   
MyAsyncTask myAsyncTask = new MyAsyncTask();
myAsyncTask.executeOnExecutor(AsyncTask.THREAD_POOL_EXECUTOR,"name","age","gender");

```