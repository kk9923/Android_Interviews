## Activity启动流程

在Android中有两种操作会引发Activity的启动，一种用户点击Launcher的应用程序图标时，Launcher会为启动应用程序的主Activity。另外一种是在已经起来的Activity内部通过调用startActvity接口启动新的Activity。每一个Activity都可以在内部启动新的Activity。

下面先分析从Activity内部启动新的Activity。

**开始源码分析：Activity#startActivity()**

```java
    @Override
    public void startActivity(Intent intent) {
        this.startActivity(intent, null);
    }
    
    public void startActivity(Intent intent, @Nullable Bundle options) {
        if (options != null) {
            startActivityForResult(intent, -1, options);
        } else {
            // Note we want to go through this call for compatibility with
            // applications that may have overridden the method.
            startActivityForResult(intent, -1);
        }
    }

    public void startActivityForResult(Intent intent, int requestCode) {
        startActivityForResult(intent, requestCode, null);
    }
    
```
从上面的代码中我们发现：调用startActivty实际上最后还是调用了startActivityForResult 方法。那我们接着看startActivityForResult()方法：

**Activity#startActivityForResult**

```java
    public void startActivityForResult(Intent intent, int requestCode, @Nullable Bundle options) {
    //一般的Activity的mParent为null
        if (mParent == null) {
          //真正执行启动activity的代码逻辑
            Instrumentation.ActivityResult ar =
                mInstrumentation.execStartActivity(
                    this, mMainThread.getApplicationThread(), mToken, this,
                    intent, requestCode, options);
            // 若result非空，发送结果给本activity，即onActivityResult会被调用。
            if (ar != null) {
                mMainThread.sendActivityResult(
                    mToken, mEmbeddedID, requestCode, ar.getResultCode(),
                    ar.getResultData());
            }
            if (requestCode >= 0) {
         
                 mStartedActivity = true;
            }
    
            cancelInputsAndStartExitTransition(options);
            // TODO Consider clearing/flushing other event sources and events for child windows.
        } else {
            if (options != null) {
                mParent.startActivityFromChild(this, intent, requestCode, options);
            } else {
                // Note we want to go through this method for compatibility with
                // existing applications that may have overridden it.
                mParent.startActivityFromChild(this, intent, requestCode);
            }
        }
    }

```
>  这里的mInstrumentation是Activity类的成员变量，它的类型是Intrumentation，定义在frameworks/base/core/java/android/app/Instrumentation.java文件中，它用来监控应用程序和系统的交互。

从上面的代码中发现，实际上调用的启动activity方法的还是Instrumentation类,在之后会调用mMainThread.sendActivityResult方法，**若result非空，发送结果给本activity，即onActivityResult会被调用** 。

接着看Instrumentation#execStartActivity代码调用

**Instrumentation#execStartActivity**

```java
  public ActivityResult execStartActivity(
            Context who, IBinder contextThread, IBinder token, Activity target,
            Intent intent, int requestCode, Bundle options) {
        IApplicationThread whoThread = (IApplicationThread) contextThread;
        Uri referrer = target != null ? target.onProvideReferrer() : null;
        if (referrer != null) {
            intent.putExtra(Intent.EXTRA_REFERRER, referrer);
        }
        if (mActivityMonitors != null) {
            synchronized (mSync) {
                final int N = mActivityMonitors.size();
                for (int i=0; i<N; i++) {
                    final ActivityMonitor am = mActivityMonitors.get(i);
                    ActivityResult result = null;
                    if (am.ignoreMatchingSpecificIntents()) {
                        result = am.onStartActivity(intent);
                    }
                    if (result != null) {
                        am.mHits++;
                        return result;
                    } else if (am.match(who, null, intent)) {
                        am.mHits++;
                        if (am.isBlocking()) {
                            return requestCode >= 0 ? am.getResult() : null;
                        }
                        break;
                    }
                }
            }
        }
        try {
            intent.migrateExtraStreamToClipData();
            intent.prepareToLeaveProcess(who);
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

### 参考链接

[https://www.cnblogs.com/Jackwen/p/5117313.html](https://www.cnblogs.com/Jackwen/p/5117313.html)

[https://blog.csdn.net/luoshengyang/article/details/6689748](https://blog.csdn.net/luoshengyang/article/details/6689748)

