# Activity的启动过程

## Launcher向AMS发送请求的过程
当我们点击某一应用图标时，Launcher首先会向AMS请求开启这个应用

首先点击应用图标时会触发startActivitySafely方法,该方法是BaseDraggingActivity的方法，Launcher类中重写了该方法。源码版本为Android 9.0

```
    public boolean startActivitySafely(View v, Intent intent, ItemInfo item) {
        boolean success = super.startActivitySafely(v, intent, item);
        if (success && v instanceof BubbleTextView) {
            BubbleTextView btv = (BubbleTextView) v;
            btv.setStayPressed(true);
            setOnResumeCallback(btv);
        }
        return success;
    }

```
上面为Launcher中重写的startActivitySafely方法，可以看到其也是调用了父类的startActivitySafely方法
```
    public boolean startActivitySafely(View v, Intent intent, ItemInfo item) {
        if (mIsSafeModeEnabled && !Utilities.isSystemApp(this, intent)) {
            Toast.makeText(this, R.string.safemode_shortcut_error, Toast.LENGTH_SHORT).show();
            return false;
        }

        // Only launch using the new animation if the shortcut has not opted out (this is a
        // private contract between launcher and may be ignored in the future).
        boolean useLaunchAnimation = (v != null) &&
                !intent.hasExtra(INTENT_EXTRA_IGNORE_LAUNCH_ANIMATION);
        Bundle optsBundle = useLaunchAnimation
                ? getActivityLaunchOptionsAsBundle(v)
                : null;

        UserHandle user = item == null ? null : item.user;

        // Prepare intent
        //添加Activity启动模式标记FLAG_ACTIVITY_NEW_TASK
        intent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
        if (v != null) {
            intent.setSourceBounds(getViewBounds(v));
        }
        try {
            boolean isShortcut = Utilities.ATLEAST_MARSHMALLOW
                    && (item instanceof ShortcutInfo)
                    && (item.itemType == Favorites.ITEM_TYPE_SHORTCUT
                    || item.itemType == Favorites.ITEM_TYPE_DEEP_SHORTCUT)
                    && !((ShortcutInfo) item).isPromise();
            if (isShortcut) {
                // Shortcuts need some special checks due to legacy reasons.
                //由于遗留原因，快捷方式需要进行一些特殊检查。
                startShortcutIntentSafely(intent, optsBundle, item);
            } else if (user == null || user.equals(Process.myUserHandle())) {
                // Could be launching some bookkeeping activity
                //启动Activity
                startActivity(intent, optsBundle);
            } else {
                LauncherAppsCompat.getInstance(this).startActivityForProfile(
                        intent.getComponent(), user, intent.getSourceBounds(), optsBundle);
            }
            getUserEventDispatcher().logAppLaunch(v, intent);
            return true;
        } catch (ActivityNotFoundException|SecurityException e) {
            Toast.makeText(this, R.string.activity_not_found, Toast.LENGTH_SHORT).show();
            Log.e(TAG, "Unable to launch. tag=" + item + " intent=" + intent, e);
        }
        return false;
    }


```
- 在intent中添加Activity的启动模式标记FLAG_ACTIVITY_NEW_TASK
- 判断是否有快捷方式
- 调用startActivity方法启动Activity

```
 @Override
    public void startActivity(Intent intent, @Nullable Bundle options) {
        if (options != null) {
            startActivityForResult(intent, -1, options);
        } else {
            // Note we want to go through this call for compatibility with
            // applications that may have overridden the method.
            startActivityForResult(intent, -1);
        }
    }

```
startActivity调用了startActivityForResult方法,第二个参数为-1表示不需要知道Activity的启动结果
```
    public void startActivityForResult(@RequiresPermission Intent intent, int requestCode,
            @Nullable Bundle options) {
        if (mParent == null) {
            options = transferSpringboardActivityOptions(options);
            Instrumentation.ActivityResult ar =
                mInstrumentation.execStartActivity(
                    this, mMainThread.getApplicationThread(), mToken, this,
                    intent, requestCode, options);
            if (ar != null) {
                mMainThread.sendActivityResult(
                    mToken, mEmbeddedID, requestCode, ar.getResultCode(),
                    ar.getResultData());
            }
            if (requestCode >= 0) {
              
                mStartedActivity = true;
            }

            cancelInputsAndStartExitTransition(options);
         
        } else {
            if (options != null) {
                mParent.startActivityFromChild(this, intent, requestCode, options);
            } else {
                mParent.startActivityFromChild(this, intent, requestCode);
            }
        }
    }


```

- mParent表示当前的Activity的父类，因为根Activity还未创建出来，所以mParent=null

- 然后调用mInstrumentation.execStartActivity方法，Instrumentation主要是用来监控应用程序和系统的交互


```
public ActivityResult execStartActivity(
            Context who, IBinder contextThread, IBinder token, Activity target,
            Intent intent, int requestCode, Bundle options) {
        IApplicationThread whoThread = (IApplicationThread) contextThread;
        Uri referrer = target != null ? target.onProvideReferrer() : null;
        if (referrer != null) {
            intent.putExtra(Intent.EXTRA_REFERRER, referrer);
        }
        ...
        try {
            intent.migrateExtraStreamToClipData();
            intent.prepareToLeaveProcess(who);
            //获取AMS的代理对象并调用它的startActivity方法
            int result = ActivityManager.getService()
                .startActivity(whoThread, who.getBasePackageName(), intent,
                        intent.resolveTypeIfNeeded(who.getContentResolver()),
                        token, target != null ? target.mEmbeddedID : null,
                        requestCode, 0, null, options);
            checkStartActivityResult(result, intent);
        } catch (RemoteException e) {
            throw new RuntimeException("Failure from system", e);
        }
        return null;
    }


```
首先通过ActivityManager.getService方法获得AMS的代理对象然后通过代理对象调用startActivity方法。接下来看下getService方法做了什么

```
  public static IActivityManager getService() {
        return IActivityManagerSingleton.get();
    }

    @UnsupportedAppUsage
    private static final Singleton<IActivityManager> IActivityManagerSingleton =
            new Singleton<IActivityManager>() {
                @Override
                protected IActivityManager create() {
                    //获取AMS对应的IBinder实例
                    final IBinder b = ServiceManager.getService(Context.ACTIVITY_SERVICE);
                    //通过IBinder对象获取AMS代理对象
                    final IActivityManager am = IActivityManager.Stub.asInterface(b);
                    return am;
                }
            };

```

getService方法调用了IActivityManagerSingleton的get方法,通过下方可以看到IActivityManagerSingleton是Singleton类

首先得到名字为ACTIVITY_SERVICE也就是activity的service引用，也就是IBinder类型的AMS引用；然后通过这个IBinder引用获得IActivityManager对象。这段是AIDL

SystemServer进程开启系统服务时会把服务对应的IBinder对象存放在ServiceManager中。

ServiceManager是一个单例对象，是系统服务的管理类



## AMS到ApplicationThread的调用过程
启动Activity的消息通过AIDL传递到AMS中后，会调用AMS中的startActivity方法


```
    @Override
    public final int startActivity(IApplicationThread caller, String callingPackage,
            Intent intent, String resolvedType, IBinder resultTo, String resultWho, int requestCode,
            int startFlags, ProfilerInfo profilerInfo, Bundle bOptions) {
        return startActivityAsUser(caller, callingPackage, intent, resolvedType, resultTo,
                resultWho, requestCode, startFlags, profilerInfo, bOptions,
                UserHandle.getCallingUserId());
    }

```
startActivity方法会调动startActivityAsUser方法,startActivityAsUser有多个重载方法，这里只放最终调用的startActivityAsUser方法

```
       public final int startActivityAsUser(IApplicationThread caller, String callingPackage,
            Intent intent, String resolvedType, IBinder resultTo, String resultWho, int requestCode,
            int startFlags, ProfilerInfo profilerInfo, Bundle bOptions, int userId,
            boolean validateIncomingUser) {
        //判断调用者进程是否被隔离
        enforceNotIsolatedCaller("startActivity");
        //检查调用者的权限
        userId = mActivityStartController.checkTargetUser(userId, validateIncomingUser,
                Binder.getCallingPid(), Binder.getCallingUid(), "startActivityAsUser");

        // TODO: Switch to user app stacks here.
        //切换到用户应用程序堆栈
        return mActivityStartController.obtainStarter(intent, "startActivityAsUser")
                .setCaller(caller)
                .setCallingPackage(callingPackage)
                .setResolvedType(resolvedType)
                .setResultTo(resultTo)
                .setResultWho(resultWho)
                .setRequestCode(requestCode)
                .setStartFlags(startFlags)
                .setProfilerInfo(profilerInfo)
                .setActivityOptions(bOptions)
                .setMayWait(userId)
                .execute();

    }

```
这里的逻辑和Android8.0时不同，这里添加了一个控制类ActivityStartController，用于管理ActivityStarter类，ActivityStarter作用是收集所有的逻辑来决定如何将Intent和Flags转换为Activity,并将Activity和Task以及Stack相关联。

mActivityStartController.obtainStarter方法返回ActivityStarter类

ActivityStarter使用了Builder模式，最终调用execute方法来打开Activity


```
    int execute() {
        try {
            // TODO(b/64750076): Look into passing request directly to these methods to allow
            // for transactional diffs and preprocessing.
            if (mRequest.mayWait) {
                return startActivityMayWait(mRequest.caller, mRequest.callingUid,
                        mRequest.callingPackage, mRequest.intent, mRequest.resolvedType,
                        mRequest.voiceSession, mRequest.voiceInteractor, mRequest.resultTo,
                        mRequest.resultWho, mRequest.requestCode, mRequest.startFlags,
                        mRequest.profilerInfo, mRequest.waitResult, mRequest.globalConfig,
                        mRequest.activityOptions, mRequest.ignoreTargetSecurity, mRequest.userId,
                        mRequest.inTask, mRequest.reason,
                        mRequest.allowPendingRemoteAnimationRegistryLookup,
                        mRequest.originatingPendingIntent);
            } else {
                return startActivity(mRequest.caller, mRequest.intent, mRequest.ephemeralIntent,
                        mRequest.resolvedType, mRequest.activityInfo, mRequest.resolveInfo,
                        mRequest.voiceSession, mRequest.voiceInteractor, mRequest.resultTo,
                        mRequest.resultWho, mRequest.requestCode, mRequest.callingPid,
                        mRequest.callingUid, mRequest.callingPackage, mRequest.realCallingPid,
                        mRequest.realCallingUid, mRequest.startFlags, mRequest.activityOptions,
                        mRequest.ignoreTargetSecurity, mRequest.componentSpecified,
                        mRequest.outActivity, mRequest.inTask, mRequest.reason,
                        mRequest.allowPendingRemoteAnimationRegistryLookup,
                        mRequest.originatingPendingIntent);
            }
        } finally {
            onExecutionComplete();
        }
    }


```

execute方法根据mRequest.mayWait是否为true，来决定是调用startActivityMayWait方法还是调用startActivity方法

startActivityMayWait方法最终也会调用startActivity方法
```
    private int startActivity(IApplicationThread caller, Intent intent, Intent ephemeralIntent,
            String resolvedType, ActivityInfo aInfo, ResolveInfo rInfo,
            IVoiceInteractionSession voiceSession, IVoiceInteractor voiceInteractor,
            IBinder resultTo, String resultWho, int requestCode, int callingPid, int callingUid,
            String callingPackage, int realCallingPid, int realCallingUid, int startFlags,
            SafeActivityOptions options, boolean ignoreTargetSecurity, boolean componentSpecified,
            ActivityRecord[] outActivity, TaskRecord inTask, String reason,
            boolean allowPendingRemoteAnimationRegistryLookup,
            PendingIntentRecord originatingPendingIntent) {
        //启动理由不能为null
        if (TextUtils.isEmpty(reason)) {
            throw new IllegalArgumentException("Need to specify a reason.");
        }
        mLastStartReason = reason;
        mLastStartActivityTimeMs = System.currentTimeMillis();
        mLastStartActivityRecord[0] = null;
       //调用重载方法startActivity
        mLastStartActivityResult = startActivity(caller, intent, ephemeralIntent, resolvedType,
                aInfo, rInfo, voiceSession, voiceInteractor, resultTo, resultWho, requestCode,
                callingPid, callingUid, callingPackage, realCallingPid, realCallingUid, startFlags,
                options, ignoreTargetSecurity, componentSpecified, mLastStartActivityRecord,
                inTask, allowPendingRemoteAnimationRegistryLookup, originatingPendingIntent);

        if (outActivity != null) {
            // mLastStartActivityRecord[0] is set in the call to startActivity above.
            outActivity[0] = mLastStartActivityRecord[0];
        }

        return getExternalResult(mLastStartActivityResult);
    }


```
接下来看调用的重载方法startActivity方法
```
    private int startActivity(IApplicationThread caller, Intent intent, Intent ephemeralIntent,
            String resolvedType, ActivityInfo aInfo, ResolveInfo rInfo,
            IVoiceInteractionSession voiceSession, IVoiceInteractor voiceInteractor,
            IBinder resultTo, String resultWho, int requestCode, int callingPid, int callingUid,
            String callingPackage, int realCallingPid, int realCallingUid, int startFlags,
            SafeActivityOptions options,
            boolean ignoreTargetSecurity, boolean componentSpecified, ActivityRecord[] outActivity,
            TaskRecord inTask, boolean allowPendingRemoteAnimationRegistryLookup,
            PendingIntentRecord originatingPendingIntent) {
        int err = ActivityManager.START_SUCCESS;
        // Pull the optional Ephemeral Installer-only bundle out of the options early.
        final Bundle verificationBundle
                = options != null ? options.popAppVerificationBundle() : null;

        ProcessRecord callerApp = null;
        if (caller != null) {
            //得到Launcher进程
            callerApp = mService.getRecordForAppLocked(caller);
            if (callerApp != null) {
                //获取Launcher进程的pid和uid
                callingPid = callerApp.pid;
                callingUid = callerApp.info.uid;
            } else {
                Slog.w(TAG, "Unable to find app for caller " + caller
                        + " (pid=" + callingPid + ") when starting: "
                        + intent.toString());
                err = ActivityManager.START_PERMISSION_DENIED;
            }
        }

        ...
        
        //创建即将要启动的Activity的描述类ActivityRecord
        ActivityRecord r = new ActivityRecord(mService, callerApp, callingPid, callingUid,
                callingPackage, intent, resolvedType, aInfo, mService.getGlobalConfiguration(),
                resultRecord, resultWho, requestCode, componentSpecified, voiceSession != null,
                mSupervisor, checkedOptions, sourceRecord);
        //把ActivityRecord存入outActivity
        if (outActivity != null) {
            outActivity[0] = r;
        }

        ...
        
        //调用startActivity
        return startActivity(r, sourceRecord, voiceSession, voiceInteractor, startFlags,
                true /* doResume */, checkedOptions, inTask, outActivity);
    }


```

又调用了重载方法startActivity

```
    private int startActivity(final ActivityRecord r, ActivityRecord sourceRecord,
                IVoiceInteractionSession voiceSession, IVoiceInteractor voiceInteractor,
                int startFlags, boolean doResume, ActivityOptions options, TaskRecord inTask,
                ActivityRecord[] outActivity) {
        int result = START_CANCELED;
        try {
            mService.mWindowManager.deferSurfaceLayout();
            result = startActivityUnchecked(r, sourceRecord, voiceSession, voiceInteractor,
                    startFlags, doResume, options, inTask, outActivity);
        } finally {
            
            final ActivityStack stack = mStartActivity.getStack();
            if (!ActivityManager.isStartResultSuccessful(result) && stack != null) {
                stack.finishActivityLocked(mStartActivity, RESULT_CANCELED,
                        null /* intentResultData */, "startActivity", true /* oomAdj */);
            }
            mService.mWindowManager.continueSurfaceLayout();
        }

        postStartActivityProcessing(r, result, mTargetStack);

        return result;
    }


```

调用了startActivityUnchecked方法，此方法只会在startActivity方法中调用
```
    private int startActivityUnchecked(final ActivityRecord r, ActivityRecord sourceRecord,
            IVoiceInteractionSession voiceSession, IVoiceInteractor voiceInteractor,
            int startFlags, boolean doResume, ActivityOptions options, TaskRecord inTask,
            ActivityRecord[] outActivity) {

        
        ...
        
        int result = START_SUCCESS;
        if (mStartActivity.resultTo == null && mInTask == null && !mAddingToTask
                && (mLaunchFlags & FLAG_ACTIVITY_NEW_TASK) != 0) {
            newTask = true;
            //创建一个新的Activity栈
            result = setTaskFromReuseOrCreateNewTask(taskToAffiliate, topStack);
        } else if (mSourceRecord != null) {
            result = setTaskFromSourceRecord();
        } else if (mInTask != null) {
            result = setTaskFromInTask();
        } else {
           
            //没有任务栈或者从已经存在的Activity跳转时，创建一个新的任务栈或者直接放入当前任务栈栈顶
            setTaskToCurrentTopOrCreateNewTask();
        }
        if (result != START_SUCCESS) {
            return result;
        }

        ...
        
        if (mDoResume) {
            //拿到当前在栈顶的Activity
            final ActivityRecord topTaskActivity =
                    mStartActivity.getTask().topRunningActivityLocked();
              //如果有则判断和当前启动Activity是否相同
            if (!mTargetStack.isFocusable()
                    || (topTaskActivity != null && topTaskActivity.mTaskOverlay
                    && mStartActivity != topTaskActivity)) {
                
                mTargetStack.ensureActivitiesVisibleLocked(null, 0, !PRESERVE_WINDOWS);
               
                mService.mWindowManager.executeAppTransition();
            } else {
                //第一次启动的Activity肯定会执行下面的逻辑
                if (mTargetStack.isFocusable() && !mSupervisor.isFocusedStack(mTargetStack)) {
                    mTargetStack.moveToFront("startActivityUnchecked");
                }
                mSupervisor.resumeFocusedStackTopActivityLocked(mTargetStack, mStartActivity,
                        mOptions);
            }
        } else if (mStartActivity != null) {
            mSupervisor.mRecentTasks.add(mStartActivity.getTask());
        }
        mSupervisor.updateUserStackLocked(mStartActivity.userId, mTargetStack);

        mSupervisor.handleNonResizableTaskIfNeeded(mStartActivity.getTask(), preferredWindowingMode,
                preferredLaunchDisplayId, mTargetStack);

        return START_SUCCESS;
    }


```

startActivityUnchecked主要是进行Activity任务栈的切换，根据intent的flag参数来判断，是新创建任务栈还是在已有任务栈内执行操作；如果没有设置flag参数，则会执行默认的setTaskToCurrentTopOrCreateNewTask方法，先检查是否需要创建任务栈，然后把Activity移到栈顶

第一次启动的Activity会执行resumeFocusedStackTopActivityLocked方法

```
    boolean resumeFocusedStackTopActivityLocked(
            ActivityStack targetStack, ActivityRecord target, ActivityOptions targetOptions) {

        if (!readyToResume()) {
            return false;
        }

        if (targetStack != null && isFocusedStack(targetStack)) {
            return targetStack.resumeTopActivityUncheckedLocked(target, targetOptions);
        }
        //获取要启动的Activity所在栈顶的不是处于停止状态的ActivitRecord
        final ActivityRecord r = mFocusedStack.topRunningActivityLocked();
        //如果栈顶没有处于停止状态的Activity，则判断它是否处于resumed状态
        if (r == null || !r.isState(RESUMED)) {
            mFocusedStack.resumeTopActivityUncheckedLocked(null, null);
        } else if (r.isState(RESUMED)) {
            // Kick off any lingering app transitions form the MoveTaskToFront operation.
            mFocusedStack.executeAppTransition(targetOptions);
        }

        return false;
    }


```

根Activity启动时r肯定为null所以会执行resumeTopActivityUncheckedLocked方法

```
@GuardedBy("mService")
    boolean resumeTopActivityUncheckedLocked(ActivityRecord prev, ActivityOptions options) {
        if (mStackSupervisor.inResumeTopActivity) {
            // Don't even start recursing.
            return false;
        }

        boolean result = false;
        try {
           
            mStackSupervisor.inResumeTopActivity = true;
            result = resumeTopActivityInnerLocked(prev, options);

            
            final ActivityRecord next = topRunningActivityLocked(true /* focusableOnly */);
            if (next == null || !next.canTurnScreenOn()) {
                checkReadyForSleep();
            }
        } finally {
            mStackSupervisor.inResumeTopActivity = false;
        }

        return result;
    }


```
接着看resumeTopActivityInnerLocked方法
```
  mStackSupervisor.startSpecificActivityLocked(next, true, true);

```

resumeTopActivityInnerLocked方法内容很多，我们只需要关注最后调用的 mStackSupervisor.startSpecificActivityLocked(next, true, true)方法就可以


```
    void startSpecificActivityLocked(ActivityRecord r,
            boolean andResume, boolean checkConfig) {
        // Is this activity's application already running?
        //判断应用程序进程是否运行
        //获取应用程序进程
        ProcessRecord app = mService.getProcessRecordLocked(r.processName,
                r.info.applicationInfo.uid, true);

        if (app != null && app.thread != null) {
            try {
                if ((r.info.flags&ActivityInfo.FLAG_MULTIPROCESS) == 0
                        || !"android".equals(r.info.packageName)) {
                  
                    //添加package
                    app.addPackage(r.info.packageName, r.info.applicationInfo.longVersionCode,
                            mService.mProcessStats);
                }
                //在应用程序进程中启动Activity
                realStartActivityLocked(r, app, andResume, checkConfig);
                return;
            } catch (RemoteException e) {
                Slog.w(TAG, "Exception when starting activity "
                        + r.intent.getComponent().flattenToShortString(), e);
            }

        }

        mService.startProcessLocked(r.processName, r.info.applicationInfo, true, 0,
                "activity", r.intent.getComponent(), false, false, true);
    }


```

继续看realStartActivityLocked方法，内容较多只看关键部分

```
  final boolean realStartActivityLocked(ActivityRecord r, ProcessRecord app,
            boolean andResume, boolean checkConfig) throws RemoteException {
     ....
                //创建activity启动事务
                final ClientTransaction clientTransaction = ClientTransaction.obtain(app.thread,
                        r.appToken);
                //把要启动的Activity的信息封装到LaunchActivityItem中
                clientTransaction.addCallback(LaunchActivityItem.obtain(new Intent(r.intent),
                        System.identityHashCode(r), r.info,
                        mergedConfiguration.getGlobalConfiguration(),
                        mergedConfiguration.getOverrideConfiguration(), r.compat,
                        r.launchedFromPackage, task.voiceInteractor, app.repProcState, r.icicle,
                        r.persistentState, results, newIntents, mService.isNextTransitionForward(),
                        profilerInfo));

                ...
                
                mService.getLifecycleManager().scheduleTransaction(clientTransaction);
    ...
    }

```

在9.0系统中把要启动的Activity的相关信息封装在LaunchActivityItem中，然后由ClientTransaction统一管理。

通过ClientTransaction的addCallback方法把LaunchActivityItem存入clientTransaction中的mActivityCallbacks集合中

最后通过AMS获取ClientLifecycleManager类后执行其scheduleTransaction方法

```
    void scheduleTransaction(ClientTransaction transaction) throws RemoteException {
        final IApplicationThread client = transaction.getClient();
        
        transaction.schedule();
        if (!(client instanceof Binder)) { //当不是ApplicationThread的Binder引用时，则可以安全的清空事物
            transaction.recycle();
        }
    }

```
执行了ClientTransaction的schedule方法
```

  public void schedule() throws RemoteException {
        //执行IApplicationThread的scheduleTransaction方法，把当前的ClientTransaction发送到应用程序进程
        mClient.scheduleTransaction(this);
    }


```
mClient的类型是IApplicationThread，这一步是Binder通信。经过这一步ClientTransaction就从SystemServer进程的AMS中发送到了应用程序进程中的ApplicationThread中


## ActivityThread启动Activity的过程

通过Binder发送消息到应用程序进程后，会执行ActivityThread类的内部类ApplicationThread中的scheduleTransaction方法。ApplicationThread类继承了IApplicationThread.Stub方法，用于实现Binder通信

```
  @Override
        public void scheduleTransaction(ClientTransaction transaction) throws RemoteException {
            ActivityThread.this.scheduleTransaction(transaction);
        }

```
可以看到scheduleTransaction方法执行了ActivityThread.this.scheduleTransaction(transaction)方法，位于ClientTransactionHandler抽象类内，而ActivityThread继承了该抽象类

```
 void scheduleTransaction(ClientTransaction transaction) {
        //遍历所有ClientTransactionItem,并调用其preExecute方法准备数据
        transaction.preExecute(this);
        //通过ActivityThread类中的Hnadler把消息从ApplicationThread发送到ActivityThread中
        sendMessage(ActivityThread.H.EXECUTE_TRANSACTION, transaction);
    }

```
调用sendMessage方法后会把transaction发送的H的handleMessage中,这时会进入case EXECUTE_TRANSACTION中
```
case EXECUTE_TRANSACTION:
    final ClientTransaction transaction = (ClientTransaction) msg.obj;
    mTransactionExecutor.execute(transaction);
    if (isSystem()) {
        transaction.recycle();
        }
    break;


```
执行TransactionExecutor的execute方法

```
    public void execute(ClientTransaction transaction) {
        final IBinder token = transaction.getActivityToken();
        log("Start resolving transaction for client: " + mTransactionHandler + ", token: " + token);
        
        executeCallbacks(transaction);
        
        executeLifecycleState(transaction);
        mPendingActions.clear();
        log("End resolving transaction");
    }


```

execute方法中会执行ClientTransaction中的保存的所有callback方法

```
public void executeCallbacks(ClientTransaction transaction) {
        final List<ClientTransactionItem> callbacks = transaction.getCallbacks();
        
        ...
        
        final int size = callbacks.size();
        for (int i = 0; i < size; ++i) {
           ...
            if (closestPreExecutionState != UNDEFINED) {
                //执行Activity的生命周期
                cycleToPath(r, closestPreExecutionState);
            }

           ...
        }
    }

```

executeCallbacks中主要方法就是执行cycleToPath


```
  private void cycleToPath(ActivityClientRecord r, int finish,
            boolean excludeLastState) {
        //获取当前activity的生命周期状态
        final int start = r.getLifecycleState();
        log("Cycle from: " + start + " to: " + finish + " excludeLastState:" + excludeLastState);
        final IntArray path = mHelper.getLifecyclePath(start, finish, excludeLastState);
        //根据path执行相应的activity生命周期
        performLifecycleSequence(r, path);
    }


```

最终会执行到performLifecycleSequence方法中
```
 private void performLifecycleSequence(ActivityClientRecord r, IntArray path) {
        final int size = path.size();
        for (int i = 0, state; i < size; i++) {
            state = path.get(i);
            log("Transitioning to state: " + state);
            switch (state) {
                case ON_CREATE:
                    //创建Activity
                    mTransactionHandler.handleLaunchActivity(r, mPendingActions,
                            null /* customIntent */);
                    break;
                case ON_START:
                    mTransactionHandler.handleStartActivity(r, mPendingActions);
                    break;
                case ON_RESUME:
                    mTransactionHandler.handleResumeActivity(r.token, false /* finalStateRequest */,
                            r.isForward, "LIFECYCLER_RESUME_ACTIVITY");
                    break;
                case ON_PAUSE:
                    mTransactionHandler.handlePauseActivity(r.token, false /* finished */,
                            false /* userLeaving */, 0 /* configChanges */, mPendingActions,
                            "LIFECYCLER_PAUSE_ACTIVITY");
                    break;
                case ON_STOP:
                    mTransactionHandler.handleStopActivity(r.token, false /* show */,
                            0 /* configChanges */, mPendingActions, false /* finalStateRequest */,
                            "LIFECYCLER_STOP_ACTIVITY");
                    break;
                case ON_DESTROY:
                    mTransactionHandler.handleDestroyActivity(r.token, false /* finishing */,
                            0 /* configChanges */, false /* getNonConfigInstance */,
                            "performLifecycleSequence. cycling to:" + path.get(size - 1));
                    break;
                case ON_RESTART:
                    mTransactionHandler.performRestartActivity(r.token, false /* start */);
                    break;
                default:
                    throw new IllegalArgumentException("Unexpected lifecycle state: " + state);
            }
        }
    }


```

performLifecycleSequence就是根据不同的state来执行activity的生命周期，启动Activity会执行case ON_CREATE分支中，然后执行handleLaunchActivity方法创建Activity

```
 public Activity handleLaunchActivity(ActivityClientRecord r,
            PendingTransactionActions pendingActions, Intent customIntent) {
        ...
       
        // Initialize before creating the activity
        //创建activity前初始化GraphicsEnvironment
        if (!ThreadedRenderer.sRendererDisabled) {
            GraphicsEnvironment.earlyInitEGL();
        }
        //初始化WMS
        WindowManagerGlobal.initialize();
        //启动Activity
        final Activity a = performLaunchActivity(r, customIntent);

        ...

        return a;
    }


```

handleLaunchActivity方法会调用performLaunchActivity方法来启动Activity


```
  private Activity performLaunchActivity(ActivityClientRecord r, Intent customIntent) {
        //获取ActivityInfo类
        ActivityInfo aInfo = r.activityInfo;
        if (r.packageInfo == null) {
            //获取APK文件的描述类LoadedApk
            r.packageInfo = getPackageInfo(aInfo.applicationInfo, r.compatInfo,
                    Context.CONTEXT_INCLUDE_CODE);
        }

        ComponentName component = r.intent.getComponent();
        if (component == null) {
            component = r.intent.resolveActivity(
                mInitialApplication.getPackageManager());
            r.intent.setComponent(component);
        }

        if (r.activityInfo.targetActivity != null) {
            component = new ComponentName(r.activityInfo.packageName,
                    r.activityInfo.targetActivity);
        }
        //创建要启动Activity的上下文环境
        ContextImpl appContext = createBaseContextForActivity(r);
        Activity activity = null;
        try {
            java.lang.ClassLoader cl = appContext.getClassLoader();
            //使用类加载器创建activity实例
            activity = mInstrumentation.newActivity(
                    cl, component.getClassName(), r.intent);
            StrictMode.incrementExpectedActivityCount(activity.getClass());
            r.intent.setExtrasClassLoader(cl);
            r.intent.prepareToEnterProcess();
            if (r.state != null) {
                r.state.setClassLoader(cl);
            }
        } catch (Exception e) {
            if (!mInstrumentation.onException(activity, e)) {
                throw new RuntimeException(
                    "Unable to instantiate activity " + component
                    + ": " + e.toString(), e);
            }
        }

        try {
            //创建Application
            Application app = r.packageInfo.makeApplication(false, mInstrumentation);

            if (localLOGV) Slog.v(TAG, "Performing launch of " + r);
            if (localLOGV) Slog.v(
                    TAG, r + ": app=" + app
                    + ", appName=" + app.getPackageName()
                    + ", pkg=" + r.packageInfo.getPackageName()
                    + ", comp=" + r.intent.getComponent().toShortString()
                    + ", dir=" + r.packageInfo.getAppDir());

            if (activity != null) {
                CharSequence title = r.activityInfo.loadLabel(appContext.getPackageManager());
                Configuration config = new Configuration(mCompatConfiguration);
                if (r.overrideConfig != null) {
                    config.updateFrom(r.overrideConfig);
                }
                if (DEBUG_CONFIGURATION) Slog.v(TAG, "Launching activity "
                        + r.activityInfo.name + " with config " + config);
                Window window = null;
                if (r.mPendingRemoveWindow != null && r.mPreserveWindow) {
                    window = r.mPendingRemoveWindow;
                    r.mPendingRemoveWindow = null;
                    r.mPendingRemoveWindowManager = null;
                }
                appContext.setOuterContext(activity);
                //初始化Activity，这里会创建Window对象，并与activity关联
                activity.attach(appContext, this, getInstrumentation(), r.token,
                        r.ident, app, r.intent, r.activityInfo, title, r.parent,
                        r.embeddedID, r.lastNonConfigurationInstances, config,
                        r.referrer, r.voiceInteractor, window, r.configCallback);

                if (customIntent != null) {
                    activity.mIntent = customIntent;
                }
                r.lastNonConfigurationInstances = null;
                checkAndBlockForNetworkAccess();
                activity.mStartedActivity = false;
                int theme = r.activityInfo.getThemeResource();
                //设置主题
                if (theme != 0) {
                    activity.setTheme(theme);
                }

                activity.mCalled = false;
                if (r.isPersistable()) {
                    //启动activity
                    mInstrumentation.callActivityOnCreate(activity, r.state, r.persistentState);
                } else {
                    mInstrumentation.callActivityOnCreate(activity, r.state);
                }
                if (!activity.mCalled) {
                    throw new SuperNotCalledException(
                        "Activity " + r.intent.getComponent().toShortString() +
                        " did not call through to super.onCreate()");
                }
                r.activity = activity;
            }
            r.setState(ON_CREATE);

            mActivities.put(r.token, r);

        } catch (SuperNotCalledException e) {
            throw e;

        } catch (Exception e) {
            if (!mInstrumentation.onException(activity, e)) {
                throw new RuntimeException(
                    "Unable to start activity " + component
                    + ": " + e.toString(), e);
            }
        }

        return activity;
    }
```

最后会调用callActivityOnCreate方法来启动activity

```
  public void callActivityOnCreate(Activity activity, Bundle icicle,
            PersistableBundle persistentState) {
        prePerformCreate(activity);
        activity.performCreate(icicle, persistentState);
        postPerformCreate(activity);
    }

```

这时activity已经启动然后会调用activity.performCreate方法执行生命周期的onCreate方法

```
  @UnsupportedAppUsage
    final void performCreate(Bundle icicle, PersistableBundle persistentState) {
        mCanEnterPictureInPicture = true;
        restoreHasCurrentPermissionRequest(icicle);
        //onCreate方法
        if (persistentState != null) {
            onCreate(icicle, persistentState);
        } else {
            onCreate(icicle);
        }
        writeEventLog(LOG_AM_ON_CREATE_CALLED, "performCreate");
        mActivityTransitionState.readState(icicle);

        mVisibleFromClient = !mWindow.getWindowStyle().getBoolean(
                com.android.internal.R.styleable.Window_windowNoDisplay, false);
        mFragments.dispatchActivityCreated();
        mActivityTransitionState.setEnterActivityOptions(this, getActivityOptions());
    }

```

到这里根Activity就已经启动了，生命周期方法onCreate也执行了

## 根activity启动涉及到的进程
主要有zygote进程、SystemServer进程、应用程序进程和Launcher进程
### 调用顺序
- 点击Launcher进程中的应用图标
- Launcher向AMS请求创建根activity，通过Binder与AMS通信
- AMS判断该activity对应的应用程序进程是否启动
- AMS向Zygote请求创建应用程序进程，通过Socket与Zygote通信
- Zygote会fork自身创建子进程并根据命令找到ActivityThread类并执行main方法
- main方法中会调用IActivityManager的attachApplication方法来通知AMS应用程序进程已经创建完成
- AMS会把要启动的Activity信息发送给ActivityThread中的ApplicationThread中,这是Binder通信
- ApplicationThread会执行scheduleTransaction方法，并通过Activity生命周期辅助类ClientTransactionHandler和TransactionExecutor类，来执行启动Activity和管理其生命周期

