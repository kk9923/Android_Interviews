
## View.post(Runnable)获取View的宽高

### View#post 基本使用

```java
        Button test = findViewById(R.id.test);
        test.post(new Runnable() {
            @Override
            public void run() {
                System.out.println("post  Runnable");
            }
        });
```

### post() 执行过程以及源码分析(API26)


```java
    public boolean post(Runnable action) {
        final AttachInfo attachInfo = mAttachInfo;
        if (attachInfo != null) {
            return attachInfo.mHandler.post(action);
        }

        // Postpone the runnable until we know on which thread it needs to run.
        // Assume that the runnable will be successfully placed after attach.
        getRunQueue().post(action);
        return true;
    }
```
> 将 Runnable 添加到执行队列中，其最终会在 UI 线程中执行.

View中的变量mAttachInfo 赋值的地方：

```java
    void dispatchAttachedToWindow(AttachInfo info, int visibility) {
        mAttachInfo = info;
        // Transfer all pending runnables. 转移所有待办任务
        if (mRunQueue != null) {
            mRunQueue.executeActions(info.mHandler);
            mRunQueue = null;
        }
        // 回调方法
        onAttachedToWindow();
    }
```
其中**View的dispatchAttachedToWindow()**方法是在**ViewRootImpl中的performTraversals()方法**中被调用,也就是说我们在Activity的onCreat()方法中去post(Runnable)时,mAttachInfo为空。接下来我们来看一看**getRunQueue().post(action)** 究竟做了什么.

**getRunQueue()代码**

```java
    /**
     * Returns the queue of runnable for this view.
     *
     * @return the queue of runnables for this view
     */
    private HandlerActionQueue getRunQueue() {
        if (mRunQueue == null) {
            mRunQueue = new HandlerActionQueue();
        }
        return mRunQueue;
    }
```
 > 返回当前 View 自身的一个 Runnable 队列。
 
**HandlerActionQueue代码**


```java
public class HandlerActionQueue {
    private HandlerAction[] mActions;
    private int mCount;

    public void post(Runnable action) {
        postDelayed(action, 0);
    }

    public void postDelayed(Runnable action, long delayMillis) {
        final HandlerAction handlerAction = new HandlerAction(action, delayMillis);

        synchronized (this) {
            if (mActions == null) {
                mActions = new HandlerAction[4];
            }
            mActions = GrowingArrayUtils.append(mActions, mCount, handlerAction);
            mCount++;
        }
    }
    ... 代码省略
    public void executeActions(Handler handler) {
        synchronized (this) {
            final HandlerAction[] actions = mActions;
            for (int i = 0, count = mCount; i < count; i++) {
                final HandlerAction handlerAction = actions[i];
                handler.postDelayed(handlerAction.action, handlerAction.delay);
            }

            mActions = null;
            mCount = 0;
        }
    }
    ... 代码省略
    private static class HandlerAction {
        final Runnable action;
        final long delay;

        public HandlerAction(Runnable action, long delay) {
            this.action = action;
            this.delay = delay;
        }

        public boolean matches(Runnable otherAction) {
            return otherAction == null && action == null
                    || action != null && action.equals(otherAction);
        }
    }
}
```
在getRunQueue().post(action)时根据我们传入的Runnable对象构造内部类HandlerAction ，并将其保存至HandlerAction[] mActions数组中。这里只是保存了我们post的Runnable，并没有做其他操作,所以其run方法不会立即执行。

**performTraversals()**

在View的绘制流程中很非常重要的就是这个方法。在该方法中完成了View的测量，布局，绘制等流程。

```java
private void performTraversals() {
    
    // 此处的 host 是根布局 DecorView，用递归的方式一层一层的调用 dispatchAttachedToWindow
  
    //  mAttachInfo 在 ViewRootImpl 的构造器中初始化的，其持有 ViewRootImpl 的 Handler 对象
    
    host.dispatchAttachedToWindow(mAttachInfo, 0);
    
    getRunQueue().executeActions(mAttachInfo.mHandler);
    
    // 绘制流程就从这里开始
    performMeasure();
    performLayout();
    performDraw();
}
```
**host对象分析**

在ActivityThread的handleResumeActivity()方法中

```java
    View decor = r.window.getDecorView();
    decor.setVisibility(View.INVISIBLE);
    ViewManager wm = a.getWindowManager();
    wm.addView(decor, l);
    
    //  wm 其实是 WindowManagerImpl
    
    @Override
    public void addView(@NonNull View view, @NonNull ViewGroup.LayoutParams params) {
        applyDefaultToken(params);
        mGlobal.addView(view, params, mContext.getDisplay(), mParentWindow);
    }
    
    private final WindowManagerGlobal mGlobal = WindowManagerGlobal.getInstance();
    
    ViewRootImpl root = new ViewRootImpl(view.getContext(), display);
     
    root.setView(view, wparams, panelParentView);
     
    if (mView == null) {
        mView = view;
    }
    
```
所以 **host.dispatchAttachedToWindow(mAttachInfo, 0)** 的host就是DecorView。

**View的dispatchAttachedToWindow分析**


```java
void dispatchAttachedToWindow(AttachInfo info, int visibility) {
    mAttachInfo = info;
    // Transfer all pending runnables. 
    if (mRunQueue != null) {
        mRunQueue.executeActions(info.mHandler);
        mRunQueue = null;
    }
    // 回调方法
    onAttachedToWindow();
}

```
**分析：**
 - 第一个参数 AttachInfo 就是ViewRootImpl中初始化的AttachInfo，它持有当前 ViewRootImpl 的 Handler对象引用,当调用**mRunQueue.executeActions(info.mHandler)** 时，通过该Handler将Runnable封装至Message对象的callback变量。再将该Message对象添加至消息队列。因为**这里的Handler为 ViewRootImpl中的mHandler引用**,因此所有的消息都添加到了主线程的MessageQueue中。
 
```java
    public void executeActions(Handler handler) {
        synchronized (this) {
            final HandlerAction[] actions = mActions;
            for (int i = 0, count = mCount; i < count; i++) {
                final HandlerAction handlerAction = actions[i];
                handler.postDelayed(handlerAction.action, handlerAction.delay);
            }

            mActions = null;
            mCount = 0;
        }
    }
```

在mRunQueue.executeActions(info.mHandler)方法执行完毕后将mRunQueue置空，节约资源。然后回调View的 onAttachedToWindow()方法。


host.dispatchAttachedToWindow()分析完后，接着网下看:
```java
  getRunQueue().executeActions(mAttachInfo.mHandler);
```
ViewRootImpl.getRunQueue()又是啥？

```java
    static HandlerActionQueue getRunQueue() {
        HandlerActionQueue rq = sRunQueues.get();
        if (rq != null) {
            return rq;
        }
        rq = new HandlerActionQueue();
        sRunQueues.set(rq);
        return rq;
    }
    
    static final ThreadLocal<HandlerActionQueue> sRunQueues = new ThreadLocal<HandlerActionQueue>();

```

通过ThreadLocal获取主线程保存的HandlerActionQueue队列,且是唯一的。这里和View中的getRunQueue() 方法不同,View中的mRunQueue在dispatchAttachedToWindow()方法调用后就会变为null。

```java
    private HandlerActionQueue getRunQueue() {
    if (mRunQueue == null) {
        mRunQueue = new HandlerActionQueue();
    }
    return mRunQueue;
}

```
初始化主线程中的HandlerActionQueue，并将其队列中待执行的任务通过Handler全部发送到主线程的消息队列。

**小结：**
 - 在当前 View attach 到 Window 之前，会自己先维护一个 HandlerActionQueue 对象，用来存储当前的 Runnable 对象，然后等到 Attach 到 Window 的时候 (也就是 ViewRootImpl 执行到 performTraversal 方法时) ，会统一将 Runnable 转交给 ViewRootImpl 处理；

 - 而在 View#dispatchAttachedToWindow 时，也会为当前 View 初始化一个 AttachInfo 对象，该对象持有 ViewRootImpl 中AttachInfo 的引用，当 View 有此对象后，后续的所有 Runnable 都将由ViewRootImpl.AttachInfo 中的Handler发送至主线程的消息队列；

 - 而 ViewRootImpl 也会在执行 performTraversal 方法，也会调用 ViewRootImpl#getRunQueue ，利用 ThreadLocal 来为主线程维护一个 HandlerActionQueue 对象，至此，ViewRootImpl 内部都将使用该队列来进行 Runnable 任务的短期维护；

 - 但需要注意的是，各个 View 调用的 post 方法，仍然是由各自的 HandlerActionQueue 对象来入队任务的，然后在 View#dispatchAttachedToWindow 的时候将Runnable发送至主线程消息队列。


### View#post的版本变动(API23)

```java
    public boolean post(Runnable action) {
        final AttachInfo attachInfo = mAttachInfo;
        if (attachInfo != null) {
            return attachInfo.mHandler.post(action);
        }
        // Assume that post will succeed later
        // 注意此处，不同于我们之前介绍的，这里是直接使用 ViewRootImpl#getRunQueue 来入队任务的
        ViewRootImpl.getRunQueue().post(action);
        return true;
    }
```
API23 版本中，View#post 在没有 attach 到 window 之前，也就是 mAttachInfo 是 null 的时候，不是自己维护任务队列，而是直接使用 ViewRootImpl#getRunQueue 来入队任务的。

```java
    static final ThreadLocal<RunQueue> sRunQueues = new ThreadLocal<RunQueue>();
    static RunQueue getRunQueue() {
        RunQueue rq = sRunQueues.get();
        if (rq != null) {
            return rq;
        }
        rq = new RunQueue();
        sRunQueues.set(rq);
        return rq;
    }
```

其中ThreadLocal 是以线程为 key 值来存取RunQueue对象。假如我们在子线程中 post 了一个 runnable，其实是以当前子线程为key，创建RunQueue队列并将runnable保存起来等待执行。但在API23的源码中只有一次调用executeActions()方法，那就是在ViewRootImpl#performTraversal中。当执行到ViewRootImpl中的executeActions()方法时，系统却是去主线程队列中寻找待执行的 Runnable，那当然是永远都得不到执行了。

在 attach 到 window之后，View会持有ViewRootImpl中AttachInfo的引用，然后直接使用AttachInfo中的Handler对象将Runnable封装成Message后发送至主线程消息队列。


### 参考文章

[https://mp.weixin.qq.com/s/aA-9UTlebdj5-K4z30f_4g](https://mp.weixin.qq.com/s/aA-9UTlebdj5-K4z30f_4g)

[https://blog.csdn.net/a740169405/article/details/69668957](https://blog.csdn.net/a740169405/article/details/69668957)

[https://blog.csdn.net/scnuxisan225/article/details/49815269](https://blog.csdn.net/scnuxisan225/article/details/49815269)

[https://www.cnblogs.com/plokmju/p/7481727.html](https://www.cnblogs.com/plokmju/p/7481727.html)







 





























