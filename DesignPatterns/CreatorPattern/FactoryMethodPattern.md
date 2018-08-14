## 工厂方法模式
### 定义
工厂方法模式(Factory Method Pattern)又称为工厂模式，也叫虚拟构造器(Virtual Constructor)模式或者多态工厂(Polymorphic Factory)模式，它属于类创建型模式。

在工厂方法模式中，工厂父类负责定义创建产品对象的公共接口，而工厂子类则负责生成具体的产品对象，这样做的目的是将产品类的实例化操作延迟到工厂子类中完成，即通过工厂子类来确定究竟应该实例化哪一个具体产品类。

#### 核心模块

**抽象工厂**

Factory,其为工厂方法模式的核心，任何创建对象的工厂类必须实现该接口，通常以抽象类的形式出现。

**具体工厂**

ConcreteFactory,实现了具体的业务逻辑，有多少种产品，就需要有多少个具体的工厂实现。

**抽象产品**

Product,工厂方法模式所创建的产品的父类，通常以抽象类实现。

**具体产品**

ConcreteProduct,实现了抽象产品所声明的接口，工厂方法模式所创建的每一个对象都是某个具体产品的实例.

### 工厂方法模式的使用实例
 以汽车工厂生产汽车为例使用工厂方法模式写代码。其中ICarFactory为抽象的工厂角色，AudiQ3Factory和AudiQ5Factory为具体的工厂实现类，ICar为抽象的产品角色，AudiQ3和AudiQ5为具体的产品。
 
 **工厂类**
 ```java
 /**
  * 汽车的抽象工厂类
  */
 public abstract class ICarFactory {
     abstract ICar creatCar();
 }
 /**
  * 汽车工厂的具体具体实现类
  */
 public class AudiQ3Factory extends ICarFactory {
     @Override
     ICar creatCar() {
         return new AudiQ3();
     }
 }
 
  public class AudiQ5Factory extends ICarFactory {
      @Override
      ICar creatCar() {
          return new AudiQ5();
      }
  }
 ```
 **产品类**
 ```java
 /**
  * 汽车的抽象产品类
  */
 public abstract  class ICar {
     /**
      * 定义汽车的两个方法
      */
     abstract void drive();
 
     abstract void selfNavigation();
 
 }
 
 /**
  * 汽车的具体实现类
  */
 public class AudiQ3 extends ICar {
     @Override
     void drive() {
         System.out.println("Q3开车了");
     }
 
     @Override
     void selfNavigation() {
         System.out.println("Q3开启自动导航");
     }
 }
 
 public class AudiQ5 extends ICar {
     @Override
     void drive() {
         System.out.println("Q5开车了");
     }
 
     @Override
     void selfNavigation() {
         System.out.println("Q5开启自动导航");
     }
 }

 ```
 **客户端调用**
 ```java
   //创建Q3的工厂实现类，并生产Q3
         ICar carQ3 = new AudiQ3Factory().creatCar();
         carQ3.drive();
         carQ3.selfNavigation();
         //创建Q5的工厂实现类，并生产Q5
         ICar carQ5 = new AudiQ5Factory().creatCar();
         carQ5.drive();
         carQ5.selfNavigation();
         
        //输出结果 
         Q3开车了
         Q3开启自动导航
         Q5开车了
         Q5开启自动导航
         
 ```
 
 ### 工厂方法模式的优缺点
 
 **优点**
 - 在工厂方法模式中，工厂方法用来创建客户所需要的产品，向客户隐藏了产品类实例化的细节，用户只需关心所需产品对应的工厂，无需关心创建细节。
 - 基于工厂角色和产品角色的多态性设计是工厂方法模式的关键。它能够使**工厂可以自主确定创建何种产品对象**，而如何创建这个对象的细节则完全封装在具体工厂内部。工厂方法模式之所以又被称为多态工厂模式，是因为所有的具体工厂类都具有同一抽象父类。
 - 使用工厂方法模式的另一个优点是**在系统中加入新产品时**，无须修改抽象工厂和抽象产品提供的接口，无须修改客户端，也无须修改其他的具体工厂和具体产品，而**只要添加一个具体工厂和具体产品就可以了**。这样，系统的可扩展性也就变得非常好，**完全符合“开闭原则”**，这点比简单工厂模式更优秀。
 
 **缺点**
 - 在添加新产品时，需要编写新的具体产品类，而且还要提供与之对应的具体工厂类，**系统中类的个数将成对增加，在一定程度上增加了系统的复杂度**，有更多的类需要编译和运行，会给系统带来一些额外的开销
 - 由于考虑到系统的可扩展性，需要引入抽象层，在客户端代码中均使用抽象层进行定义，增加了系统的抽象性和理解难度，且在实现时可能需要用到DOM、反射等技术，增加了系统的实现难度