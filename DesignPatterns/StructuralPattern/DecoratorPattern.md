## 装饰模式
### 定义
 装饰模式(Decorator Pattern) 也被称为包装模式(Wrapper Pattern),结构型设计模式之一，其使用对客户端透明的方式来动态的扩展对象的功能，同时它也是继承关系的一种替代方案值一。
 **动态的给一个对象添加一些额外的职责**。就增加功能来说，装饰模式相比生成子类更为灵活。
 
 装饰模式可以在不改变一个对象本身功能的基础上给对象增加额外的新行为。装饰模式是一种用于替代继承的技术，它通过一种无须定义子类的方式来给对象动态增加职责，使用对象之间的关联关系取代类之间的继承关系。在装饰模式中引入了装饰类，在装饰类中既可以调用待装饰的原有类的方法，还可以增加新的方法，以扩充原有类的功能。
 
 ### 核心
 
 **概念： 动态地给一个对象增加一些额外的职责，就增加对象功能来说，装饰模式比生成子类实现更为灵活。装饰模式是一种对象结构型模式。**
 
 
 **Component(抽象组件)** 
 
 它是具体组件和抽象装饰类的共同父类，声明了在具体组件中实现的业务方法，可以是一个接口或者抽象类，其充当的就是被装饰的原始对象。

**ConcreteComponent（具体组件）**

它是抽象组件类的子类，用于定义具体的组件对象，实现了在抽象组件中声明的方法，是我们装饰的具体对象。

**Decorator（抽象装饰类）**

它也是抽象组件类的子类，用于给具体组件增加职责，但是具体职责在其子类中实现。它维护一个指向抽象组件对象的引用，通过该引用可以调用装饰之前组件对象的方法，并通过其子类扩展该方法，以达到装饰的目的。在大多数情况下，该类为抽象类，需要根据不同的装饰逻辑实现不同的具体子类。如果装饰逻辑单一，我们可以省略该类，直接作为具体的装饰者。

**ConcreteDecorator（具体装饰类）**

它是抽象装饰类的子类，负责向构件添加新的职责。每一个具体装饰类都定义了一些新的行为，它可以调用在抽象装饰类中定义的方法，并可以增加新的方法用以扩充对象的行为。

### 代码实现

抽象组件

```java
        /**
         *   抽象组件 ---  可以是一个接口或抽象类， 充当被装饰的原始对象。
         */
        public abstract class Hamburger {
            protected  String name ;
        
            public String getName(){
                return name;
            }
        
            public abstract double getPrice();
        }

```

具体组件实现类

```java
        /**
         *   组件具体实现类 ---  被装饰的具体对象
         */
        public class ChickenBurger extends Hamburger {
        
            public ChickenBurger() {
                name = "鸡腿堡";
            }
        
            @Override
            public double getPrice() {
                return 10;
            }
        }
```

抽象装饰者

```java
        /**
         *   抽象装饰者，内部一定要有一个指向组件对象的引用.
         */
        public abstract class HamburgerWrapper extends Hamburger {
        
            protected Hamburger mHamburger ;
        
            public HamburgerWrapper(Hamburger hamburger) {
                mHamburger = humburger;
            }
            
            public abstract String getName();
        
            @Override
            public double getPrice() {
                return mHamburger.getPrice();
            }
        }
```
具体装饰者


```java
        /**
         *  具体装饰类，抽象装饰者的具体实现
         */
        public class Lettuce extends HamburgerWrapper {
        
            public Lettuce(Hamburger hamburger) {
                super(hamburger);
            }
            @Override
            public String getName() {
                return mHamburger.getName() + "  加生菜";
            }
        
            @Override
            public double getPrice() {
                return mHamburger.getPrice() + 2.5;
            }
        }
```


```
        /**
         *  具体装饰类，抽象装饰者的具体实现
         */
        public class Chilli extends HamburgerWrapper {
        
            public Chilli(Hamburger hamburger) {
               super(hamburger);
            }
        
            @Override
            public String getName() {
                return mHamburger.getName() + "  加辣椒";
            }
        
            @Override
            public double getPrice() {
                return mHamburger.getPrice() + 0.5;
            }
        }
```
测试代码

```
       // 构造被装饰的组件对象
        Hamburger hamburger = new ChickenBurger();
        // 输出  被装饰组件对象的   Name 和  Price
        System.out.println(hamburger.getName()+"  价钱："+hamburger.getPrice());

        // 根据 被装饰的组件对象构造装饰者
        Lettuce lettuce = new Lettuce(hamburger);
        //  输出装饰后的   Name 和  Price
        System.out.println(lettuce.getName()+"  价钱："+lettuce.getPrice());

        //  以下同理
        Chilli chilli = new Chilli(hamburger);
        System.out.println(chilli.getName()+"  价钱："+chilli.getPrice());
        Chilli chilli2 = new Chilli(lettuce);
        System.out.println(chilli2.getName()+"  价钱："+chilli2.getPrice());
```
输出


```
        鸡腿堡  价钱：10.0
        鸡腿堡  加生菜  价钱：12.5
        鸡腿堡  加辣椒  价钱：10.5
        鸡腿堡  加生菜  加辣椒  价钱：13.0

```

### 透明性的要求
装饰者模式对客户端的透明性要求程序不要声明一个ConcreteComponent类型的变量，而应当声明一个Component类型的变量

下面的做法是对的：

```
        Hamburger hamburger = new ChickenBurger();
        Hamburger lettuce = new Lettuce(hamburger);
        Hamburger chilli = new Chilli(hamburger);
```
而下面的做法是不对的：

```
        Hamburger hamburger = new ChickenBurger();
        Lettuce lettuce = new Lettuce(hamburger);
        Chilli chilli = new Chilli(hamburger);
```

### 半透明的装饰者模式

纯粹的装饰模式很难找到。装饰模式的用意是在不改变接口的前提下，增强所考虑的类的性能。在增强性能的时候，往往需要建立新的公开的方法，意味着装饰者的具体实现类需要创建新的方法。大多数的装饰模式的实现都是“半透明”的，而不是完全透明的。换言之，允许装饰模式改变接口，增加新的方法。这意味着客户端可以声明ConcreteDecorator类型的变量，从而可以调用ConcreteDecorator类中才有的方法：

```
        Hamburger hamburger = new ChickenBurger();
        Chili chili = new Chili(hamburger);
        chili.addMoreChili();
```

半透明的装饰模式是介于装饰模式和适配器模式之间的。适配器模式的用意是改变所考虑的类的接口，也可以通过改写一个或几个方法，或增加新的方法来增强或改变所考虑的类的功能。大多数的装饰模式实际上是半透明的装饰模式，这样的装饰模式也称做半装饰、半适配器模式。

装饰模式和适配器模式都是“包装模式(Wrapper Pattern)”，它们都是通过封装其他对象达到设计的目的的，但是它们的形态有很大区别。

理想的装饰模式在对被装饰对象进行功能增强的同时，要求具体构件角色、装饰角色的接口与抽象构件角色的接口完全一致。而适配器模式则不然，一般而言，适配器模式并不要求对源对象的功能进行增强，但是会改变源对象的接口，以便和目标接口相符合。

装饰模式有透明和半透明两种，这两种的区别就在于装饰角色的接口与抽象构件角色的接口是否完全一致。透明的装饰模式也就是理想的装饰模式，要求具体构件角色、装饰角色的接口与抽象构件角色的接口完全一致。相反，如果装饰角色的接口与抽象构件角色接口不一致，也就是说装饰角色的接口比抽象构件角色的接口宽的话，装饰角色实际上已经成了一个适配器角色，这种装饰模式也是可以接受的，称为“半透明”的装饰模式，如下图所示。

![image](https://camo.githubusercontent.com/3238d9d2e1805e229ded171ece568451ed17f2cb/687474703a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f333938353536332d666665613562383532313133383930352e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430)

在适配器模式里面，适配器类的接口通常会与目标类的接口重叠，但往往并不完全相同。换言之，适配器类的接口会比被装饰的目标类接口宽。

显然，半透明的装饰模式实际上就是处于适配器模式与装饰模式之间的灰色地带。如果将装饰模式与适配器模式合并成为一个“包装模式”的话，那么半透明的装饰模式倒可以成为这种合并后的“包装模式”的代表。

**优点**
 - 装饰模式与继承关系的目的都是要扩展对象的功能，但是装饰模式可以提供比继承更多的灵活性。装饰模式允许系统动态决定“贴上”一个需要的“装饰”，或者除掉一个不需要的“装饰”。继承关系则不同，继承关系是静态的，它在系统运行前就决定了。

 - 过使用不同的具体装饰类以及这些装饰类的排列组合，设计师可以创造出很多不同行为的组合。

**缺点**
 - 由于使用装饰模式，可以比使用继承关系需要较少数目的类。使用较少的类，当然使设计比较易于进行。但是，在另一方面，使用装饰模式会产生比使用继承关系更多的对象。更多的对象会使得查错变得困难，特别是这些对象看上去都很相像。
 - 多层的装饰是比较复杂的。
 

### 装饰者模式在Java IO流中的应用
装饰者模式在Java语言中的最著名的应用莫过于Java I/O标准库的设计了。

由于Java I/O库需要很多性能的各种组合，如果这些性能都是用继承的方法实现的，那么每一种组合都需要一个类，这样就会造成大量性能重复的类出现。而如果采用装饰者模式，那么类的数目就会大大减少，性能的重复也可以减至最少。因此装饰者模式是Java I/O库的基本模式。

Java I/O库的对象结构图如下，由于Java I/O的对象众多，因此只画出InputStream的部分。

![](http://upload-images.jianshu.io/upload_images/3985563-61242a7817bd22bc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

  


  
根据上图可以看出：

　　●　　抽象构件(Component)角色：由InputStream扮演。这是一个抽象类，为各种子类型提供统一的接口。

　　●　　具体构件(ConcreteComponent)角色：由ByteArrayInputStream、FileInputStream、PipedInputStream、StringBufferInputStream等类扮演。它们实现了抽象构件角色所规定的接口。

　　●　　抽象装饰(Decorator)角色：由FilterInputStream扮演。它实现了InputStream所规定的接口。

　　●　　具体装饰(ConcreteDecorator)角色：由几个类扮演，分别是BufferedInputStream、DataInputStream以及两个不常用到的类LineNumberInputStream、PushbackInputStream。
　　
　　##### InputStream类型中的装饰者模式

InputStream类型中的装饰者模式是半透明的。为了说明这一点，不妨看一看作装饰者模式的抽象构件角色的InputStream的源代码。这个抽象类声明了九个方法，并给出了其中八个的实现，另外一个是抽象方法，需要由子类实现。

```java
public abstract class InputStream implements Closeable {

    public abstract int read() throws IOException;


    public int read(byte b[]) throws IOException {}

    public int read(byte b[], int off, int len) throws IOException {}

    public long skip(long n) throws IOException {}

    public int available() throws IOException {}

    public void close() throws IOException {}

    public synchronized void mark(int readlimit) {}

    public synchronized void reset() throws IOException {}

    public boolean markSupported() {}
}
```

下面是作为装饰模式的抽象装饰角色FilterInputStream类的源代码。可以看出，FilterInputStream的接口与InputStream的接口是完全一致的。也就是说，直到这一步，还是与装饰模式相符合的。

```java
public class FilterInputStream extends InputStream {
    protected FilterInputStream(InputStream in) {}

    public int read() throws IOException {}

    public int read(byte b[]) throws IOException {}

    public int read(byte b[], int off, int len) throws IOException {}

    public long skip(long n) throws IOException {}

    public int available() throws IOException {}

    public void close() throws IOException {}

    public synchronized void mark(int readlimit) {}

    public synchronized void reset() throws IOException {}

    public boolean markSupported() {}
}
```

下面是具体装饰角色PushbackInputStream的源代码。

```java
public class PushbackInputStream extends FilterInputStream {
    private void ensureOpen() throws IOException {}

    public PushbackInputStream(InputStream in, int size) {}

    public PushbackInputStream(InputStream in) {}

    public int read() throws IOException {}

    public int read(byte[] b, int off, int len) throws IOException {}

    public void unread(int b) throws IOException {}

    public void unread(byte[] b, int off, int len) throws IOException {}

    public void unread(byte[] b) throws IOException {}

    public int available() throws IOException {}

    public long skip(long n) throws IOException {}

    public boolean markSupported() {}

    public synchronized void mark(int readlimit) {}

    public synchronized void reset() throws IOException {}

    public synchronized void close() throws IOException {}
}
```

查看源码，你会发现，这个装饰类提供了额外的方法unread\(\)，这就意味着PushbackInputStream是一个半透明的装饰类。换言 之，它破坏了理想的装饰者模式的要求。如果客户端持有一个类型为InputStream对象的引用in的话，那么如果in的真实类型是 PushbackInputStream的话，只要客户端不需要使用unread\(\)方法，那么客户端一般没有问题。但是如果客户端必须使用这个方法，就 必须进行向下类型转换。将in的类型转换成为PushbackInputStream之后才可能调用这个方法。但是，这个类型转换意味着客户端必须知道它 拿到的引用是指向一个类型为PushbackInputStream的对象。这就破坏了使用装饰者模式的原始用意。

现实世界与理论总归是有一段差距的。纯粹的装饰者模式在真实的系统中很难找到。一般所遇到的，都是这种半透明的装饰者模式。

下面是使用I/O流读取文件内容的简单操作示例。

```java
public class IOTest {

    public static void main(String[] args) throws IOException {
        // 流式读取文件
        DataInputStream dis = null;
        try{
            dis = new DataInputStream(new BufferedInputStream(new FileInputStream("test.txt"))
            );
            //读取文件内容
            byte[] bs = new byte[dis.available()];
            dis.read(bs);
            String content = new String(bs);
            System.out.println(content);
        }finally{
            dis.close();
        }
    }
}
```

观察上面的代码，会发现最里层是一个FileInputStream对象，然后把它传递给一个BufferedInputStream对象，经过BufferedInputStream处理，再把处理后的对象传递给了DataInputStream对象进行处理，这个过程其实就是装饰器的组装过程，FileInputStream对象相当于原始的被装饰的对象，而BufferedInputStream对象和DataInputStream对象则相当于装饰器。


### 参考文档
[https://blog.csdn.net/jason0539/article/details/22713711](https://blog.csdn.net/jason0539/article/details/22713711)

[https://blog.csdn.net/caihuangshi/article/details/51334097](https://blog.csdn.net/caihuangshi/article/details/51334097)