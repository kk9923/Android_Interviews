## 抽象工厂模式
### 定义
为创建一组相关或是相互依赖的对象提供一个接口，而不需要指定他们的具体类。抽象工厂模式又称为Kit模式，属于对象创建型模式。

与工厂方法模式的区别在于工厂方法模式针对的是**一个产品等级结构**；抽象工厂模式则是针对的**多个产品等级结构**。工厂方法模式提供的所有产品都是实现同一个接口，而抽象工厂模式所提供的产品是实现自不同的接口。

**概念**

**产品等级结构**： 产品等级结构就是类的继承结构.比如一个抽象类是轮胎，普通轮胎和越野轮胎都是它的子类，**则抽象类与这些子类构成了产品等级结构**。

**产品族**： 在抽象工厂模式中，产品族是指由同一个工厂生产的，位于不同产品等级结构中的一组产品。比如汽车工厂生产出来的轮胎、发动机，其中轮胎就属于轮胎产品等级中，发动机就属于发动机产品等级中。

**当生产多个位于不同产品等级结构中属于不同类型的具体产品时需要使用抽象工厂模式。**

**工厂方法模式针对的是一个产品等级结构，而抽象工厂模式则需要面对多个产品等级结构**，一个工厂等级结构可以负责多个不同产品等级结构中的产品对象的创建 。当一个工厂等级结构可以创建出分属于不同产品等级结构的一个产品族中的所有对象时，抽象工厂模式比工厂方法模式更为简单、有效率。

### 核心模块

**AbstractFactory(抽象工厂角色)**

声明一组用于创建一族产品的方法，每个方法对应一组产品(位于不同的产品等级结构中).

**ConcreteFactory(具体工厂角色)**

实现了抽象工厂中声明的创建产品的方法，生成一组具体的产品，这些产品构成了产品族，同时每一个产品都位于某个产品等级结构中。

**AbstractProduct(抽象产品角色)** 

它为每个产品声明接口，声明每个产品所具有的业务方法。

**Product(具体产品角色)** 

定义具体工厂生产的的具体产品对象。实现抽象产品接口中声明的业务方法。

 ### 抽象工厂使用实例
 以汽车工厂造制造轮胎和发动机为例使用抽象工厂模式写代码，其中IFactory为抽象工厂角色，
 NormalFactory，NBFactory就是具体的工厂角色，ITire和IEngine就是抽象产品角色，NormalTire、SUVTire 、NormalEngine、ImportEngine则属于具体产品角色。
 
 **抽象产品类**
 ```java
 /**
  *  抽象产品角色  ----   发动机产品结构等级   
  */
 public interface IEngine {
     void createEngine();
 }
 
 /**
  * 抽象产品角色   -----  轮胎产品等级结构   
  */
 public interface ITire {
     void createTire();
 }
 ```
 **具体产品类**
 ```java
 /**
  * 具体产品角色  --  轮胎产品等级结构
  */
 public class NormalTire implements ITire {
     @Override
     public void createTire() {
         System.out.println("生产普通轮胎");
     }
 }
 /**
  * 具体产品角色  --  轮胎产品等级结构
  */
 public class SUVTire implements ITire {
     @Override
     public void createTire() {
         System.out.println("生产越野轮胎");
     }
 }

/**
 * 具体产品角色  ---  发动机产品等级结构
 */
public class NormalEngine implements IEngine {
    @Override
    public void createEngine() {
        System.out.println("生产国产发动机");
    }
}
/**
 *  具体产品角色 ---  发动机产品等级结构
 */
public class ImportEngine implements IEngine {
    @Override
    public void createEngine() {
        System.out.println("生产进口发动机");
    }
}

 ```
 
 **工厂类**
 ```java
 /**
  *  抽象工厂角色     定义一组用于创建 属于不同产品等级结构产品的方法
  */
 public abstract class  IFactory {
     /**
      * 生产轮胎
      * @return
      */
     public abstract ITire createTire();
 
     /**
      * 生产发动机
      * @return
      */
     public abstract IEngine createEngine();
 }

 ```
  **具体工厂类**
  ```java
  /**
   *  具体的工厂角色 生成一组具体的产品，  每个产品都位于某个产品等级结构中(继承结构)
   */
  public class NormalFactory extends IFactory {
      @Override
      public ITire createTire() {
          return new NormalTire();
      }
  
      @Override
      public IEngine createEngine() {
          return new NormalEngine();
      }
  }
  
  /**
   *  具体的工厂角色 生成一组具体的产品，  每个产品都位于某个产品等级结构中(继承结构)
   */
  public class NBFactory extends IFactory {
      @Override
      public ITire createTire() {
          return new SUVTire();
      }
  
      @Override
      public IEngine createEngine() {
          return new ImportEngine();
      }
  }
  
  ```
  **客户端调用**
  ```java
          IFactory normalFactory = new NormalFactory();
          normalFactory.createTire().createTire();
          normalFactory.createEngine().createEngine();
  
          System.out.println("-------------------------------");
  
          IFactory nbFactory = new NBFactory();
          nbFactory.createTire().createTire();
          nbFactory.createEngine().createEngine();
          
          
          //  运行结果
          
           *   生产普通轮胎
           *   生产国产发动机
           *   -------------------------------
           *   生产越野轮胎
           *   生产进口发动机
          
  ```
  ### 工厂方法模式的优缺点
   
   **优点**
   - 显著的优点是分离接口与实现，隔离了具体类的生成，客户不需要知道具体的创建对象是谁。
   - 增加新的具体工厂和产品族很方便，因为一个具体的工厂实现代表的是一个产品族，无须修改已有系统，符合“开闭原则”。
   
   **缺点**
   -  类文件的爆炸性增加。
   -  不太容易增加新的产品等级结构，当我们增加一个新的产品类就需要修改抽象工厂，那么所有的具体工厂都要修改.会带来较大的不便。
   -  增加新的工厂和产品族容易，增加新的产品等级结构麻烦
   
   
   **简单工厂 ： 用来生产同一等级结构中的任意产品。（不支持拓展增加产品）**
   
   **工厂方法 ：用来生产同一等级结构中的固定产品。（支持拓展增加产品）**   
   
   **抽象工厂 ：用来生产不同产品族的全部产品。（不支持拓展增加产品；支持增加产品族）**  
  
  