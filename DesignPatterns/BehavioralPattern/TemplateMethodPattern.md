## 模板方法模式

### 定义
 
定义一个操作中的算法框架，而将一些步骤延迟到子类中，使得子类可以不改变一个算法的结构即可重定义该算法的某些特定步骤。

模板方法模式是类的行为模式。准备一个抽象类，将部分逻辑以具体方法以及具体构造函数的形式实现，然后声明一些抽象方法来迫使子类实现剩余的逻辑。不同的子类可以以不同的方式实现这些抽象方法，从而对剩余的逻辑有不同的实现。这就是模板方法模式的用意。

### 模式中的角色

**AbstractClass(抽象类)** : 在抽象类中定义了一系列基本操作(PrimitiveOperations)，这些基本操作可以是具体的，也可以是抽象的，每一个基本操作对应算法的一个步骤，在其子类中可以重定义或实现这些步骤。同时，在抽象类中实现了一个模板方法(Template Method)，用于定义一个算法的框架，模板方法不仅可以调用在抽象类中实现的基本方法，也可以调用在抽象类的子类中实现的基本方法，还可以调用其他对象中的方法。所以模板方法模式中的抽象层只能是抽象类，而不是接口。

**ConcreteClass（具体子类）**：它是抽象类的子类，用于实现在父类中声明的抽象基本操作以完成子类特定算法的步骤，也可以覆盖在父类中已经实现的具体基本操作。

### 使用场景 
 - 对一些复杂的算法进行分割，将其算法中固定不变的部分设计为模板方法和父类具体方法，而一些可以改变的细节由其子类来实现。即：一次性实现一个算法的不变部分，并将可变的行为留给子类来实现。
 - 各子类中公共的行为应被提取出来并集中到一个公共父类中以避免代码重复。
 - 需要通过子类来决定父类算法中某个步骤是否执行，实现子类对父类的反向控制。
 

### 具体实现代码

**抽象类**

```java
    /**
     *  抽象类
     */
    abstract class AbstractPerson {
        //抽象类定义整个流程骨架
        public void prepareGotoSchool(){
            dressUp();
            eatBreakfast();
            takeThings();
        }
        //以下是不同子类根据自身特性完成的具体步骤
         abstract void dressUp();
         abstract void eatBreakfast();
         abstract void takeThings();
    }
```
**具体实现类**


```java
    /**
     *  具体实现类
     */
    public class Student extends AbstractPerson{
        @Override
        protected void dressUp() {
            System.out.println("穿校服");
        }
        @Override
        protected void eatBreakfast() {
            System.out.println("吃妈妈做好的早饭");
        }
    
        @Override
        protected void takeThings() {
            System.out.println("背书包，带上家庭作业和红领巾");
        }
    }
```


```java
    /**
     * 具体实现类.
     */
    public class Teacher extends AbstractPerson{
        @Override
        protected void dressUp() {
            System.out.println("穿工作服");
        }
        @Override
        protected void eatBreakfast() {
            System.out.println("做早饭，照顾孩子吃早饭");
        }
    
        @Override
        protected void takeThings() {
            System.out.println("带上昨晚准备的考卷");
        }
    }
```
**测试代码**

```java
        Student student = new Student();
        student.prepareGotoSchool();

        Teacher teacher  = new Teacher();
        teacher.prepareGotoSchool();
```

输出结果过于简单就不贴出来了

### 优点

 - 在父类中形式化地定义一个算法，而由它的子类来实现细节的处理，在子类实现详细的处理算法时并不会改变算法中步骤的执行次序。

 - 模板方法模式是一种代码复用技术，它在类库设计中尤为重要，它提取了类库中的公共行为，将公共行为放在父类中，而通过其子类来实现不同的行为，它鼓励我们恰当使用继承来实现代码复用。

 - 可实现一种反向控制结构，通过子类覆盖父类的钩子方法来决定某一特定步骤是否需要执行。

 - 在模板方法模式中可以通过子类来覆盖父类的基本方法，不同的子类可以提供基本方法的不同实现，更换和增加新的子类很方便，符合单一职责原则和开闭原则。


**钩子方法**

抽象父类
```java
    public abstract class AbstractClass {
    
        public abstract boolean isOpen();
    
        public final void operating() {
            if(isOpen()) {
                System.out.println("钩子方法开启");
            }else {
                System.out.println("钩子方法关闭");
            }
        }
    }
```

实现类

```java
    public class AchieveClass extends AbstractClass {
    
      //钩子方法能挂在到operating能干预到operating业务逻辑
        @Override
        public boolean isOpen() {
            return true;
        }
    
        public static void main(String[] args) {
            AchieveClass ac = new AchieveClass();
            ac.operating();
        }
    
    }
```
通过子类覆盖父类的钩子方法来决定某一特定步骤是否需要执行。











