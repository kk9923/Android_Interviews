## 享元模式

### 定义

使用共享对象可有效的支持大量的细粒度的对象.

享元模式是对象池的一种实现，代表轻量级的意思。享元模式用来尽可能减少内存使用量，它适用于可能存在大量重复对象的场景，来缓存可共享的对象，达到共享对象，避免创建过多对象的效果，可以提升性能，避免内存溢出。

享元对象中部分状态是可以共享的，可以共享的状态称为内部状态，内部状态不会随着环境变化，不可共享的状态则称之为外部状态，他们会随着环境的改变而改变。在享元模式中会建立一个对象容器，在经典的享元模式中该容器为Map,它的键是享元对象的内部状态，它的值就是享元对象本身,客户端程序通过这个内部状态从享元工厂中获取享元对象，如果有缓存则使用缓存对象，否则创建一个享元对象并且存入容器中，这样一来就避免了创建过多对象的问题。

### 享元模式分类
 - 单纯享元模式
 - 复合享元模式
 
### 单纯享元模式结构重要核心模块

**FlyWeight(抽象享元角色)**

为具体享元角色规定了必须实现的方法，而外部状态就是以参数的形式通过此方法传入。在Java中可以由抽象类、接口来担当。

**ConcreteFlyWeight(具体享元角色)**

实现抽象角色规定的方法。如果存在内部状态，就负责为内部状态提供存储空间.


**FlyWeightFactory(享元工厂角色)**

负责创建和管理享元角色。要想达到共享的目的，这个角色的实现是关键！

单纯享元模式和创建型的简单工厂模式实现上非常相似，但是它的重点或者用意却和工厂模式截然不同。工厂模式的使用主要是为了使系统不依赖于实现的细节；而在享元模式的主要目的是避免大量拥有相同内容对象的开销。

### 单纯享元模式简单实现


**FlyWeight**


```java
    /**
     *  抽象享元角色
     */
     interface TicketFlyWeight {
         void buyTicketInfo(String ticketType);
    }

```

**ConcreteFlyWeight**


```java
    /**
     * 具体享元角色
     */
    public class TicketConcreteFlyWeight implements TicketFlyWeight {
        private String from;
        private String to;
    
         TicketConcreteFlyWeight(String from, String to) {
            this.from = from;
            this.to = to;
        }
    
        @Override
        public void buyTicketInfo(String ticketType) {
            System.out.println("购买 从" + from +"  到  " + to + "  的火车票   ticketType"  + ticketType
                + "  ,价格 ：  " +     getTicketPrice(ticketType)
            );
        }
    
        private int getTicketPrice(String ticketType){
            if ("坐票".equals(ticketType)){
                return 50;
            }else if ("上铺".equals(ticketType)){
                return 100;
            }else if ("下铺".equals(ticketType)){
                return 200;
            }else {
                return  10;
            }
        }
    }
```

**FlyWeightFactory**


```java
    /**
     *  享元工厂
     */
    public class FlyWeightFactory  {
    
       public static Map<String,TicketConcreteFlyWeight> sTicketMap = new ConcurrentHashMap<>();
    
        public static TicketFlyWeight getTicket(String from,String to){
            String key = from + "-" + to;
            TicketConcreteFlyWeight ticketConcreteFlyWeight = sTicketMap.get(key);
            if (ticketConcreteFlyWeight ==null){
                System.out.println("创建对象  ==> " +key);
                ticketConcreteFlyWeight = new TicketConcreteFlyWeight(from,to);
                sTicketMap.put(key,ticketConcreteFlyWeight);
                return ticketConcreteFlyWeight ;
            }
            System.out.println("使用缓存  ==> " +key);
            return ticketConcreteFlyWeight ;
        }
    }
```

**测试代码**


```java
        TicketFlyWeight ticket1 = FlyWeightFactory.getTicket("武汉", "北京");
        ticket1.buyTicketInfo("上铺");

        TicketFlyWeight ticket2 = FlyWeightFactory.getTicket("武汉", "北京");
        ticket2.buyTicketInfo("下铺");

        TicketFlyWeight ticket3 = FlyWeightFactory.getTicket("武汉", "北京");
        ticket3.buyTicketInfo("走道");


        System.out.println("缓存容器大小 ：  "+FlyWeightFactory.sTicketMap.size());  
```

**输出结果**


```
         创建对象  ==> 武汉-北京
         购买 从武汉  到  北京  的火车票   ticketType上铺  ,价格 ：  100

         使用缓存  ==> 武汉-北京
         购买 从武汉  到  北京  的火车票   ticketType下铺  ,价格 ：  200

         使用缓存  ==> 武汉-北京
         购买 从武汉  到  北京  的火车票   ticketType走道  ,价格 ：  10

         缓存容器大小 ：  1
```

当内部状态即缓存的key相同时，只在第一次创建对象，之后都从缓存Map中去取对象，避免了多次创建对象。


Flyweight(享元)模式是如此的重要，因为它能帮你在一个复杂的系统中大量的节省内存空间。在JAVA语言中，String类型就是使用了享元模式。String对象是final类型，对象一旦创建就不可改变。在JAVA中字符串常量都是存在常量池中的，JAVA会确保一个字符串常量在常量池中只有一个拷贝。String a="abc"，其中"abc"就是一个字符串常量。


```java
    String a = "hello";
    String b = "hello";
    if(a == b)
    　System.out.println("OK");
    else
    　System.out.println("Error");
```
输出结果是：OK。可以看出if条件比较的是两a和b的地址，也可以说是内存空间。


**享元模式优点**

 - 可以极大减少内存中对象的数量，使得相同或相似对象在内存中只保存一份，从而可以节约系统资源，提高系统性能。
 - 
 - 享元模式的外部状态相对独立，而且不会影响其内部状态，从而使得享元对象可以在不同的环境中被共享。
 

 **享元模式缺点**
 
 - 享元模式使得系统变得复杂，需要分离出内部状态和外部状态，这使得程序的逻辑复杂化。
 - 为了使对象可以共享，享元模式需要将享元对象的部分状态外部化，而读取外部状态将使得运行时间变长。




