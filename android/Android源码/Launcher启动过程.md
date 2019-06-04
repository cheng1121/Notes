# Launcher启动过程
Launcher的作用主要有两点：
- 作为Android系统的启动器，用于启动应用程序
- 作为Android系统的桌面，用于显示和管理应用程序的快捷图标或者其他桌面组件


## Launcher的入口
Launcher的入口为AMS的systemReady方法，在SystemServer的startOtherServices方法中被调用
```

...
 mActivityManagerService.systemReady(() -> {
        Slog.i(TAG, "Making services ready");
        traceBeginAndSlog("StartActivityManagerReadyPhase");
        mSystemServiceManager.startBootPhase(
                SystemService.PHASE_ACTIVITY_MANAGER_READY);
                    
    ...
}
...
```
现在看下systemReady方法内部做了什么
```
public void systemReady(final Runnable goingCallback, TimingsTraceLog traceLog){
...
synchronized(this){
...
 mStackSupervisor.resumeFocusedStackTopActivityLocked();
            mUserController.sendUserSwitchBroadcasts(-1, currentUserId);
            
...
    }
}
```
systemReady方法中调用了ActivityStackSupervisor的resumeFocusedStackTopActivityLocked方法：
```
    boolean resumeFocusedStackTopActivityLocked(
            ActivityStack targetStack, ActivityRecord target, ActivityOptions targetOptions) {

        if (!readyToResume()) {
            return false;
        }

        if (targetStack != null && isFocusedStack(targetStack)) {
           //1
            return targetStack.resumeTopActivityUncheckedLocked(target, targetOptions); 
        }

        final ActivityRecord r = mFocusedStack.topRunningActivityLocked();
        if (r == null || !r.isState(RESUMED)) {
            mFocusedStack.resumeTopActivityUncheckedLocked(null, null);
        } else if (r.isState(RESUMED)) {
            // Kick off any lingering app transitions form the MoveTaskToFront operation.
            mFocusedStack.executeAppTransition(targetOptions);
        }

        return false;
    }

```
注释1处调用了ActivityStack的resumeTopActivityUncheckedLocked方法，ActivityStack对象是用来描述Activity堆栈的。
```
@GuardedBy("mService")
    boolean resumeTopActivityUncheckedLocked(ActivityRecord prev, ActivityOptions options) {
        if (mStackSupervisor.inResumeTopActivity) {
            // Don't even start recursing.
            return false;
        }

        boolean result = false;
        try {
            // Protect against recursion.
            mStackSupervisor.inResumeTopActivity = true;
            result = resumeTopActivityInnerLocked(prev, options); //1

            
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
注释1处调用了resumeTopActivityInnerLocked方法
```
private boolean resumeTopActivityInnerLocked(ActivityRecord prev, ActivityOptions options) {
     ...
     
     if (!hasRunningActivity) {
            //内部会调用mStackSupervisor.resumeHomeStackTask方法
            return resumeTopActivityInNextFocusableStack(prev, options, "noMoreActivities");
    }
        
    ....
        
}
```
这个方法中的代码很长，这里只截取对我们有用的部分

```
  private boolean resumeTopActivityInNextFocusableStack(ActivityRecord prev,
            ActivityOptions options, String reason) {
        if (adjustFocusToNextFocusableStack(reason)) {
           
            return mStackSupervisor.resumeFocusedStackTopActivityLocked(
                    mStackSupervisor.getFocusedStack(), prev, null);
        }

        // Let's just start up the Launcher...
        ActivityOptions.abort(options);
        if (DEBUG_STATES) Slog.d(TAG_STATES,
                "resumeTopActivityInNextFocusableStack: " + reason + ", go home");
        if (DEBUG_STACK) mStackSupervisor.validateTopActivitiesLocked();
        // Only resume home if on home display
        return isOnHomeDisplay() &&
                mStackSupervisor.resumeHomeStackTask(prev, reason);
    }
```

接下来继续看resumeHomeStackTask方法
```
    boolean resumeHomeStackTask(ActivityRecord prev, String reason) {
        if (!mService.mBooting && !mService.mBooted) {
            // Not ready yet!
            return false;
        }

        mHomeStack.moveHomeStackTaskToTop();
        ActivityRecord r = getHomeActivity();
        final String myReason = reason + " resumeHomeStackTask";

        // Only resume home activity if isn't finishing.
        if (r != null && !r.finishing) {
            moveFocusableActivityStackToFrontLocked(r, myReason);
            return resumeFocusedStackTopActivityLocked(mHomeStack, prev, null);
        }
        return mService.startHomeActivityLocked(mCurrentUser, myReason);
    }

```
此方法最后调用了AMS中的startHomeActivityLocked方法
```
      boolean startHomeActivityLocked(int userId, String reason) {
        //判断工厂模式和mTopAction的值，符合要求就继续执行下去
        if (mFactoryTest == FactoryTest.FACTORY_TEST_LOW_LEVEL
                && mTopAction == null) {
            return false;
        }
        //创建Launcher启动所需的Intent
        Intent intent = getHomeIntent();
        ActivityInfo aInfo = resolveActivityInfo(intent, STOCK_PM_FLAGS, userId);
        if (aInfo != null) {
            intent.setComponent(new ComponentName(aInfo.applicationInfo.packageName, aInfo.name));
            // Don't do this if the home app is currently being
            // instrumented.
            aInfo = new ActivityInfo(aInfo);
            aInfo.applicationInfo = getAppInfoForUser(aInfo.applicationInfo, userId);
            ProcessRecord app = getProcessRecordLocked(aInfo.processName,
                    aInfo.applicationInfo.uid, true);
            //判断Launcher是否已运行
            if (app == null || app.instr == null) {
                intent.setFlags(intent.getFlags() | FLAG_ACTIVITY_NEW_TASK);
                final int resolvedUserId = UserHandle.getUserId(aInfo.applicationInfo.uid);
                // For ANR debugging to verify if the user activity is the one that actually
                // launched.
                final String myReason = reason + ":" + userId + ":" + resolvedUserId;
                //启动Launcher
                mActivityStartController.startHomeActivity(intent, aInfo, myReason);
            }
        } else {
            Slog.wtf(TAG, "No home screen found for " + intent, new Throwable());
        }

        return true;
    }

```

- mFactoryTest代表系统的运行模式，系统的运行模式分为三种，分别是非工厂模式、低级工厂模式和高级工厂模式，mTopAction则用来描述第一个被启动的Activity组件的Action，默认值为Inten.ACTION_MAIN
- getHomeIntent创建Intent对象
- app == null || app.instr == null判断Launcher是否已运行
- mActivityStartController.startHomeActivity启动Launcher

接着看下startHomeActivity方法
```
    void startHomeActivity(Intent intent, ActivityInfo aInfo, String reason) {
        mSupervisor.moveHomeStackTaskToTop(reason);

        mLastHomeActivityStartResult = obtainStarter(intent, "startHomeActivity: " + reason)
                .setOutActivity(tmpOutRecord)
                .setCallingUid(0)
                .setActivityInfo(aInfo)
                .execute();
        mLastHomeActivityStartRecord = tmpOutRecord[0];
        if (mSupervisor.inResumeTopActivity) {
            // If we are in resume section already, home activity will be initialized, but not
            // resumed (to avoid recursive resume) and will stay that way until something pokes it
            // again. We need to schedule another resume.
            mSupervisor.scheduleResumeTopActivities();
        }
    }

```
首先把Launcher放入HomeStack中，HomeStack是在ActivityStackSupervisor中定义的用于存储Launcher的变量。接着调用obtainStarter来启动Launcher

Launcher实际上就是一个应用程序，只不过是在系统启动后，也就是AMS启动后就会直接创建，主要用于展示其他应用的图标、时间等信息。

所以obtainStarter方法启动Launcher就是启动Activity的过程
