# 深入ReentrantLock底层原理01

## 1、Thread线程

```java
package top.juntech.lock;

import java.util.concurrent.locks.ReentrantLock;

public class TestLock {
    public static void main(String[] args) {
        Thread t1 = new Thread(){

            public void run(){
                testSync();
            }
        };
        Thread t2 = new Thread(){

            public void run(){
                testSync();
            }
        };
        t1.setName("t1");
        t2.setName("t2");
        t1.start();
        t2.start();

    }

    public static void testSync(){
           
            System.out.println(Thread.currentThread().getName());
            try {
                Thread.sleep(2000);
            }catch (Exception e){
                e.printStackTrace();
            }

 


    }


}

```

在这里，我们的线程t1,t2同时调用了testSync方法,尽管有thread.sleep(2000)，但我们发现t1,t2几乎是同时打印出来的，为什么呢，在t1进入这个方法里面，虽然在睡眠但不影响主线程继续执行，接下来t2启动，那我们怎么使他们一前一后执行呢，答案时加锁。我们这里就将ReentranLock.

1、添加一个全局变量： ReentranLock reen = new ReentranLock(true);

2、在方法里面添加lock方法

3、释放锁

代码如下：

```java
package top.juntech.lock;

import java.util.concurrent.locks.ReentrantLock;   //初始化锁对象

public class TestLock {
    static  ReentrantLock reentrantLock = new ReentrantLock(true);
    public static void main(String[] args) {
        Thread t1 = new Thread(){

            public void run(){
                testSync();
            }
        };
        Thread t2 = new Thread(){

            public void run(){
                testSync();
            }
        };
        t1.setName("t1");
        t2.setName("t2");
        t1.start();
        t2.start();

    }

    public static void testSync(){
        try {
            reentrantLock.lock();   //加锁
            System.out.println(Thread.currentThread().getName());
            try {
                Thread.sleep(2000);
            }catch (Exception e){
                e.printStackTrace();
            }

        }finally {
            reentrantLock.unlock();  //释放锁
        }


    }


}

```

这样，我们就可以看到t1打印完了过了2s左右，才开始打印t2.

## 2、思考

既然java里面有关键字sysnchronized，为什么还需要ReentranLock？

java中的线程是和操作系统中的线程是一一对应的，synchronized在执行多线程时，t1进入synchronized方法后，t2也想进去，它会让t2操作操作系统的一个函数，t2占用之后，会让操作系统的哪个函数产生阻塞，直到t1醒来，执行完释放，t2拿到锁之后，接着执行同步代码块。

### 2.1 多线程同步内部是如何实现的

wait/notify,synchronized	,ReentranLock

模拟一些同步的思路：

**自旋锁：**

```java
volatile int status = 0;
void lock(){
    while(!compareAndSet(0,1)){
        
    }
    //lock
}
void unlock(){
    status = 0;
}
boolean compareAndSet(int except,int newValue){
    //compare  and  set操作,修改status成功返回true
}
```

缺点： 耗费cpu资源，没有竞争到锁的线程会一直占用cas操作。

思路：让得不到锁的线程让出cpu

不能用wait,因为wait是和sync一起使用的，单独使用会报错，且wait不只是拿来使线程阻塞的，它还可以用来线程通信。

**yield+自旋锁**

```java
volatile int status = 0;
void lock(){
    while(!compareAndSet(0,1)){
       yield();//自己实现 
    }
    //lock
}
void unlock(){
    status = 0;
}
boolean compareAndSet(int except,int newValue){
    //compare  and  set操作,修改status成功返回true
}
```

要解决自旋锁的性能问题必须让竞争锁失败的的线程不空转，而是在获取不到锁的时候让出cpu,yeild()方法可以让出cpu;但是yield并不能完全解决问题，在线程数为2个时，可以使用，但线程数比较多的时候，就不适用了，因为yeild是由操作系统控制的。

yield方法，请参考：<https://blog.csdn.net/zhuwei898321/article/details/72844506>

**sleep+自旋锁**

```java
volatile int status = 0;
void lock(){
    while(!compareAndSet(0,1)){
       sleep(10};//自己实现 
    }
    //lock --- 1s
}
void unlock(){
    status = 0;
}
boolean compareAndSet(int except,int newValue){
    //compare  and  set操作,修改status成功返回true
}
```

缺点：睡眠时间不确定,易造成cpu资源浪费

**park+自旋锁**

这里要使用的是UNSAFE类里面的park方法,下面是park方法的具体使用

```java
package top.juntech.lock;


import java.util.concurrent.locks.LockSupport;

public class TestPark {

    public static void main(String[] args) {
        Thread t1 = new Thread(){

            public void run(){
                testSync();
            }
        };
        t1.setName("t1");
        t1.start();

        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("--------main-------");   // 3、打印main
        LockSupport.unpark(/*线程名*/t1);   //4、叫醒t1，继续执行，打印t1---2
    }

    public static void testSync(){
        System.out.println(/*Thread.currentThread().getName()*/ "t1 -----1"); //1.首先打印t1 --1
        LockSupport.park();   //2.t1开始睡眠
        System.out.println("t1 ----- 2");


    }
}

```

伪代码如下：

```java
volatile int status = 0;
Queue parkQueue; //集合 数组
void lock(){
    while(!compareAndSet(0,1)){
      park();
    }
    //lock    20s
    unpark();
}
void unlock(){
status = 0;
    lock_notify();
}
boolean compareAndSet(int except,int newValue){
    //compare  and  set操作,修改status成功返回true
}
void park(){
//将当前线程添加到等待队列
    parkQueue.add(currentThread)
  //将当前线程释放cpu，阻塞
  releaseCpu();
}
void lock_notify(){
//得到要唤醒的线程头部线程
    Thread t = parkQueue.header();
    //唤醒等待线程
    unpark(t);
}
```



