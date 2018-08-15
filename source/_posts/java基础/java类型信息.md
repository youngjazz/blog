---
title: java类型信息
date: 2017-03-05 18:45:12
tags: java基础
categories: 技术
---
## 运行时类型信息使得你在程序运行时发现和使用类型信息
我们如何使用类型信息呢？主要有两种方式：一是传统的**RTTI**，它假设在编译时我们直到所有的类型；二是**反射机制**，它允许我们在运行时发现和使用类的信息。
## 为什么使用RTTI（Run-Time Type Identification）
![这里写图片描述](http://p7b5cwgjy.bkt.clouddn.com/java_RTTI)
实际上他是把所有的对象当Object持有--会自动将结果转型为Shape。这是RTTI的最基本的使用形式，因为在java中，所有的类型都是在运行时检查的。这也是RTTI名字的含义，在运行时，识别一个对象的类型。
## class对象
类是程序的一部分，每个类都有一个Class对象。所有的类都是在第一次使用时，动态加载到JVM中的。

<!--more-->
```java
package type;

/**
 * Created by leon on 2017/2/25.
 */
public class SweetShop {
    public static void main(String[] args) {
        System.out.println("Inside main...");
        new Candy();
        System.out.println("After createing Candy");
        try {
            Class.forName("type.Gum");
        } catch (ClassNotFoundException e) {
            System.out.println("Couldn't find Gum");
        }
        System.out.println("After Class.forName(\"Gum\")");
        new Cookie();
        System.out.println("After creating Cookie");
    }
}

class Candy{
    static {
        System.out.println("Loading Candy");
    }
}

class Gum{
    static {
        System.out.println("loading Gum");
    }
}

class Cookie{
    static{
        System.out.println("Loading Cookie");
    }
}
```

```
Inside main...
Loading Candy
After createing Candy
loading Gum
After Class.forName("Gum")
Loading Cookie
After creating Cookie
```
所有的类的使用都是在第一次使用时，动态加载到JVM中的。当程序创建的第一个对类的静态成员的引用时，就会加载这个类。这个证明**构造器也是类的静态方法**，即使在构造器前没有使用static关键字。因此，用new操作符创建的新对象也会被当做对类的静态成员的引用。
java程序在它开始运行之前并非完全加载，其各个部分是在必需时才加载的。动态加载能力使能的行为，在诸如C++这样的静态加载的语言中很难或者根本不可能复制的。
类加载器首先会验证Class对象是否完整加载，一旦某各类的Class对象呗载入内存，它会被利用创建这个类的所有对象。
上面每一个类都有一个static字句，该字句在类的第一次加载时执行。
**Class.forName("...")** 是取得Class对象的引用的另一种便捷的方法，因为你不用为了获得Class引用而持有该类型的对象。但但是如果你已经拥有了一个感兴趣的对象，那就调用**getClass()**方法来获取Class引用了，这个方法属于Object对象的一部分，将返回给对象实际类型Class的引用。Class还包括很多其他有用的方法：
```java
package type;

/**
 * Created by leon on 2017/2/25.
 */
interface HashBatteries{}
interface Waterproof{}
interface Shoots{}
class Toy{
    Toy() {}
    Toy(int i){}
}
class FancyToy extends Toy{
    FancyToy() {
        super(1);
    }
}
public class ToyTesy {
    static void printInfo(Class cc){
        System.out.println("Class name:"+cc.getName()
                +" -- is Interface? :"+cc.isInterface());
        System.out.println("Simple name:"+cc.getSimpleName());
        System.out.println("Canonical name:"+cc.getCanonicalName());
    }

    public static void main(String[] args) {
        Class c = null;
        try {
            c = Class.forName("type.FancyToy");
        } catch (ClassNotFoundException e) {
            System.out.println("Can't find FancyToy");
            e.printStackTrace();
        }
        printInfo(c);
        for (Class aClass : c.getInterfaces()) {
            printInfo(aClass);
        }
        Object obj = null;
        Class up = c.getSuperclass();

        try {
            obj = up.newInstance();
        } catch (InstantiationException e) {
            e.printStackTrace();
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        }
        printInfo(obj.getClass());
    }
}
```

```
Class name:type.FancyToy -- is Interface? :false
Simple name:FancyToy
Canonical name:type.FancyToy
Class name:type.Toy -- is Interface? :false
Simple name:Toy
Canonical name:type.Toy
```
### 类字面常量
Java提供另一种方法来生成对Class对象的引用，即**类字面常量**，对上述程序来说就是Fancy.class 。这样做不仅简单而且更安全，因为在编译时就会受到检查（因此不需要try catch），并且它根除了对forName() 方法的调用，所以更高效。类字面常量不仅可以应用于普通的类，也可以应用于接口、数组、以及基本数据类型。
值得注意的是，使用“.class”来创建对Class对象的引用是，不会自动的初始化该Class对象。准备工作包含三个不走

 1. 加载。 类加载器执行，查找字节码，并从这些字节码中创建一个Class对象。
 2. 链接。 在连接阶段将验证类中的字节码，为静态域分配空间，并且如果必须的话，将解析这个类创建的对其他类的引用。
 3. 初始化。如果该类具有超类，则对其进行初始化，执行静态初始化器和静态初始化代码块。
初始化被延迟到了对静态方法（构造器隐式地是静态的）或者非静态域进行首次引用是才执行：
```java
package thingkingInJava.type;

import java.util.Random;

/**
 * Created by leon on 2017/3/5.
 */
class Initable{
    static final int staticFinal = 47;
    static final int staticFinal2 =ClassInitialization.rand.nextInt(1000);
    static {
        System.out.println("Initializing Initable");
    }
}

class Initable2{
    static int staticNonFinal = 147;
    static {
        System.out.println("Initializing Initable2");
    }
}

class Initable3{
    static int staticNonFinal = 74;
    static{
        System.out.println("Initializing Initable3");
    }
}

public class ClassInitialization {
    public static Random rand = new Random(47);

    public static void main(String[] args) {
        Class initable = Initable.class;
        System.out.println("After creating Initable ref");
        //Does not trigger initalization
        System.out.println(Initable.staticFinal);

        //Does trigger initalization
        System.out.println(Initable.staticFinal2);

        //Does trigger initalization
        System.out.println(Initable2.staticNonFinal);
        try {
            Class initable3 = Class.forName("thingkingInJava.type.Initable3");
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
        System.out.println("After creating Initable3 ref");
        System.out.println(Initable3.staticNonFinal);
    }
}
```

```
After creating Initable ref
47
Initializing Initable
258
Initializing Initable2
147
Initializing Initable3
After creating Initable3 ref
74
```
初始化实现了尽可能的“惰性”。从对initable引用的创建可以看出，仅使用.class语法获取的对类的引用不会引发初始化。但是为了产生Class引用，Class.forName()会立即就进行初始化，就像对initable3引用的创建中看到的。

