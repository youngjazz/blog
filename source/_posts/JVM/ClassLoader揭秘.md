---
title: ClassLoader揭秘
date: 2018-09-17 14:38:16
tags: JVM
categories: 技术
---

## 什么是Classloader

一个Java程序要想运行起来，首先需要经过编译成.class文件，然后创建一个环境（JVM）来加载字节码文件到内存中运行，而.class文件是怎么加载到JVM中去的呢？这就是Java Classloader做的事情了。

什么时候加载？

- new 操作
- Class.forName("包路径+类名")
- Class.forName("包路径+类名", classloader)
- classloader.loadclass("包路径+类名")

以上操作会触发类加载器去加载对应的路径去查找*.class, 并创建Class对象。

<!--more-->

## Java自带的Classloader

### BootstrapClassLoader

引导类加载器，又称启动类加载器，是最顶层的加载器，主要用来加载Java核心类，比如resouce.jar, rt.jar, charset.jar等，Sun的JVM中，执行java的命令使用-Xbootclasspath选项或是用-D选项指定sun.boot.class.path系统属性值可以指定附加的类，他不是java.lang.ClassLoader的子类，而是JVM自身实现的该类C语言实现，Java程序访问不到该加载器。通过下面代码可以查看该加载器加载了哪些jar包

```java
public void test(){
    URL[] urLs = Launcher.getBootstrapClassPath().getURLs();
    Arrays.stream(urLs).forEach(System.out::println);
}}
```

执行结果：

```java
file:/Library/Java/JavaVirtualMachines/jdk1.8.0_131.jdk/Contents/Home/jre/lib/resources.jar
file:/Library/Java/JavaVirtualMachines/jdk1.8.0_131.jdk/Contents/Home/jre/lib/rt.jar
file:/Library/Java/JavaVirtualMachines/jdk1.8.0_131.jdk/Contents/Home/jre/lib/sunrsasign.jar
file:/Library/Java/JavaVirtualMachines/jdk1.8.0_131.jdk/Contents/Home/jre/lib/jsse.jar
file:/Library/Java/JavaVirtualMachines/jdk1.8.0_131.jdk/Contents/Home/jre/lib/jce.jar
file:/Library/Java/JavaVirtualMachines/jdk1.8.0_131.jdk/Contents/Home/jre/lib/charsets.jar
file:/Library/Java/JavaVirtualMachines/jdk1.8.0_131.jdk/Contents/Home/jre/lib/jfr.jar
file:/Library/Java/JavaVirtualMachines/jdk1.8.0_131.jdk/Contents/Home/jre/classes
```

我们并没有在classpath里面指定这些类的路径，为什么还是能够加载到jvm中的，因为这是bootstrap来加载的。

### ExtClassLoader

扩展类加载器，主要负责加载java扩展类。默认加载JAVA_HOME/jre/lib/ext/目录下的所有jar或者由java.ext.dirs系统属性指定的jar包。放入这个目录下的jar对所有AppClassLoader可见（后面会知道ExtClassLoader是APPClassLoader的父类加载器）。那么ExtClassLoader在哪些地方加载类呢：

```java
System.out.println(System.getProperty("java.ext.dirs"));
```

```
/Users/leon/Library/Java/Extensions:/Library/Java/JavaVirtualMachines/jdk1.8.0_131.jdk/Contents/Home/jre/lib/ext:/Library/Java/Extensions:/Network/Library/Java/Extensions:/System/Library/Java/Extensions:/usr/lib/java
```

### AppClassLoader

系统类加载器，又称应用类加载器。它负责在JVM启动时加载来自命令中-classpath或者java.class.path系统属性或者CLASSPATH操作系统属性指定的jar类包和类路径。调用ClassLoader.getSystemClassLoader()可以获取类加载器，如果没有特别指定，用户自定义的任何类加载器都将该类加载器作为他的父加载器，通过调用ClassLoader的无参构造器：

```java
protected ClassLoader() {
    this(checkCreateClassLoader(), getSystemClassLoader());
}
```

执行以下代码可以获得classpath的加载路径

```java
Arrays.stream(System.getProperty("java.class.path").split(":")).forEach(System.out::println);
```

### 三种加载器的联系

![三种加载器联系](http://p7b5cwgjy.bkt.clouddn.com/%E4%B8%89%E7%A7%8D%E5%8A%A0%E8%BD%BD%E5%99%A8%E8%81%94%E7%B3%BB)

用户自定义的无参构造器的父类加载器是默认的AppCLassLoader加载器，而AppClassLoader的父类加载器是ExtClassLoader

```java
System.out.println(ClassLoader.getSystemClassLoader().);
ClassLoader parent = ClassLoader.getSystemClassLoader().getParent();
System.out.println(parent);
System.out.println(parent.getParent());
```

一般我们任务ExrClassloader的父类加载器是BootStrapClassLoader，但是它们之间并没有父子关系，只是在ExtClassLoader找不到要加载类时候回去委派BootStrap加载器去加载。上面的第四行代码打印出的就是null。

### 类加载器的原理

Java类的加载器使用的是委托机制，也就是子类加载器在加载一个类时会让其父类加载，为什么使用这种方式？因为这样可以避免重复加载，当父类加载了该类的时候，就没有必要子ClassLoader再加载一次。考虑到安全因素，如果不使用这种委托模式，那我们就可以随时使用自定义的String来动态替换Java核心api中定义的类型，这样会存在很大的安全隐患。双亲委派可以避免这种情况，因为String已经在启动时就被引导类（Bootstrap ClassLoader）加载，所以用户自定义的ClassLoader永远无法架子一个自己写的String，除非你改变了JDK中ClassLoader搜索类的默认算法。从源码中我们可以看出如何实现委派机制：

```java
protected Class<?> loadClass(String name, boolean resolve)
        throws ClassNotFoundException
    {
        synchronized (getClassLoadingLock(name)) {
            // First, check if the class has already been loaded
            Class<?> c = findLoadedClass(name);
            if (c == null) {
                long t0 = System.nanoTime();
                try {
                    if (parent != null) {
                        c = parent.loadClass(name, false);
                    } else {
                        c = findBootstrapClassOrNull(name);
                    }
                } catch (ClassNotFoundException e) {
                    // ClassNotFoundException thrown if class not found
                    // from the non-null parent class loader
                }

                if (c == null) {
                    // If still not found, then invoke findClass in order
                    // to find the class.
                    long t1 = System.nanoTime();
                    c = findClass(name);

                    // this is the defining class loader; record the stats
                    sun.misc.PerfCounter.getParentDelegationTime().addTime(t1 - t0);
                    sun.misc.PerfCounter.getFindClassTime().addElapsedTimeFrom(t1);
                    sun.misc.PerfCounter.getFindClasses().increment();
                }
            }
            if (resolve) {
                resolveClass(c);
            }
            return c;
        }
    }
```

分析源码我们知道，首先从JVM缓存总查找类，如果已经加载过，直接从JVM缓存返回，否则看当前类加载器是否有父加载器，如果有则委托父加载器去加载，如果没有父加载器就委托BootstrapClassloader去加载，如果还没有找到，这才调用当前classLoader类的findClass方法。**自己实现的载入策略需要覆盖ClassLoader的findClass方法或覆盖loadClass方法来实现**。

```java
/**
     * Finds the class with the specified <a href="#name">binary name</a>.
     * This method should be overridden by class loader implementations that
     * follow the delegation model for loading classes, and will be invoked by
     * the {@link #loadClass <tt>loadClass</tt>} method after checking the
     * parent class loader for the requested class.  The default implementation
     * throws a <tt>ClassNotFoundException</tt>.
     *
     * @param  name
     *         The <a href="#name">binary name</a> of the class
     *
     * @return  The resulting <tt>Class</tt> object
     *
     * @throws  ClassNotFoundException
     *          If the class could not be found
     *
     * @since  1.2
     */
protected Class<?> findClass(String name) throws ClassNotFoundException {
    throw new ClassNotFoundException(name);
}
```



### 如何构造三种类加载器的结构

下面从源码来分析JVM如何构建内置classloader的，具体rt.jar 包里面sum.misc.Launcher类：

```java
public Launcher(){  
        ExtClassLoader localExtClassLoader;  
        try 
        {  //首先创建了ExtClassLoader
          localExtClassLoader = ExtClassLoader.getExtClassLoader();  
        }  
        catch (IOException localIOException1)  
        {  
          throw new InternalError("Could not create extension class loader");  
        }  
        try 
        {  //然后以ExtClassloader作为父加载器创建了AppClassLoader
          this.loader = AppClassLoader.getAppClassLoader(localExtClassLoader);  
        }  
        catch (IOException localIOException2)  
        {  
          throw new InternalError("Could not create application class loader");  
        }  //这个是个特殊的加载器后面会讲到，这里只需要知道默认下线程上下文加载器为appclassloader
        Thread.currentThread().setContextClassLoader(this.loader);  
 
        ................
}
```

下面看下ExtClassLoader.getExtClassLoader()的代码

```java
public static Launcher.ExtClassLoader getExtClassLoader() throws IOException {
    final File[] var0 = getExtDirs();

    try {
        return (Launcher.ExtClassLoader)AccessController.doPrivileged(new PrivilegedExceptionAction<Launcher.ExtClassLoader>() {
            public Launcher.ExtClassLoader run() throws IOException {
                int var1 = var0.length;

                for(int var2 = 0; var2 < var1; ++var2) {
                    MetaIndex.registerDirectory(var0[var2]);
                }

                return new Launcher.ExtClassLoader(var0);
            }
        });
    } catch (PrivilegedActionException var2) {
        throw (IOException)var2.getException();
    }
}
```

getExtDirs() 是使用了System.getProperty("java.ext.dirs")

AppClassLoader.getAppClassLoader的代码

```java
public static ClassLoader getAppClassLoader(final ClassLoader var0) throws IOException {
    //AppClassLoader加载路径为java.class.path
    final String var1 = System.getProperty("java.class.path");
    final File[] var2 = var1 == null ? new File[0] : Launcher.getClassPath(var1);
    return (ClassLoader)AccessController.doPrivileged(new PrivilegedAction<Launcher.AppClassLoader>() {
        public Launcher.AppClassLoader run() {
            URL[] var1x = var1 == null ? new URL[0] : Launcher.pathToURLs(var2);
            return new Launcher.AppClassLoader(var1x, var0);
        }
    });
}
```

总结下，Java应用启动过程首先是BootstrapClassLoader加载rt.jar里面的Launcher类，而该类内不是用BootstrapClassloader加载器构建和初始化Java中三类加载和线程上下文加载器，然后根据不同的 场景使用这些加载器去自己的类查找路径去加载类。

## 一种特殊的类加载器 ContextClassLoader

阅读过tomcat源码的话对于ContextClassLoader一定很熟悉的：

```java
//获取当前线程上下文类加载器
ClassLoader classloader = Thread.currentThread().getContextClassLoader();
try{
    Thread.currentThread().setContextClassLoader(targetTcc1);
    doSomething();
}finally{
    //设置当前线程上下文类加载器为原始类加载器
    Thread.currentThread().setContextClassLoader(classloader);
}
```

doSomething 里面调用的是Thread.currentThread().getContextClassLoader()获取当前线程上下文类加载器来做些事情。这其中的奥秘和使用场景是什么？

我们知道java默认的类加载机制是委托机制，但是这种加载顺序**有时不能正常工作**，通常发生在有些JVM核心代码需要动态加载有程序开发者提供的资源时。以JNDI为例，它的核心内容和接口在rt.jar中引导类中实现了，但是这些JNDI可能会加载由独立厂商实现和部署在classpath的JNDI提供者。**这个场景要求一个父类加载器去加载一个他的子类加载器中可见的类**。这个例子中父类加载器指的是加载rt.jar的bootstrap加载器，子类加载器为AppClassLoader。此时通常的J2SE**委托机制就不能胜任**了。解决办法是让JNDI核心使用线程上下文加载器（默认线程上下文加载器为AppCLassLoader）

## Tomcat ClassLoader



