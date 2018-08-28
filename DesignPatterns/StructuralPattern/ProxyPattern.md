## 代理模式

### 定义

代理模式(Proxy Pattern) ：**为其他对象提供一种代理以控制对这个的访问。** 当无法或不想直接访问某个对象或访问某个对象存在困难时可以通过一个代理对象来间接访问。

一种代表了某一类，即代理类和被代理类必须实现同一接口，这个接口下的所有实现类都能被代理访问到。

### 核心模块

**Subject（抽象主题角色）** 

它声明了真实主题和代理主题的共同接口方法，该类既可以是一个抽象类，也可以是一个接口。

**RealSubject（真实主题角色）**

该类也被称为被委托类或被代理类，该类定义了代理所表示的真实对象，由其执行具体的业务逻辑，而客户类则通过代理类间接的调用真实主题类中定义的方法。

**ProxySubject（代理主题角色）**

该类被称为委托类或代理类，该类持有一个对真实主题类的引用，在其所实现的接口方法中调用真实主题类中相应的接口方法执行，以起到代理的作用。

### 代理模式分类

**远程代理(Remote Proxy)**

给一个位于不同的地址空间的对象提供一个本地的代理对象，这个不同的地址空间可以是在同一台主机中，也可是在另一台主机中，远程代理又称为大使(Ambassador)。

**虚拟代理(Virtual Proxy)**

如果需要创建一个资源消耗较大的对象，先创建一个消耗相对较小的对象来表示，真实对象只在需要时才会被真正创建。

**保护代理(Protect Proxy)**

控制对一个对象的访问，可以给不同的用户提供不同级别的使用权限。

**缓冲代理(Cache Proxy)**

为某一个目标操作的结果提供临时的存储空间，以便多个客户端可以共享这些结果。

**智能引用代理(Smart Reference Proxy)**

当一个对象被引用时，提供一些额外的操作，例如将对象被调用的次数记录下来等。

### 静态代理模式的简单实现

**抽象主题角色**

```java
    /**
     *  抽象主题角色 ---- 声明真实主题与代码共同的接口方法
     */
     abstract  class Subject {
    
         abstract void  buyComputer();
    
    }
```

**具体主题角色**

```java
    /**
     *  真实主题类(也被称为被委托类或被代理类)  ----    执行具体的业务逻辑
     */
     class RealSubject extends Subject {
        @Override
        void buyComputer() {
            System.out.println("购买了一台最新款的MacPro");
        }
    }
```

**代理类**

```java
    /**
     *  代理类(也被称为委托类)  --   持有一个真实主题的引用
     */
     class ProxySubject extends Subject {
    
        // 持有一个被代理对象的引用
        private  Subject mSubject ;
    
         ProxySubject(Subject subject) {
            mSubject = subject;
        }
    
        @Override
        void buyComputer() {
            mSubject.buyComputer();
        }
    }

```
**测试代码**


```java
        // 构造真实主题类(被代理类)
        Subject realSubject = new RealSubject();
        // 构造代理类(只有真实主题类的引用)
        Subject proxySubject = new ProxySubject(realSubject);
        //  由代理类去购买电脑
        proxySubject.buyComputer();
```

**输出结果**

```
        购买了一台最新款的MacPro
```

### 静态代理模式的简单实现

Proxy代理类作为中介提供对目标资源的访问，但是Proxy和RealSubject目标对象，本质上是相同的，如果只是为了提供对某些资源的访问，就创建一个代理类，编译并运行使之存在于系统中，当需要大量代理类时，系统结构会变得比较臃肿。而且当接口需要修改方法时，不仅其所有实现类需要修改代码，代理类也要修改，程序就会变得复杂和难以维护。
-- 为了解决这种问题，就有了jdk动态代理创建代理类的想法，在运行状态时，在需要代理的地方而不是每个要代理的地方，根据接口Subject和目标对象RealSubject，动态地创建一个代理类，用完后就销毁，这样就能避免Proxy在系统中大量存在的问题。


Java给我们提供了一个便捷的动态代理接口InvocationHandler,实现接口需要重写其invoke方法。


```java
/**
 *  动态代理类  ---  持有 被代理类的引用
 */
public class DynamicProxy implements InvocationHandler {
    // 被代理类的引用
    private Object mObject;

    public DynamicProxy(Object object) {
        mObject = object;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        // 调用被代理类 对象的方法
        Object invoke = method.invoke(mObject, args);
        return invoke;
    }
}
```

**修改后的客户类**

```java
        // 构造一个动态代理
        DynamicProxy proxy = new DynamicProxy(realSubject);
        //  获取被代理类的ClassLoader
        ClassLoader classLoader = realSubject.getClass().getClassLoader();
        //  动态的构造一个代理类
        Subject dynamicProxy = (Subject) Proxy.newProxyInstance(classLoader,new Class[]{Subject.class},proxy);
        //  由代理类去购买电脑
        dynamicProxy.buyComputer();
```
**注意：动态代理需要将抽象主题角色Subject改为接口，否则会报错**


```java
  Caused by: java.lang.IllegalArgumentException: com.kx.designpatterns.StructuralPattern.ProxyPattern.Subject is not an interface
        at java.lang.reflect.Proxy$ProxyClassFactory.apply(Proxy.java:589)
        at java.lang.reflect.Proxy$ProxyClassFactory.apply(Proxy.java:566)
        at java.lang.reflect.WeakCache$Factory.get(WeakCache.java:230)
        at java.lang.reflect.WeakCache.get(WeakCache.java:127)
        at java.lang.reflect.Proxy.getProxyClass0(Proxy.java:419)
        at java.lang.reflect.Proxy.newProxyInstance(Proxy.java:820)
        at com.kx.designpatterns.StructuralPattern.ProxyPattern.ProxyActivity.onCreate(ProxyActivity.java:30)
```
输出结果和静态代理的输出结果一样，就不显示了。

动态代理通过一个代理类来处理N多个被代理类，其实质是对代理者与被代理者的解耦，使两者没有直接的耦合关系。相对而言静态代理则只能为给定接口下的实现类做代理，如果接口不同那么就需要重新定义不同代理类，较为复杂，但是静态代理更符合面向对象原则。

### Android源码中的代理模式实现

在Android源码中ActivityManagerProxy就是代理类，其具体代理的是ActivityManagerNative的子类ActivityManagerService。其中ActivityManagerProxy和ActivityManagerNative都实现了IActivityManager接口。严格来说，IActivityManager就相当于代理模式中的抽象主题，ActivityManagerNative就是真实主题，ActivityManagerProxy就是代理类，但ActivityManagerNative是个抽象类，并不处理过多的具体逻辑，大部分具体实现都由其子类ActivityManagerService承担，所以真实主题应该是ActivityManagerService而非ActivityManagerNative。

ActivityManagerService是系统级的Service并且运行与独立的进程空间中，我们可以通过ServiceManager来获取它。而ActivityManagerProxy也运行于自己所处的进程空间中，两者之间必定是通过Android的Binder机制来实现跨进程通信的。

### 参考文档

《Android源码设计模式》

[https://www.jianshu.com/p/5478f170d9ee](https://www.jianshu.com/p/5478f170d9ee)










