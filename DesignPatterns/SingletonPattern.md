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








