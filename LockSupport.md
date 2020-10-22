# LockSupport

**java.util.concurrent.locks.LockSupport**

**LockSupport**是用于创建锁和其他同步类的基本线程阻塞原语 本质是线程等待唤醒机制(wait/notify)的加强版

**LockSupport**中的 park() 和 unpark() 的作用分别是阻塞线程和解除阻塞线程 

**LockSupport**类使用了一种名为**Permit（许可）**的概念来做到阻塞和唤醒线程的作用 每个线程都有一个许可

**Permit**只有两个值 0和1 默认是0

可以把许可看成是一种(0,1)信号量(Semaphore) 但是于Semaphore不同的是 许可的累积上线是1

***

调用**LockSupport.park()**时其实就是调用**UnSafe**类

![](http://img.tomato530.com/Park.png)

![](http://img.tomato530.com/Park2.png)

**Permit**默认是0 所以一开始调用park()方法 当前线程就会阻塞 直到别的线程将当前线程的**Permit**设置为1时 park方法会被唤醒 然后会将**Permit**再置为0并返回

***

调用**LockSupport.unpark(new Thread());**时其实也是调用**UnSafe**类

![](http://img.tomato530.com/unpark.png)

![](http://img.tomato530.com/unpark2.png)

调用unpack(thread)方法时 会将thread线程的许可**Permit**设置为1（注意多次调用unpack()方法 不会累加 **Permit**就是1）会自动唤醒thread线程 即之前阻塞中的park方法会立即返回

***

```java
public static void main(String[] args) {
    Thread a = new Thread(() -> {
        System.out.println(Thread.currentThread().getName() + "\tcome in");
        // 被阻塞 等待通知等待放行 它要通过需要许可证
        LockSupport.park();
        System.out.println(Thread.currentThread().getName() + "\t被唤醒");
    }, "A");
    a.start();

    try {
        TimeUnit.SECONDS.sleep(3);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }

    Thread b = new Thread(() -> {
        System.out.println(Thread.currentThread().getName() + "\t通知了");
        LockSupport.unpark(a);
    }, "B");
    b.start();
}
```

![](http://img.tomato530.com/LockSupport.png)

***

**面试题**

![](http://img.tomato530.com/LockSupport2.png)