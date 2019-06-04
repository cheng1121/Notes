# SystemServer处理过程
SystemServer进程主要用于创建系统服务，比如AMS、WMS、PMS等都是由它创建的

## Zygote启动SystemServer
```
    private static Runnable forkSystemServer(String abiList, String socketName,
            ZygoteServer zygoteServer) {
         ...
        /* 
        * SystemServer需要的参数
        */
        String args[] = {
                "--setuid=1000",
                "--setgid=1000",
                "--setgroups=1001,1002,1003,1004,1005,1006,1007,1008,1009,1010,1018,1021,1023,"
                        + "1024,1032,1065,3001,3002,3003,3006,3007,3009,3010",
                "--capabilities=" + capabilities + "," + capabilities,
                "--nice-name=system_server",
                "--runtime-args",
                "--target-sdk-version=" + VMRuntime.SDK_VERSION_CUR_DEVELOPMENT,
                "com.android.server.SystemServer",
        };
        ZygoteArguments parsedArgs = null;

        int pid;

        try {
            parsedArgs = new ZygoteArguments(args);
            Zygote.applyDebuggerSystemProperty(parsedArgs);
            Zygote.applyInvokeWithSystemProperty(parsedArgs);

            boolean profileSystemServer = SystemProperties.getBoolean(
                    "dalvik.vm.profilesystemserver", false);
            if (profileSystemServer) {
                parsedArgs.mRuntimeFlags |= Zygote.PROFILE_SYSTEM_SERVER;
            }

            String use_app_image_cache = SystemProperties.get(
                    PROPERTY_USE_APP_IMAGE_STARTUP_CACHE, "");
            
            if (!TextUtils.isEmpty(use_app_image_cache) && !use_app_image_cache.equals("false")) {
                parsedArgs.mRuntimeFlags |= Zygote.USE_APP_IMAGE_STARTUP_CACHE;
            }

            //forkSystemServer进程
            pid = Zygote.forkSystemServer(
                    parsedArgs.mUid, parsedArgs.mGid,
                    parsedArgs.mGids,
                    parsedArgs.mRuntimeFlags,
                    null,
                    parsedArgs.mPermittedCapabilities,
                    parsedArgs.mEffectiveCapabilities);
        } catch (IllegalArgumentException ex) {
            throw new RuntimeException(ex);
        }

        
        if (pid == 0) {
        //运行在SystemServer进程中
            if (hasSecondZygote(abiList)) {
                waitForSecondaryZygote(socketName);
            }
            //关闭Zygote进程创建的Socket
            zygoteServer.closeServerSocket();
            //启动SystemServer进程
            return handleSystemServerProcess(parsedArgs);
        }

        return null;
    }

```
- forkSystemServer，复制一个和zygote相同的进程
- 关闭zygote进程创建的Socket，使用closeServerSocket方法
- 调用handleSystemServerProcess启动SystemServer进程


handleSystemServerProcess的内部逻辑
```
    private static Runnable handleSystemServerProcess(ZygoteArguments parsedArgs) {
         ...

        if (parsedArgs.mInvokeWith != null) {
           ...
        } else {
            //创建classLoader
            createSystemServerClassLoader();
            ClassLoader cl = sCachedSystemServerClassLoader;
            if (cl != null) {
                Thread.currentThread().setContextClassLoader(cl);
            }

            /*
             * Pass the remaining arguments to SystemServer.
             * 
             */
            return ZygoteInit.zygoteInit(parsedArgs.mTargetSdkVersion,
                    parsedArgs.mRemainingArgs, cl);
        }

        /* should never reach here */
    }

```
- 创建classLoader对象
- 调用zygoteInit方法


zygoteInit方法的作用
```
    public static final Runnable zygoteInit(int targetSdkVersion, String[] argv,
            ClassLoader classLoader) {
        if (RuntimeInit.DEBUG) {
            Slog.d(RuntimeInit.TAG, "RuntimeInit: Starting application from zygote");
        }

        Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "ZygoteInit");
        RuntimeInit.redirectLogStreams();

        RuntimeInit.commonInit();
        //执行native方法，创建binder线程池
        ZygoteInit.nativeZygoteInit();
        //进入SystemServer的main方法
        return RuntimeInit.applicationInit(targetSdkVersion, argv, classLoader);
    }

```

只需要关注两个方法就可以了：
- 执行nativeZygoteInit方法创建Binder线程池
- 执行RuntimeInit.applicationInit方法进入SystemServer的main方法中


首先来看下nativeZygoteInit方法：

它是一个JNI方法，其具体的实现在/frameworks/base/core/jni/AndroidRuntime.cpp文件中
```
static AndroidRuntime* gCurRuntime = NULL;

static void com_android_internal_os_ZygoteInit_nativeZygoteInit(JNIEnv* env, jobject clazz)
{
    gCurRuntime->onZygoteInit();
}
```
可以看到gCurRuntime是AndroidRuntime类型的指针，具体指向了/frameworks/base/cmds/app_process/app_main.cpp文件中的onZygoteInit函数
```
  virtual void onZygoteInit()
    {
        sp<ProcessState> proc = ProcessState::self();
        ALOGV("App process: starting thread pool.\n");
        proc->startThreadPool();
    }
```
onZygoteInit函数中通过startThreadPool开启了Binder线程池


接下来看下RuntimeInit.applicationInit方法
```
    protected static Runnable applicationInit(int targetSdkVersion, String[] argv,
            ClassLoader classLoader) {
         
        nativeSetExitWithoutCleanup(true);

       
        VMRuntime.getRuntime().setTargetHeapUtilization(0.75f);
        VMRuntime.getRuntime().setTargetSdkVersion(targetSdkVersion);

        final Arguments args = new Arguments(argv);

       
        Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);

        //查找静态main方法
        return findStaticMain(args.startClass, args.startArgs, classLoader);
    }

```
applicationInit方法中执行findStaticMain查找静态main方法

```
      protected static Runnable findStaticMain(String className, String[] argv,
            ClassLoader classLoader) {
        Class<?> cl;

        try {
            //反射获取对应的SystemServer类
            cl = Class.forName(className, true, classLoader);
        } catch (ClassNotFoundException ex) {
            throw new RuntimeException(
                    "Missing class when invoking static main " + className,
                    ex);
        }

        Method m;
        try {
            //找到main方法
            m = cl.getMethod("main", new Class[] { String[].class });
        } catch (NoSuchMethodException ex) {
            throw new RuntimeException(
                    "Missing static main on " + className, ex);
        } catch (SecurityException ex) {
            throw new RuntimeException(
                    "Problem getting static main on " + className, ex);
        }
        //判断是不是public和static修饰
        int modifiers = m.getModifiers();
        if (! (Modifier.isStatic(modifiers) && Modifier.isPublic(modifiers))) {
            throw new RuntimeException(
                    "Main method is not public and static on " + className);
        }

        /*
         * This throw gets caught in ZygoteInit.main(), which responds
         * by invoking the exception's run() method. This arrangement
         * clears up all the stack frames that were required in setting
         * up the process.
         */
        /**
         * 这个throw被ZygoteInit.main（）捕获，它通过调用异常的run（）方法来响应。 这种安排清除了设置过程所需的所有堆栈帧。
         */
        //调用main方法
        return new MethodAndArgsCaller(m, argv);
    }

```
抛出的异常交由ZygoteInit.main来捕获处理
```
  if (caller != null) {
            caller.run();
        }
```
在main方法的最后，有上面这一段代码。caller是Runnable对象，执行run方法后，就会在调用SystemServer的main方法

```
    static class MethodAndArgsCaller implements Runnable {
        /** method to call */
        private final Method mMethod;

        /** argument array */
        private final String[] mArgs;

        public MethodAndArgsCaller(Method method, String[] args) {
            mMethod = method;
            mArgs = args;
        }

        public void run() {
            try {
                //调用main方法
                mMethod.invoke(null, new Object[] { mArgs });
            } catch (IllegalAccessException ex) {
                throw new RuntimeException(ex);
            } catch (InvocationTargetException ex) {
                Throwable cause = ex.getCause();
                if (cause instanceof RuntimeException) {
                    throw (RuntimeException) cause;
                } else if (cause instanceof Error) {
                    throw (Error) cause;
                }
                throw new RuntimeException(ex);
            }
        }
    }

```

接下来看下SystemServer的main方法
```
  public static void main(String[] args) {
        new SystemServer().run();
    }
```
只有一句话，就是实例化当前的SystemServer，并执行run方法

```
    private void run() {
        try {
            ...
            
        
            //设置binder的最大线程数
            BinderInternal.setMaxThreads(sMaxBinderThreads);

            android.os.Process.setThreadPriority(
                android.os.Process.THREAD_PRIORITY_FOREGROUND);
            android.os.Process.setCanSelfBackground(false);
            //创建Looper 
            Looper.prepareMainLooper();
            Looper.getMainLooper().setSlowLogThresholdMs(
                    SLOW_DISPATCH_THRESHOLD_MS, SLOW_DELIVERY_THRESHOLD_MS);

         
            //加载android_servers.so动态库
            System.loadLibrary("android_servers");

           
            performPendingShutdown();


            //初始化系统context
            createSystemContext();

            //创建SystemServiceManager
            mSystemServiceManager = new SystemServiceManager(mSystemContext);
            mSystemServiceManager.setStartInfo(mRuntimeRestart,
                    mRuntimeStartElapsedTime, mRuntimeStartUptime);
            LocalServices.addService(SystemServiceManager.class, mSystemServiceManager);

            SystemServerInitThreadPool.get();
        } finally {
            traceEnd();  // InitBeforeStartServices
        }

        //开启服务
        try {
            traceBeginAndSlog("StartServices");
            //开启引导服务
            startBootstrapServices();
            //开启核心服务
            startCoreServices();
            //开启其他服务
            startOtherServices();
            SystemServerInitThreadPool.shutdown();
        } catch (Throwable ex) {
            Slog.e("System", "******************************************");
            Slog.e("System", "************ Failure starting system services", ex);
            throw ex;
        } finally {
            traceEnd();
        }

        ...
        
        Looper.loop();
        throw new RuntimeException("Main thread loop unexpectedly exited");
    }

```

省略了一部分代码，都是开启服务前的一些配置，这里不细看。看下主要逻辑
- 创建looper
- 加载动态库android_servers.so
- 初始化系统Context
- 创建SystemServiceManager，主要进行系统服务的创建、启动和生命周期管理
- 开启服务；主要分为三类：引导服务、核心服务、其他服务
- Looper.loop()等待消息




## 总结
SystemServer被创建后主要做了如下工作：
- 启动Binder线程池，这样就可以与其他进程进行通信
- 创建SystemServiceManager，其用于对系统的服务进行创建、启动和生命周期管理
- 启动各种系统服务








