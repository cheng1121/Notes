# Zygote进程启动过程

当init进程通过fork()或者clone()创建子进程后，就会进入到Zygote进程的启动过程，主要分为以下几步：

### 创建AndroidRuntime并执行start方法
 
```
int main(int argc, char* const argv[])
{
    ...
    //解析指令参数
    ++i;  // Skip unused "parent dir" argument.
    while (i < argc) {
        const char* arg = argv[i++];
        if (strcmp(arg, "--zygote") == 0) {
            //如果当前运行在Zygote进程中，则将zygote设置为true
            zygote = true;
            niceName = ZYGOTE_NICE_NAME;
        } else if (strcmp(arg, "--start-system-server") == 0) {
            //如果当前运行在SystemServer进程中，则将startSystemServer设置为true
            startSystemServer = true;
        } else if (strcmp(arg, "--application") == 0) {
            application = true;
        } else if (strncmp(arg, "--nice-name=", 12) == 0) {
            niceName.setTo(arg + 12);
        } else if (strncmp(arg, "--", 2) != 0) {
            className.setTo(arg);
            break;
        } else {
            --i;
            break;
        }
    }
    
    ...
    
    //运行在zygote进程中
    if (zygote) {
        runtime.start("com.android.internal.os.ZygoteInit", args, zygote);
    } else if (className) {
        runtime.start("com.android.internal.os.RuntimeInit", args, zygote);
    } else {
        fprintf(stderr, "Error: no class name or --zygote supplied.\n");
        app_usage();
        LOG_ALWAYS_FATAL("app_process: no class name or --zygote supplied.");
    }
}
```
可以看到最后会调用start函数启动zygote,runtime是AndroidRuntime。start函数是一种jni调用，会调用java中main方法继续执行,start函数位于/frameworks/base/core/jni/AndroidRuntime.cpp
```
void AndroidRuntime::start(const char* className, const Vector<String8>& options, bool zygote)
{
     ...
    
    //配置ANDROID_ROOT环境变量
    const char* rootDir = getenv("ANDROID_ROOT");
    if (rootDir == NULL) {
        rootDir = "/system";
        if (!hasDir("/system")) {
            LOG_FATAL("No root directory specified, and /system does not exist.");
            return;
        }
        setenv("ANDROID_ROOT", rootDir, 1);
    }

    const char* runtimeRootDir = getenv("ANDROID_RUNTIME_ROOT");
    ...
    
    const char* tzdataRootDir = getenv("ANDROID_TZDATA_ROOT");
    ...

    
    //启动Java虚拟机
    JniInvocation jni_invocation;
    jni_invocation.Init(NULL);
    JNIEnv* env;
    if (startVm(&mJavaVM, &env, zygote) != 0) {
        return;
    }
    onVmCreated(env);

     /*
     * 为Java虚拟机注册JNI方法
     */
    if (startReg(env) < 0) {
        ALOGE("Unable to register all android natives\n");
        return;
    }

    /*
     * We want to call main() with a String array with arguments in it.
     * At present we have two arguments, the class name and an option string.
     * Create an array to hold them.
     */
    jclass stringClass;
    jobjectArray strArray;
    jstring classNameStr;

    stringClass = env->FindClass("java/lang/String");
    assert(stringClass != NULL);
    strArray = env->NewObjectArray(options.size() + 1, stringClass, NULL);
    assert(strArray != NULL);
    //从app_main的main函数得知className为com.android.internal.os.ZygoteInit
    classNameStr = env->NewStringUTF(className);
    assert(classNameStr != NULL);
    env->SetObjectArrayElement(strArray, 0, classNameStr);

    for (size_t i = 0; i < options.size(); ++i) {
        jstring optionsStr = env->NewStringUTF(options.itemAt(i).string());
        assert(optionsStr != NULL);
        env->SetObjectArrayElement(strArray, i + 1, optionsStr);
    }

    /*
     * Start VM.  This thread becomes the main thread of the VM, and will
     * not return until the VM exits.
     */
     //启动虚拟机的线程，就是当前虚拟机的主线程，直到虚拟机退出
     //将className的.替换为/
    char* slashClassName = toSlashClassName(className != NULL ? className : "");
    //找到ZygoteInit
    jclass startClass = env->FindClass(slashClassName);
    if (startClass == NULL) {
        ALOGE("JavaVM unable to locate class '%s'\n", slashClassName);
        /* keep going */
    } else {
        //找到ZygoteInit的main方法
        jmethodID startMeth = env->GetStaticMethodID(startClass, "main",
            "([Ljava/lang/String;)V");
        if (startMeth == NULL) {
            ALOGE("JavaVM unable to find main() in '%s'\n", className);
            /* keep going */
        } else {
           //通过JNI调用ZygoteInit的main方法
            env->CallStaticVoidMethod(startClass, startMeth, strArray);

#if 0
            if (env->ExceptionCheck())
                threadExitUncaughtException(env);
#endif
        }
    }
    free(slashClassName);

    ALOGD("Shutting down VM\n");
    if (mJavaVM->DetachCurrentThread() != JNI_OK)
        ALOGW("Warning: unable to detach main thread\n");
    if (mJavaVM->DestroyJavaVM() != 0)
        ALOGW("Warning: VM did not shut down cleanly\n");
  ...
 
```
start()函数的执行逻辑是：
1. 配置环境变量
2. 创建Java虚拟机，并注册JNI方法
3. 找到ZygoteInit和其main方法，然后通过JNI调用main方法


ZygoteInit的main方法时由Java语言编写的，当前的运行逻辑在Native中，这就需要通过JNI来调用Java。这样Zygote就从Native层进入了Java框架层

### ZygoteInit的Main方法
```
    @UnsupportedAppUsage
    public static void main(String argv[]) {
        /**
         * 实例化ZygoteServer
         */
        ZygoteServer zygoteServer = new ZygoteServer();

        // Mark zygote start. This ensures that thread creation will throw
        // an error.
        ZygoteHooks.startZygoteNoThreadCreation();

        // Zygote goes into its own process group.
        try {
            Os.setpgid(0, 0);
        } catch (ErrnoException ex) {
            throw new RuntimeException("Failed to setpgid(0,0)", ex);
        }

        Runnable caller;
        try {
            // Report Zygote start time to tron unless it is a runtime restart
            if (!"1".equals(SystemProperties.get("sys.boot_completed"))) {
                MetricsLogger.histogram(null, "boot_zygote_init",
                        (int) SystemClock.elapsedRealtime());
            }

            String bootTimeTag = Process.is64Bit() ? "Zygote64Timing" : "Zygote32Timing";
            TimingsTraceLog bootTimingsTraceLog = new TimingsTraceLog(bootTimeTag,
                    Trace.TRACE_TAG_DALVIK);
            bootTimingsTraceLog.traceBegin("ZygoteInit");
            RuntimeInit.enableDdms();

            boolean startSystemServer = false;
            String socketName = "zygote";
            String abiList = null;
            boolean enableLazyPreload = false;
            //解析命令
            for (int i = 1; i < argv.length; i++) {
                if ("start-system-server".equals(argv[i])) {
                    startSystemServer = true;
                } else if ("--enable-lazy-preload".equals(argv[i])) {
                    enableLazyPreload = true;
                } else if (argv[i].startsWith(ABI_LIST_ARG)) {
                    abiList = argv[i].substring(ABI_LIST_ARG.length());
                } else if (argv[i].startsWith(SOCKET_NAME_ARG)) {
                    socketName = argv[i].substring(SOCKET_NAME_ARG.length());
                } else {
                    throw new RuntimeException("Unknown command line argument: " + argv[i]);
                }
            }

            if (abiList == null) {
                throw new RuntimeException("No ABI list supplied.");
            }

            // TODO (chriswailes): Wrap these three calls in a helper function?
            final String blastulaSocketName =
                    socketName.equals(ZygoteProcess.ZYGOTE_SOCKET_NAME)
                            ? ZygoteProcess.BLASTULA_POOL_SOCKET_NAME
                            : ZygoteProcess.BLASTULA_POOL_SECONDARY_SOCKET_NAME;

            //创建一个Server端的Socket，socketName的值为zygote
            zygoteServer.createZygoteSocket(socketName);
            Zygote.createBlastulaSocket(blastulaSocketName);

            Zygote.getSocketFDs(socketName.equals(ZygoteProcess.ZYGOTE_SOCKET_NAME));

            // In some configurations, we avoid preloading resources and classes eagerly.
            // In such cases, we will preload things prior to our first fork.
            if (!enableLazyPreload) {
                bootTimingsTraceLog.traceBegin("ZygotePreload");
                EventLog.writeEvent(LOG_BOOT_PROGRESS_PRELOAD_START,
                        SystemClock.uptimeMillis());
                //预加载类和资源
                preload(bootTimingsTraceLog);
                EventLog.writeEvent(LOG_BOOT_PROGRESS_PRELOAD_END,
                        SystemClock.uptimeMillis());
                bootTimingsTraceLog.traceEnd(); // ZygotePreload
            } else {
                Zygote.resetNicePriority();
            }

            // Do an initial gc to clean up after startup
            bootTimingsTraceLog.traceBegin("PostZygoteInitGC");
            gcAndFinalize();
            bootTimingsTraceLog.traceEnd(); // PostZygoteInitGC

            bootTimingsTraceLog.traceEnd(); // ZygoteInit
            // Disable tracing so that forked processes do not inherit stale tracing tags from
            // Zygote.
            Trace.setTracingEnabled(false, 0);

            Zygote.nativeSecurityInit();

            // Zygote process unmounts root storage spaces.
            Zygote.nativeUnmountStorageOnInit();

            ZygoteHooks.stopZygoteNoThreadCreation();

            if (startSystemServer) {
                //创建SystemServer进程
                Runnable r = forkSystemServer(abiList, socketName, zygoteServer);

                // {@code r == null} in the parent (zygote) process, and {@code r != null} in the
                // child (system_server) process.
                if (r != null) {
                    r.run();
                    return;
                }
            }

            // If the return value is null then this is the zygote process
            // returning to the normal control flow.  If it returns a Runnable
            // object then this is a blastula that has finished specializing.
            caller = Zygote.initBlastulaPool();

            if (caller == null) {
                Log.i(TAG, "Accepting command socket connections");

                // The select loop returns early in the child process after a fork and
                // loops forever in the zygote.
                //等待AMS请求
                caller = zygoteServer.runSelectLoop(abiList);
            }
        } catch (Throwable ex) {
            Log.e(TAG, "System zygote died with exception", ex);
            throw ex;
        } finally {
            zygoteServer.closeServerSocket();
        }

        // We're in the child process and have exited the select loop. Proceed to execute the
        // command.
        if (caller != null) {
            caller.run();
        }
    }

```
代码逻辑：
- 实例化ZygoteServer
- 解析传递过来的命令
- 使用zygoteServer的createZygoteSocket方法创建socket，名字为zygote
- 根据enableLazyPreload来判断是否执行preload方法，进行预加载类和资源
- 使用forkSystemServer方法，创建SystemServer进程
- 使用zygoteServer的runSelectLoop方法来获取AMS请求


## Zygote进程启动总结
- 创建AndroidRuntime并调用其start方法，启动Zygote进程
- 创建Java虚拟机并为Java虚拟机注册JNI方法
- 通过JNI调用ZygoteInit的main方法，进入Zygote的Java框架层
- 通过createZygoteScoket方法创建服务器端Socket，并通过runSelectLoop方法等待AMS的请求来创建新的应用程序进程
- 启动SystemServer进程


