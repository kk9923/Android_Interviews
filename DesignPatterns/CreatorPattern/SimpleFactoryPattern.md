## 简单工厂模式
### 定义

简单工厂模式(Simple Factory Pattern)：又称为静态工厂方法(Static Factory Method)模式，它属于类创建型模式。在简单工厂模式中，可以根据参数的不同返回不同类的实例。简单工厂模式专门定义一个类来负责创建其他类的实例，被创建的实例通常都具有共同的父类。

### 核心模块

 **Factory(工厂角色)**
 
 工厂角色即工厂类，它是简单工厂模式的核心，负责实现创建所有产品实力的内部逻辑;工厂类通过静态的方式可以直接被外界调用，创建出所需的产品对象，返回值为抽象的产品类型。
 
 **Product(抽象产品角色)**
 
 工厂类所创建的产品的父类，负责描述所有实例所共有的公共接口，封装了各种产品对象的公有方法。
 
 **(ConcreteProduct具体产品角色)**
 
 实现抽象产品的某个具体产品对象，每个产品角色都继承自抽象产品角色，都需要实现在抽象产品中声明的抽象方法。
 
 ### 简单工厂使用实例
 以玩具工厂造玩具为例使用简单工厂模式写代码，其中Itoy为抽象的产品角色，ChildToy，WoMenToy，ManToy等就是具体的产品角色，ToyFactory就是工厂角色。
 
```java
    /**
     * 玩具类 -----  抽象的产品角色
     */
    interface IToy {
        /**
         * 名字
         */
        String getName();
        /**
         * 价格
         */
        float price();
    }
    
    /**
     * 具体的玩具类--- 儿童玩具  -----   具体的产品角色
     */
    public class ChildToy implements IToy {
        @Override
        public String getName() {
            return "ChildToy";
        }
    
        @Override
        public float price() {
            return 20;
        }
    }
    /**
     * 具体的玩具类--- 大人玩具   -----   具体的产品角色   
     */
    public class ManToy implements IToy {
        @Override
        public String getName() {
            return "ManToy";
        }
    
        @Override
        public float price() {
            return 100;
        }
    }
```
工厂类
```java
    /**
    * 简单工厂的工厂类
    */
   public class ToyFactory {
       public static  void    creatToy  (String toyName){
           IToy iToy;
           if ("ManToy".equals(toyName)){
               iToy =  new ManToy();
               System.out.println("玩具Name =  "  + iToy.getName() + "    价格 =   " + iToy.price()  );
           }else  if ("WoMenToy".equals(toyName)){
               iToy =  new WoMenToy();
               System.out.println("玩具Name =  "  + iToy.getName() + "    价格 =   " + iToy.price()  );
           }else  if ("ChildToy".equals(toyName)){
               iToy =  new ChildToy();
               System.out.println("玩具Name =  "  + iToy.getName() + "    价格 =   " + iToy.price()  );
           }
           else {
               System.out.println("无法生成该玩具");
           }
       }
       /**
       *    通过反射获取类的实例
       */
       public static <T extends IToy> IToy creatToy(Class<T> clazz) {
           IToy iToy ;
           try {
               iToy = clazz.newInstance();
               return iToy;
           }catch (Exception e) {
               e.printStackTrace();
               throw new UnknownError(e.getMessage());
           }
       }
   }
```
客户调用

```java
        ToyFactory.creatToy("ManToy");
        ToyFactory.creatToy("ChildToy");
   
        System.out.println("----------------------------------------------");
   
        IToy woMenToy = ToyFactory.creatToy(ChildToy.class);
        IToy manToy = ToyFactory.creatToy(ManToy.class);

```

### 简单工厂模式的优点

**优点**
 - 客户端可以免除直接创建对象的职责，只需要使用对象即可，简单工厂模式实现了对象创建和使用的分离。
 - 外界不再需要关心如何创造各种具体的产品，只要提供一个产品的名称作为参数传给工厂，就可以直接得到一个想要的产品对象，并且可以按照接口规范来调用产品对象的所有功能（方法）

**缺点**
 - 系统扩展困难，一旦添加新产品就不得不修改工厂 if else 逻辑，违背了“开放-关闭原则”中的对修改关闭的准则了。
 - 工厂类负责所有对象的创建逻辑,违反了高内聚的责任分配原则,当工厂类出现问题的时候，整个系统都要受到影响。一般只在工厂类负责创建的对象比较少时使用.
 - 简单工厂模式由于使用了静态工厂方法，所以工厂角色无法形成基于继承的等级结构。
