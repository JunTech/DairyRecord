---
title: jvm参数调优
author: Juntech
top: false
cover: true
toc: true
mathjax: false
copyright: true
summary: jvm参数调优
categories: jvm
tags: jvm
keywords: jvm
abbrlink: 7c051298
date: 2019-09-25 17:40:47
---

常见参数
示例参数	描述
-Xms20m	堆初始值20M
-Xmx20m	堆最大可用值20M
-Xmn5m	新生代最大可用值5M
-XX:PrintGC	触发GC时日志打印
-XX:PrintGCDetails	触发GC时日志打印更详细
-XX:UseSerialGC	串行回收
-XX:SurvivorRatio=2	eden:from:to = 2:1:1
-XX:NewRatio=2	新生代:老年代 = 1:2
```
堆内存大小配置
// 堆内存大小配置 -Xmx100m -Xms50m
    public static void main(String[] args) {

        System.out.print("最大内存： ");
        System.out.println(Runtime.getRuntime().maxMemory() / 1024 / 1024 + "MB");
        System.out.print("可用内存： ");
        System.out.println(Runtime.getRuntime().freeMemory() / 1024 / 1024 + "MB");
        System.out.print("已使用内存： ");
        System.out.println(Runtime.getRuntime().totalMemory() / 1024 / 1024 + "MB");
    }
    ```
参数配置前运行 ，以下是本机默认值。（每台机器配置不同值可能会不同）

最大内存： 2665MB
可用内存： 177MB
已使用内存： 180MB
配置 jvm 参数

main方法位置右击
使用示例: -Xmx100m -Xms50m
说明： 当前应用最大可用内存为100M，初始内存为50M

main 方法如果从建立一次都没运行过，下图中 左侧1 的位置不会发现程序，运行一下在打开下图配置就有了。

配置jvm参数
参数配置后运行

最大内存： 89MB
可用内存： 46MB
已使用内存： 48MB
参数配置信息最大可用内存为100M，初始内存为50M
按理这里计算出来的最大内存是100M，已使用内存50M，但结果却都小了一些
这是正常的误差，不去纠结少了的那一小部分内存

设置新生代比例参数
-Xms20m -Xmx20m -Xmn1m -XX:SurvivorRatio=2 -XX:+PrintGCDetails -XX:+UseSerialGC

说明：堆内存初始化值20m,堆内存最大值20m，新生代最大值可用1m，eden空间和from/to空间的比例为2/1，GC回收日志，使用串行回收

jvm 参数配置
```
 public static void main(String[] args) {
        for (int i = 0; i < 10; i++) {
            byte[] b = new byte[1 * 1024 * 1024];
        }
    }
    ```
程序运行

运行结果
箭头所指就是 触发GC回收信息 - GC (Allocation Failure)
框起来的是新生代内存信息，我们设置了
-Xmn1m （eden+from+to <= 1m）
-XX:SurvivorRatio=2 （ eden：from：to = 2:1:1）
默认 eden：from：to = 8：1：1

设置新生代与老年代比例参数
使用示例: -Xms20m -Xmx20m -XX:SurvivorRatio=2 -XX:+PrintGCDetails
-XX:+UseSerialGC -XX:NewRatio=2

说明：堆内存初始化值20m,堆内存最大值20m，新生代最大值可用1m，eden空间和from/to空间的比例为2/1,新生代和老年代的占比为1/2

jvm 配置
```
 public static void main(String[] args) {
        for (int i = 0; i < 10; i++) {
            byte[] b = new byte[1 * 1024 * 1024];
        }
    }
```
程序运行


运行结果
-XX:SurvivorRatio=2，（上面框）新生代中 eden：from：to=2:1:1
-XX:NewRatio=2， 新生代(eden+from+to) : 老年代 约等 1: 2，（下面框）老年代内存信息
默认 新生代(eden+from+to) : 老年代 = 1: 2

实战OutOfMemoryError
原因： 堆最大内存值小了，不能满足当前程序
解决： 增大堆最大内存空间
Java堆溢出 - OutOfMemoryError异常，在开发过程中很容易遇到，在没接触 JVM 之前遇到这样的错误可能会惊一下，莫名其妙。接触 JVM 之后你会发现也就这么回事。

jvm 配置
-Xms1m -Xmx10m -XX:+PrintGCDetails
-XX:+HeapDumpOnOutOfMemoryError

说明：堆内存初始化值1m,堆内存最大值10m，GC回收日志
```
 public static void main(String[] args) {

        List<Object> listObject = new ArrayList<>();
        for (int i = 0; i < 10; i++) {
            System.out.println("i:" + i);
            Byte[] bytes = new Byte[1 * 1024 * 1024];
            listObject.add(bytes);
        }
        System.out.println("添加成功...");
    }
```
程序运行


运行结果
报错了！！OutOfMemoryError

解决

增大堆最大内存 -Xmx10m 更改为 -Xmx100m
再次运行


运行结果
添加成功，没有错误

实战StackOverflowError
栈溢出 - java.lang.StackOverflowError

栈溢出 产生于递归调用，循环遍历是不会的，但是循环方法里面产生递归调用， 也会发生栈溢出。
解决办法: -Xss5m 设置线程最大调用深度
```
private static int count;

    public static void count() {
        try {
            count++;
            count();
        } catch (Throwable e) {
            System.out.println("最大深度:" + count);
            e.printStackTrace();
        }
    }
```
    public static void main(String[] args) {
        count();
    }
main方法 里面进行递归调用，运行如下：

最大深度:10775
java.lang.StackOverflowError
    at sun.misc.Unsafe.compareAndSwapLong(Native Method)
    ...
这里每次运行最大深度不同，但相差不大（本机默认深度10775左右）
如果是比较深的递归，就需要配置最大深度

比如现在有一个递归A方法执行报错StackOverflowError，那么A递归方法的访问的深度已经大于了默认的最大深度。

Xss5m 设置最大调用深度


jvm配置
再次运行

最大深度:257058
java.lang.StackOverflowError
    at sun.misc.Unsafe.compareAndSwapLong(Native Method)
    ...
257058 对比之前 10775

修改 -Xss5m 为 - Xss1m , 再次运行 ：

 最大深度:10674
java.lang.StackOverflowError
    at sun.misc.Unsafe.compareAndSwapLong(Native Method)
10674 对比之前 10775 ， 好像差不多
可以推测默认的深度可能为 1m

StackOverflowError 产生于递归调用
循环遍历不生效
误区
```
private static int count;

    public static void count() {
        try {
            count++;
        //  count();
        } catch (Throwable e) {
            System.out.println("最大深度:" + count);
            e.printStackTrace();
        }
    }
    // 错误示例
    public static void main(String[] args) {
        for (int i = 0; i < 11000; i++) {
            count();
        } 
    }
```
内存溢出与内存泄漏区别
java内存泄漏，使用过的内存空间没有被及时释放，长时间占用内存，导致系统无法再给你提供内存资源（内存资源耗尽）
Java内存溢出，要求分配的内存超出了系统能给你的，系统不能满足需求，于是产生溢出（杯中的水满了溢出）