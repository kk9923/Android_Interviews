## 原型模式
### 定义
用原型实例指定创建对象的种类，并通过拷贝这些原型创建新的对象。原型模式是一种对象创建型模式

原型模式主要用于对象的复制，它的核心是就是类图中的原型类Prototype.

Prototype类需要具备以下两个条件：
 - 实现Cloneable接口。在java语言有一个Cloneable接口，它的作用只有一个，就是在运行时通知虚拟机可以安全地在实现了此接口的类上使用clone方法。在java虚拟机中，只有实现了这个接口的类才可以被拷贝，否则在运行时会抛出CloneNotSupportedException异常
 - 重写Object类中的clone方法。Java中，所有类的父类都是Object类，Object类中有一个clone方法，作用是返回对象的一个拷贝，但是其作用域protected类型的，一般的类无法调用，因此，Prototype类需要将clone方法的作用域修改为public类型。

**使用场景**

- 创建新对象成本较大（如初始化需要占用较长的时间，占用太多的CPU资源或网络资源），新的对象可以通过原型模式对已有对象进行复制来获得，如果是相似对象，则可以对其成员变量稍作修改。
- 一个对象需要提供给其他对象访问，而且各个调用者可能都需要修改其值时，可以考虑使用原型模式拷贝多个对象，供调用者使用，即保护性拷贝。

需要注意：通过实现Cloneable接口的原型模式在调用clone函数构造实例时并不一定比通过new操作速度快，只有当通过new构造对象较为耗时或者说成本较高时，通过clone方法才能够获得效率上的提升。因此，在使用Cloneable时需要考虑构建对象的成本以及做一些效率上的测试。

**注意事项**
 - **使用原型模式复制对象不会调用类的构造方法**。因为对象的复制是通过调用Object类的clone方法来完成的，它直接在内存中复制数据，因此不会调用到类的构造方法。不但构造方法中的代码不会执行，甚至连访问权限都对原型模式无效。还记得单例模式吗？单例模式中，只要将构造方法的访问权限设置为private型，就可以实现单例。但是**clone方法直接无视构造方法的权限，所以，单例模式与原型模式是冲突的**，在使用时要特别注意。
 - 深拷贝与浅拷贝。**Object类的clone方法只会拷贝对象中的基本的数据类型，对于数组、容器对象、引用对象等都不会拷贝，这就是浅拷贝(String类型也是浅拷贝)。如果要实现深拷贝，必须将原型模式中的数组、容器对象、引用对象等另行拷贝**。

### 原型模式的简单实现

```java
public class WordDocument implements Cloneable {
    // 文本信息
    private String mText;
    //  图片列表集合
    private ArrayList<String> mImages = new ArrayList<>();

    /**
     *  实现Cloneable接口，重写clone()方法
     */
    @Override
    public WordDocument clone() throws CloneNotSupportedException {
        WordDocument wordDocument = (WordDocument) super.clone();
        return wordDocument;
    }
    public String getText() {
        return mText;
    }
    public void setText(String text) {
        mText = text;
    }
    public ArrayList<String> getImages() {
        return mImages;
    }
    public void addImages(String image) {
        this.mImages.add(image);
    }
    /**
     *  打印文档内容
     */
    public void showWordDocument(){
        System.out.println("Text =  "  +  mText );
        for (String image : mImages) {
            System.out.println(" image =  "  +   image);
        }
    }
}
```
测试代码

```java
        //   构建文档对象
        WordDocument originDoc = new WordDocument();
        originDoc.setText("文档编号一");
        originDoc.addImages("图片a");
        originDoc.addImages("图片b");
        originDoc.showWordDocument();
        System.out.println("---------------------------------------");
        //   以原始文档对象为原型，进行拷贝
        try {
            WordDocument copyDoc = originDoc.clone();
            copyDoc.showWordDocument();
            System.out.println("---------------------------------------");
            String resultString= originDoc.getText() == copyDoc.getText() ? "String是浅拷贝的" : "String是深拷贝的";
            System.out.println(resultString);
            System.out.println("---------------------------------------");
            String resultArrayList = originDoc.getImages() == copyDoc.getImages() ? "ArrayList是浅拷贝的" : "ArrayList是深拷贝的";
            System.out.println(resultArrayList);
            System.out.println("---------------------------------------");
            copyDoc.setText("修改过的文档编号一");
            copyDoc.addImages("新增图片c");
            copyDoc.showWordDocument();
            System.out.println("---------------------------------------");
            originDoc.showWordDocument();

        } catch (CloneNotSupportedException e) {
            e.printStackTrace();
        }

```
输出结果：

``` java
     Text =  文档编号一
     image =  图片a
     image =  图片b
    ---------------------------------------
     Text =  文档编号一
     image =  图片a
     image =  图片b
    ---------------------------------------
     String是浅拷贝的
    ---------------------------------------
     ArrayList是浅拷贝的
    ---------------------------------------
     Text =  修改过的文档编号一
     image =  图片a
     image =  图片b
     image =  新增图片c
    ---------------------------------------
     Text =  文档编号一
     image =  图片a
     image =  图片b
     image =  新增图片c


```

分析：copyDoc是通过originDoc.clone创建的，通过比较 originDoc.getText() == copyDoc.getText() 发现两对String字符串的地址值是相同的，说明String是浅拷贝,但String是字符串常量，调用 copyDoc.setText("修改过的文档编号一")代码后，会重新创建字符串对象"修改过的文档编号一"，并将copyDoc中mText的引用指向新的字符串地址。

对于ArrayList，说明拷贝前后两者指向的都是同一块内存区域，当copyDoc.addImages("新增图片c")新增一张图片时，造成originDoc中的mImages集合的引用也会发生变化，所以两个对象中的图片都改变了。**其实上面的原型模式实际上是一个浅拷贝**。

要解决上述问题的办法就是采用深拷贝，在拷贝对象的同时，对于引用型的字段也要采用拷贝的形式，而不是单纯的引用。

修改代码如下：

```java
    @Override
    public WordDocument clone() throws CloneNotSupportedException {
        WordDocument wordDocument = (WordDocument) super.clone();
        wordDocument.mImages = (ArrayList<String>) this.mImages.clone();
        return wordDocument;
    }
```
将copyDoc.mImages指向this.mImages的一份拷贝，而不是this.mImages本身，修改后的运行结果如下：
```
     Text =  文档编号一
     image =  图片a
     image =  图片b
    ---------------------------------------
     Text =  文档编号一
     image =  图片a
     image =  图片b
    ---------------------------------------
     String是浅拷贝的
    ---------------------------------------
     ArrayList是深拷贝的
    ---------------------------------------
     Text =  修改过的文档编号一
     image =  图片a
     image =  图片b
     image =  新增图片c
    ---------------------------------------
     Text =  文档编号一
     image =  图片a
     image =  图片b

```
这样修改copyDoc中的mImages就不会影响到originDoc的mImages。

### 总结

原型模式本质上就是对象拷贝,使用原型模式可以解决构建复杂对象的资源消耗问题，能够在某些场景下提升创建对象的效率。还有一个重要的用途就是保护性拷贝，也就是某个对象对外可能是只读的，为了防止外部对这个只读对象进行修改，通常可以通过返回一个对象拷贝的形式实现只读限制。

**优点**

 - 原型模式实在内存中二进制流的拷贝，要比直接new一个对象性能好的多，特别是要在一个循环体内产生大量的对象时，原型模式可以更好的体现其优点。

**缺点**

 - 直接在内存中拷贝，对象的构造函数时不会执行的。
 - 在实现深克隆时需要编写较为复杂的代码，而且当对象之间存在多重的嵌套引用时，为了实现深克隆，每一层对象对应的类都必须支持深克隆比较麻烦

### 参考文档

 《Android源码设计模式》

[详解Java中的clone方法 -- 原型模式](https://blog.csdn.net/zhangjg_blog/article/details/18369201)





