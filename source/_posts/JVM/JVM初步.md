---
title: JVM初步
date: 2018-10-11 14:38:16
tags: JVM
categories: 技术
---



## jvm初体验 - 堆内存溢出

```java
/**
 * -Xms32M -Xmx32M -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=./
 * @author leon
 */
public class HeapOOM {
    public static void main(String[] args) {

        //模拟堆内存溢出
        List<Student> demoList = new ArrayList<Student>();
        while (true){
            demoList.add(new Student());
        }
    }
    static class Student{}
}
```

<!--more-->

在IDE中将Main的虚拟机属性设置为 `-XX:+HeapDumpOnOutOfMemoryError -Xms20m -Xmx20m`
运行得到

```
java.lang.OutOfMemoryError: Java heap space
Dumping heap to java_pid94659.hprof ...
Exception in thread "main" java.lang.OutOfMemoryError: Java heap space
	at java.util.Arrays.copyOf(Arrays.java:3210)
	at java.util.Arrays.copyOf(Arrays.java:3181)
	at java.util.ArrayList.grow(ArrayList.java:261)
	at java.util.ArrayList.ensureExplicitCapacity(ArrayList.java:235)
	at java.util.ArrayList.ensureCapacityInternal(ArrayList.java:227)
	at java.util.ArrayList.add(ArrayList.java:458)
	at top.neoman.test1.Main.main(Main.java:16)
Heap dump file created [27843280 bytes in 0.180 secs]
```

可以使用mat来分析堆导出文件



### 非堆区内存溢出

```java
public class StackTest {
    public static void main(String[] args) {
        new StackTest().test();
    }

    //模拟栈溢出
    private void test(){
        System.out.println("方法执行...");
        test();
    }
}
```

```bash
*** java.lang.instrument ASSERTION FAILED ***: "!errorOutstanding" with message transform method call failed at JPLISAgent.c line: 844
*** java.lang.instrument ASSERTION FAILED ***: "!errorOutstanding" with message transform method call failed at JPLISAgent.c line: 844
Exception in thread "main" java.lang.StackOverflowError
```

```java
/**
 * 模拟metaSpace内存溢出
 * -XX:MetaspaceSize=10M -XX:MaxMetaspaceSize=10M -verbose:class
 * Created by leon on 2018-12-02
 */
public class MetaSpaceTest{

    public static void main(String[] args) {
        try {
            Thread.sleep(10000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        int i = 0;
        while (true) {
            System.out.println(i++);
            
            //Enhancer依赖于cglib
            Enhancer enhancer = new Enhancer();
            enhancer.setSuperclass(MetaSpaceTest.class);
            enhancer.setUseCache(false);
            enhancer.setCallback(
                    new MethodInterceptor() {
                        @Override
                        public Object intercept(Object o, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {
                            return methodProxy.invokeSuper(o, args);
                        }
                    }
            );
            enhancer.create();
        }


    }
}
```



## JVM参数类型

- 标准参数
- X参数
- XX参数

### 标准参数

这些参数相对来说比较稳定，在jdk各个版本中基本没有什么区别。比如-version；-showversion； -help；-server；-client；-cp；-classpath等等

### X参数 - 非标准化参数

- -Xint：解释执行
- -Xcomp：第一次使用就编译成为本地代码
- -Xmixed：混合模式，JVM自己决定是否编译成本地代码

### XX参数 - 非标准化参数

- 布尔类型：`-XX:[+-]<name>`表示启用或者禁用name属性。比如`-XX:+UseConcMarkSweepGC` `-XX:+UseG1GC`
- KV类型：`-XX:<key>=<value>`表示key属性的值为value，比如`-XX:MaxGCPauseMillis=500`

我们常用的-Xmx -Xms -xss是什么参数？实际上他们是XX参数，-Xms等价于-XX:InitialHeapSize, -Xmx等价于-XX:MaxHeapSize



### 查看运行时参数的值

- -XX:PrintFlagsInitial
- -XX:PrintFlagsFinal

如 java -XX:+PrintFlagsFinal -version

我们可以通过jinfo命令来查看某个进程的运行时参数值，如：jinfo -flag MaxHeapSize 51878; jinfo -flag UseG1GC 3317

