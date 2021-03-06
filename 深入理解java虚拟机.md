# 深入理解java虚拟机

## 1.jdk jre jvm关系

```
jdk----> java开发工具
jre---->java运行时环境
jvm---->java虚拟机
```

## 2.jvm初体验

### 2.1工具环境搭建

环境：

```
windows: win8/10

jdk:  jdk8,去官网下，下载地址
https://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html

开发工具： eclipse,下载地址：http://mirrors.neusoft.edu.cn/eclipse/technology/epp/downloads/release/2019-09/R/eclipse-jee-2019-09-R-win32-x86_64.zip

性能分析工具： mat，下载地址
http://mirrors.neusoft.edu.cn/eclipse/mat/1.9.0/rcp/MemoryAnalyzer-1.9.0.20190605-win32.win32.x86_64.zip

```

### 2.2内存溢出

首先创建一个JvmDemo类：

JvmDemo.java

```
package top.juntech.jvm;


import java.util.ArrayList;
import java.util.List;

public class JvmDemo {

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		List<Demo> demolist = new ArrayList<>();
		while(true) {
			demolist.add(new Demo());
		}

	}

}

```

demo.java

```
package top.juntech.jvm;

public class Demo {
	
}

```

打开任务管理器，查看内存占用：![](http://img.vim-cn.com/80/5aaa5c68ab0e6b309c3a80bb1f6b28b9bbf542.png)

运行程序：

再打开任务管理器查看

![](http://img.vim-cn.com/4f/b4e44d5cc2c3774390c688f775e28aa205a687.png)

最终控制台显示：

![](http://img.vim-cn.com/8e/65db6cec12950dfce10423415f60b413e202e6.png)

但是我们发现这个过程太慢了，所以我们自己设置VM参数来加速它报错的过程

```
-XX:+HeapDumpOnOutOfMemoryError -Xms20m -Xmx20m
```

运行，发现很快的给我们报了堆内存溢出：

```
java.lang.OutOfMemoryError: Java heap space
Dumping heap to java_pid9496.hprof ...
Heap dump file created [27949099 bytes in 0.099 secs]
Exception in thread "main" java.lang.OutOfMemoryError: Java heap space
	at java.util.Arrays.copyOf(Arrays.java:3210)
	at java.util.Arrays.copyOf(Arrays.java:3181)
	at java.util.ArrayList.grow(ArrayList.java:265)
	at java.util.ArrayList.ensureExplicitCapacity(ArrayList.java:239)
	at java.util.ArrayList.ensureCapacityInternal(ArrayList.java:231)
	at java.util.ArrayList.add(ArrayList.java:462)
	at top.juntech.jvm.JvmDemo.main(JvmDemo.java:13)
```

###2.3 VM参数设置

请查看本博客的另外一篇[文章：jvm参数调优](https://juntech.top/posts/7c051298.html)

### 2.4 性能分析工具 mat

具体使用，请查看博文：<https://www.cnblogs.com/loong-hon/p/10475143.html>

