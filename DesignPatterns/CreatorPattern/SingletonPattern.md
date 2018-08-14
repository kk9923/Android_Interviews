## 单例模式
### 定义
 确保某个类只有一个实例对象，而且自行实例化并向整个系统提供这个实例。

### 单例特点
 - 构造函数私有化,一般为private;
 - 通过一个静态的方法或者枚举返回单例对象;
 - 确保单例类的对象有且只有一个,尤其是在多线程的情况下;
 - 确保单例对象在反序列化时不会重新构建对象.

### 单例的优点
 - 由于在系统内存中只存在一个对象，因此可以节约系统资源，对于一些需要频繁创建和销毁的对象，单例模式无疑可以提高系统的性能。

### 单例的缺点
 - 单例模式一般没有接口，拓展很困难，基本都要修改源代码。
 - 单例类的职责过重，在一定程度上违背了“单一职责原则”。因为单例类既充当了工厂角色，提供了工厂方法，同时又充当了产品角色，包含一些业务方法，将产品的创建和产品的本身的功能融合到一起。
 - 现在很多面向对象语言(如Java、C#)的运行环境都提供了自动垃圾回收的技术，因此，如果实例化的共享对象长时间不被利用，系统会认为它是垃圾，会自动销毁并回收资源，下次利用时又将重新实例化，这将导致共享的单例对象状态的丢失。
 - 单例对象如果持有Context，那么很容易引发内存泄漏，此时需要注意传递给单例对象的Context最好是 Application Context。
 
## 示例与源码

### 一、饿汉式


```java
/**
 * 饿汉式单例模式(线程安全)
 */
public class Singleton {
    private static final Singleton sSingleton  = new Singleton();
    private Singleton() {
    }
    public static  Singleton getInstance() {
        return sSingleton;
    }
}
```
 在第一次加载类到内存中的时候就会初始化对象，所以创建实例本身是线程安全的.但类一开始就被初始化，即使客户端没有调用getInstance()方法。造成了不必要的内存开销


### 二、懒汉式

```java
/**
 * 懒汉式单例模式(线程不安全)
 */
public class Singleton {
    private static Singleton sSingleton ;
    //构造函数私有
    private Singleton() {
    }
    //对外提供公开方法
    public static Singleton getInstance() {
        if (sSingleton ==null){
           //需要用时才会创建对象,在第一次调用的时候实例化
           sSingleton = new Singleton();
        }
        return sSingleton;
    }
}

```
 但是要注意这种写法在多线程中是不安全的,有可能产生多个单例对象。当多个线程同时执行getInstance()函数时，此时sSingleton为null,然后同时执行new 操作，最后很有可能会创建多个不同的对象.


```java
/**
 * 懒汉式单例模式(线程安全)
 */
public class Singleton {
    private static Singleton sSingleton ;
    private Singleton() {
    }
    public static synchronized Singleton getInstance() {
        if (sSingleton ==null){
            sSingleton = new Singleton();
        }
        return sSingleton;
    }
}
```
 加上了synchronized关键字,做到了线程安全，并且解决了多实例的问题，但是它并不高效,因为在任何时候只能有一个线程调用 getInstance() 方法,造成不必要的同步开销

### 三、双重检查锁

双重检验锁模式（double checked locking pattern），是一种使用同步块加锁的方法有两次检查 instance == null，一次是在同步块外，一次是在同步块内。为什么在同步块内还要再检验一次？因为可能会有多个线程一起进入同步块外的 if，如果在同步块内不进行二次检验的话就会生成多个实例了

```java
public class Singleton {
    private static  Singleton mSingleton ;
    private Singleton(){}
    public static  Singleton getInstance(){
        if (mSingleton==null){
            synchronized (Singleton.class){
                if (mSingleton==null){
                    mSingleton=new Singleton();
                }
            }
        }
        return mSingleton;
    }
}

```
这段代码看起来很完美，很可惜，它是有问题。主要在于instance = new Singleton()这句，这并非是一个原子操作，事实上在 JVM 中这句话大概做了下面 3 件事情:

1.给 instance 分配内存

2.调用 Singleton 的构造函数来初始化成员变量

3.将instance对象指向分配的内存空间（执行完这步 instance 就为非 null 了）。

由于Java编译器允许处理器乱序执行，所以上面的第二步第三步的执行顺序没法得到保证。执行顺序可能是1-2-3也可能是1-3-2。当A线程执行顺序是1-3-2的时候，如果执行到了1-3，第2步还没执行的时候,这时 instance 已经是非 null 了(但却没有初始化),如果此时B线程访问getInstance()方法,会直接返回instance(非null，但没有初始化),然后在B线程使用该instance,则会报错。

### 四、静态内部类

```java
public class Singleton {  
    private static class SingletonHolder {  
        private static final Singleton INSTANCE = new Singleton();  
    }  
    private Singleton (){}  
    public static final Singleton getInstance() {  
        return SingletonHolder.INSTANCE; 
    }  
}
```
静态单例对象没有作为Singleton的成员变量直接实例化，因此类加载时不会实例化Singleton，**第一次调用getInstance()时将加载内部类SingletonHolder**，在该内部类中定义了一个static类型的变量INSTANCE ，此时会首先初始化这个成员变量,由Java虚拟机来保证其线程安全性，确保该成员变量只能初始化一次。由于getInstance()方法没有任何线程锁定，因此其性能不会造成任何影响。

由于 SingletonHolder 是私有的，除了 getInstance() 之外没有办法访问它，因此它是懒汉式的；同时读取实例的时候不会进行同步，没有性能缺陷；也不依赖 JDK 版本.

### 五、枚举单例

```java
public enum Singleton{
    INSTANCE;
    public void doThing(){
        System.out.println(this.hashCode());
    }
}
```
我们可以通过Singleton.INSTANCE来访问实例，这比调用getInstance()方法简单多了。创建枚举默认就是线程安全的，所以不需要担心double checked locking，而且还能防止反序列化导致重新创建新的对象。

### 特点
 上面的几种在有一种情况下会单例失效，出现重复创建对象，那就是反序列化。反序列化的时候会调用一个readResolve()方法重新生成一个实例，所以上面的几种方式要解决这个问题需要加入以下方法。
 
```java
    private Object readRedolve() throws ObjectStreamException{
        return SingletonHolder.mSingleton;
    }
```
 或者
```java
    private Object readRedolve() throws ObjectStreamException{
        return mSingleton;
    }
```
### 六、使用容器类实现的单例模式

```java
public class SingletonManager {
    private static  Map<String,Object> mObjectMap = new HashMap<>();
    public static void  registerService (String key,Object instance){
        if (!mObjectMap.containsKey(key)){
            mObjectMap.put(key,instance);
        }
    }
    public static Object getService(String key){
        return mObjectMap.get(key);
    }
}
```
在程序的初始，将多种单例类型注入到一个统一的管理类中，在使用时根据key获取对应类型的对象。这种方式使得我们可以管理多种类型的单例，并且在使用时可以通过同意的接口进行获取操作，降低了用户的使用成本，也对用户隐藏了具体实现，降低了耦合度。

### Android源码中的单例模式(SDK26)

在Android系统中我们经常会通过Context去获取系统级别的服务，如WindowsManagerService、ActivityManagerService、LayoutInflater等,这些服务会在合适的时候以单例的形式注册在系统中，在我们需要的时候就通过Context的getSystemService(String name)去获取，我们以LayoutInflater为例，通常我们使用LayoutInflater.from(Context)来获取LayoutInflater服务。

```java
 public static LayoutInflater from(Context context) {
        LayoutInflater LayoutInflater =
                (LayoutInflater) context.getSystemService(Context.LAYOUT_INFLATER_SERVICE);
        if (LayoutInflater == null) {
            throw new AssertionError("LayoutInflater not found.");
        }
        return LayoutInflater;
    }
```
我们可以看到内部使用的是Context的getSystemService(String name)方法，其实在Application、Activity、Service中都会存在一个Context对象。我们以Activity中的Context来说明。

一个Activity的入口是Activity的main函数，当我们启动一个Activity时，会通过Binder与ActivityManagerService通信，并且最终调用ActivityThread的performLaunchActivity()函数；


```java

    private Activity performLaunchActivity(ActivityClientRecord r, Intent customIntent) {
    
       //  获取Context对象
        ContextImpl appContext = createBaseContextForActivity(r);
        Activity activity = null;
        try {
            java.lang.ClassLoader cl = appContext.getClassLoader();
            //  通过反射创建Activity对象
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
            Application app = r.packageInfo.makeApplication(false, mInstrumentation);

           

            if (activity != null) {
             
                Window window = null;
               //  调用Activity的attach()方法，其中PhoneWindow就是在Activity的attach()方法中创建的
               //  将appContext等对象attach到Activity中
                activity.attach(appContext, this, getInstrumentation(), r.token,
                        r.ident, app, r.intent, r.activityInfo, title, r.parent,
                        r.embeddedID, r.lastNonConfigurationInstances, config,
                        r.referrer, r.voiceInteractor, window, r.configCallback);
                // 调用Activity的onCreat()方法
                if (r.isPersistable()) {
                    mInstrumentation.callActivityOnCreate(activity, r.state, r.persistentState);
                } else {
                    mInstrumentation.callActivityOnCreate(activity, r.state);
                }
            
                r.activity = activity;
                r.stopped = true;
                if (!r.activity.mFinished) {
                   //  调用Activity的onStart()方法
                    activity.performStart();
                    r.stopped = false;
                }
    }
```
可以看到Activity中获取的是ContextImpl

```java
     ContextImpl appContext = createBaseContextForActivity(r);
 
  
     private ContextImpl createBaseContextForActivity(ActivityClientRecord r) {
        //代码省略
        ContextImpl appContext = ContextImpl.createActivityContext(
                this, r.packageInfo, r.activityInfo, r.token, displayId, r.overrideConfig);
        //代码省略
        return appContext;
    }

```
继续查看ContextImpl的getSystemService(String name)方法

```java
 @Override
    public Object getSystemService(String name) {
        return SystemServiceRegistry.getSystemService(this, name);
    }
```
下面来分析下SystemServiceRegistry类的实现


```java
final class SystemServiceRegistry {

 
    // Service容器
    private static final HashMap<Class<?>, String> SYSTEM_SERVICE_NAMES = new HashMap<Class<?>, String>();
    private static final HashMap<String, ServiceFetcher<?>> SYSTEM_SERVICE_FETCHERS = new HashMap<String, ServiceFetcher<?>>();

 private static <T> void registerService(String serviceName, Class<T> serviceClass,
            ServiceFetcher<T> serviceFetcher) {
        SYSTEM_SERVICE_NAMES.put(serviceClass, serviceName);
        SYSTEM_SERVICE_FETCHERS.put(serviceName, serviceFetcher);
    }
    
    static abstract class CachedServiceFetcher<T> implements ServiceFetcher<T> {
        private final int mCacheIndex;

        public CachedServiceFetcher() {
            mCacheIndex = sServiceCacheSize++;
        }

        @Override
        @SuppressWarnings("unchecked")
        public final T getService(ContextImpl ctx) {
            final Object[] cache = ctx.mServiceCache;
            synchronized (cache) {
                // Fetch or create the service.
                Object service = cache[mCacheIndex];
                if (service == null) {
                    try {
                        service = createService(ctx);
                        cache[mCacheIndex] = service;
                    } catch (ServiceNotFoundException e) {
                        onServiceNotFound(e);
                    }
                }
                return (T)service;
            }
        }

        public abstract T createService(ContextImpl ctx) throws ServiceNotFoundException;
    }
    
   //  静态代码块,第一次加载该类时执行，
  static {
    
        //  代码省略    
        registerService(Context.LAYOUT_INFLATER_SERVICE, LayoutInflater.class,
                new CachedServiceFetcher<LayoutInflater>() {
            @Override
            public LayoutInflater createService(ContextImpl ctx) {
                return new PhoneLayoutInflater(ctx.getOuterContext());
            }});
        //  代码省略 
        
    } 
     //   根据key来获取对应的服务
     public static Object getSystemService(ContextImpl ctx, String name) {
        ServiceFetcher<?> fetcher = SYSTEM_SERVICE_FETCHERS.get(name);
        return fetcher != null ? fetcher.getService(ctx) : null;
    }

}

```
通过SystemServiceRegistry 中部分代码可知，在虚拟机第一次加载该类时会注册各种ServiceFatcher,其中就包括LayoutInflater Service,并将这些服务以键值对的形式存储在一个HashMap中，用户只需要根据key来获取到对应的ServiceFetcher,通过ServiceFetcher对象的getService函数来获取具体的服务对象。

### System Services 缓存实现

``` java
    // The system service cache for the system services that are cached per-ContextImpl.
    final Object[] mServiceCache = SystemServiceRegistry.createServiceCache();
```
在第一次创建ContextImpl时，会在ContextImpl类中创建一个Object数组，用于缓存system service，

SystemServiceRegistry.createServiceCache()代码写法，
```
    /**
     * Creates an array which is used to cache per-Context service instances.
     */
    public static Object[] createServiceCache() {
        return new Object[sServiceCacheSize];
    }
    
    
    static abstract class CachedServiceFetcher<T> implements ServiceFetcher<T> {
        private final int mCacheIndex;

        public CachedServiceFetcher() {
            mCacheIndex = sServiceCacheSize++;
        }

        @Override
        @SuppressWarnings("unchecked")
        public final T getService(ContextImpl ctx) {
            final Object[] cache = ctx.mServiceCache;
            synchronized (cache) {
                // Fetch or create the service.
                Object service = cache[mCacheIndex];
                if (service == null) {
                    try {
                        service = createService(ctx);
                        cache[mCacheIndex] = service;
                    } catch (ServiceNotFoundException e) {
                        onServiceNotFound(e);
                    }
                }
                return (T)service;
            }
        }

        public abstract T createService(ContextImpl ctx) throws ServiceNotFoundException;
    }
    
    
```
在第一次加载SystemServiceRegistry类的时候会先加载静态代码块，通过HashMap缓存了多个ServiceFetcher，不断registerService的过程中总的缓存数量sServiceCacheSize也在不断的+1，同时每个ServiceFetcher自身也保存了一个mCacheIndex索引。当静态代码块执行完毕后，sServiceCacheSize的值就是所有系统服务的个数。然后才会去调用

```java
    public static Object[] createServiceCache() {
        return new Object[sServiceCacheSize];
    }
```
这样，ContextImpl中mServiceCache数组的lenth就是所有在SystemServiceRegistry静态代码块中注册过的系统服务的个数。当调用ContextImpl的getSystemService(String name)方法获得HashMap中缓存的ServiceFetcher后，根据每个ServiceFetcher保存的mCacheInde索引去去查看ContextImpl中mServiceCache数组对应的Service是否为空，如果为空则调用createService(ctx)方法创建service，并将数组对应的位置赋值，下次调用的时候则直接从数组中取出缓存的Service;


```java
        public final T getService(ContextImpl ctx) {
            final Object[] cache = ctx.mServiceCache;
            synchronized (cache) {
                // Fetch or create the service.
                Object service = cache[mCacheIndex];
                if (service == null) {
                    try {
                        service = createService(ctx);
                        cache[mCacheIndex] = service;
                    } catch (ServiceNotFoundException e) {
                        onServiceNotFound(e);
                    }
                }
                return (T)service;
            }
        }
```
当第一次获取时会调用ServiceFetcher的creatService函数创建服务对象，并将对象缓存到一个数组中，下次直接从缓存中获取，避免重复创建对象，从而达到单例的效果。通过容器的单例模式实现方式，系统核心服务以单例形式存在，减少了资源消耗。
