---
title: java类型信息
date: 2018-03-05 21:45:12
tags: java基础
categories: 技术
---



# 1.为什么关心Java8

- 计算背景变化： 多核时代&处理大数据集
- 改进：函数式比命令式更适合新的体系架构
- 核心特征： Lambda、Stream、默认方法

## Java的演变
## Java中的函数
### 方法和Lambda作为一等公民
- Java8的第一个新功能**方法引用**
```java
File[] hiddenFiles = new File(*.*).listsFiles(new FileFilter(){
  public boolean accept(File file){
    return file.isHidden();
  }
})
```
Java8的写法：
```java
File[] hiddenFiles = new File(*.*).listFiles(File::isHidden);
```
你已经有的函数isHidden，只需要使用java8的方法引用语法::(即将这个方法作为值传给listFiles方法)。好处是，你的代码更接近问题的陈诉。与对象引用相似（new一个对象），这里传递的是一个方法引用（通过File::isHiddenc创建了一个方法引用），你同样可以传递它。

- Lambda匿名函数
  除了允许命名函数成为一等公民外，Java8还体现了更广义的将函数作为值的思想，包括匿名函数。比如，你可以将（int x）-> x + 1作为参数传给另一个函数，表示接受一个给定的x，返回x+1值的函数。这样做有什么必要呢？因为我们完全可以新建一个MathUtil类添加add1方法，然后传递MathUtil::add1嘛！确实可以这样，但要是你没有这样的类和方法可用呢，新的Lambda语法更简洁。

### 传递代码
```java
static List<Apple> filterApples(List<Apple> inventory, Predicate<Apple> p){
  List<Apple> result = new ArrayList<>();
  for(Apple apple: inventory){
    if(p.test(apple)){
      result.add(apple);
    }
  }
  return result;
}
```
方法希望接受一个Predicate<Apple>参数，**谓词**在数学上常常用来代表一个类似函数的东西。
使用的话就是可以这样写了：
```java
filterApples(inventory, Apple::isHeavy);
filterApples(inventory, Apple::isGreen)
```

### 传递Lambda
filterApples(inventory, (Apple a) -> "green".quals(a.getColor()));
如果Lambda的长度多余几行，他就不那么一目了然了，往往需要一个具体描述名称的方法。实际上Java8已经提供了过滤通用方法比如：
statuc <T> Collection<T> filter(Collection<T> c, Predicate<T> p);
可以直接写作
filter(inventory, (Apple a) -> "green".equals(a.getColor()))

## 流
和Collections处理方式不同的是，COllection是外部迭代，而使用Stream完全不用操心循环的事情，数据处理完全在库内部进行的，这种思想称为内部迭代。
另一个问题是使用单核下Collection API很难处理大量的数据，而使用多线程访问共享变量如果没有协调好数据很可能会被意外的改变。
Collection主要是为了存储和访问数据，而Stream则主要用于描述对数据的计算。函数式变成中的函数主要有两成意思：：把函数当做一等值；执行时元素之间无互动。

## 默认方法
我们在使用inventory.stream().filter((Apple a) -> a.getWeight > 150).collect(toList())
时，List<T>并没有stream或者paralelStream方法，Collection<T>接口也没有。那么现在需要在Collection接口中加入stream方法，所有的实现类都要去去实现该接口，简直如噩梦一般。Java8设计者引入了默认方法，如果实现类没有提供实现的方法签名由接口提供实现（因此就有了默认方法）。这样就给接口设计者一个扩充接口的方式而不破坏现有代码。比如我们扩展Collections的sort方法
```java
default void sort(Comparator<? super E> c){
  Collections.sort(this, c);
}
```
## 来自函数式编程的其他好思想
- 引入Optional<T>类来避免NullPointer异常。它是一个容器对象，可以包含也可以不包含一个值。
- 模式匹配。在数学中很常见，如 f(0) = 1； f(n) = n*f(n-1) 。Java8支持不是很好，可以参见JVM的类Java语言Scala



# 2. 通过行文参数化传递代码

如果我们想在一堆苹果中选出我们想要的特征的苹果：

```java
List<Apple> filter(List<Apple> apples, String color，int weight， boolean flag){
	//for each
}
```

这是一种很笨拙的做法。

```java
public interface Predicate<T>{
    boolean test(T t);
}

public static <T> List<T> filter(List<T> list, Predicate<T> p){
    List<T> result = new ArrayList<>();
    for(T e:list){
        if(p.test(e)){
            result.add(e);
        }
    }
    return result;
}

//将参数行为化，又抽象了List，在借助Lambda
List<Apple> reApples = filter(inventory, (Apple apple)->"red".equal(apple.getColor()));
List<Integer> evenNumbers = filter(numbers, (Integer i)->i % 2 == 0);
//在灵活性和简洁性上找到了最佳平衡点
```



### 用Runnable执行代码块

Lambda版本

```java
Runnable r1 = ()->System.out.println("Hello Java!")
Thread t = new Thread({()->System.out.println("Hello World!")});
```

==行为参数化，就是一个方法接受不同的行为作为参数， 并在内部使用它们，完成不同行为的能力。==行为参数化，可以让代码更好的适应不断变化的需求，减轻未来的工作量。



# 3.Lambda表达式

Lambda表达式有三部分组成：

- 参数列表
- 箭头
- Lambda主体