# CAS

> Compare And Swap 比较并交换

**通过 AtomicInteger 可以引出如下问题：**

***CAS --> UnSafe --> CAS底层思想 --> ABA --> 原子引用更新 --> 如何规避ABA问题***

***

如果线程的期望值 和 主物理内存的真实值相同 则变成要修改的值 如果不相同 则不修改**

```java
public static void main(String[] args) {
        AtomicInteger atomicInteger = new AtomicInteger(5);

        // public final boolean compareAndSet(int expect, int update) {
        //        return unsafe.compareAndSwapInt(this, valueOffset, expect, update);
        // }
        System.out.println(atomicInteger.compareAndSet(5, 2020) 
                           + "\tcurrent value " + atomicInteger.get());
        System.out.println(atomicInteger.compareAndSet(5, 1024) 
                           + "\tcurrent value " + atomicInteger.get());
//        true current value 2020
//        false current value 2020
    }
```

***

##底层原理

**UnSafe**类中的所有方法都是native修饰的 也就是说UnSafe类中的方法都直接调用操作系统底层资源执行相应任务

+ UnSafe类是CAS的核心类 由于Java方法无法访问底层系统 需要通过本地（native）方法来访问 UnSafe类相当于一个后门 基于该类可以直接操作特定内存的数据 UnSafe类存在于package sun.misc包中 其内部方法可以像C的指针一样操作内存 因为Java中的CAS操作的执行依赖于UnSafe类的方法

![](http://img.tomato530.com/CAS1.png)

变量 valueOffset 表示该变量值在内存中的偏移地址 因为UnSafe类就是根据内存偏移地址获取数据的

![](http://img.tomato530.com/CAS2.png)

变量value是用volatile修饰 保证了对线程直接的可见性

![](http://img.tomato530.com/CAS3.png)

**CAS**的全称为Compare-And-Swap 它是一条**CPU并发原语**

它的功能是判断内存中某个位置的值是否为预期值 如果是则更改为新的值 这个过程是原子的

CAS并发原语体现在Java语言中就是sun.misc.UnSafe类中的各个方法 调用UnSafe类中的CAS方法 JVM会帮我们实现出CAS汇编指令 这是一种完全依赖于硬件的功能 通过它实现了原子操作 再次强调 由于CAS是一种系统原语 原语属于系统用语范畴 是由若干条指令组成的 用于完成某个功能的一个过程 并且原语的执行必须是连续的 在执行过程中不允许被中断 也就是说CAS是一条CPU的原子指令 不会造成所谓的数据不一致问题

UnSafe类源码如下图

![](http://img.tomato530.com/CAS4.png)

源码解析：

1. var1 AtomicInteger对象本身
2. var2 该对象值的引用地址
3. var4 需要变动的数量
4. var5 是通过var1 var2 找出的内存中的真实的值（var5 = this.getIntVolatile(var1, var2);）用该对象当前的值与var5比较 如果相同 更新var5+var4并且返回true 如果不同 继续取值然后再比较 直到更新完成

假设线程A和线程B同时执行getAndAddInt操作（分别跑在不同CPU上）

1. AtomicInteger里面的value原始值为3 即主内存中的AtomicInteger的value为3 根据JMM模型 线程A和线程B各自持有一份值为3的副本分别到各自的工作内存
2. 线程A通过getIntVolatile(var1, var2)拿到value值3 这时线程A挂起
3. 线程B也通过getIntVolatile(var1, var2)方法获取到value值3 此时刚好线程B没有被挂起并执行compareAndSwapInt方法比较内存值也为3 成功修改内存值为4 线程B收工
4. 这是线程A恢复 执行compareAndSwapInt方法比较 发现自己手里的值数字3和主内存中的数字4不一样 说明该值已经被其他线程抢先一步修改过了 那线程A本次修改失败 只能重新读取重来一遍了
5. 线程A重新获取value值 因为变量value被volatile修饰 所以其他线程对它的修改 线程A总能够看到 线程A继续执行compareAndSwapInt进行比较替换 直到成功

CAS

+ 比较当前工作内存中的值和主内存中的值 如果相同则执行规定操作 否则继续比较直到主内存和工作内存值一致为止

CAS应用

+ CAS有3个操作数 内存值V 旧的预期值A 要修改的更新值B 当且仅当预期值A和内存值V相同时 将内存值V修改为B 否则什么也不做

可以看到getAndAddInt方法执行时 有个 do while 如果CAS失

##CAS缺点

可以看到getAndAddInt方法执行时 有个 do while 如果CAS失败 会一直进行尝试 如果CAS长时间一直不成功 可能会给CPU带来很大的开销

只能保证一个变量的共享操作

ABA问题

## ABA问题

CAS会导致 ABA问题

CAS算法实现一个重要前提需要取出内存中某时刻的数据并在当下时间比较并交换 那么在这个时间差会导致数据的变化

比如说一个线程one从内存位置V中取出A 这时候另一个线程two也从内存中取出A 并且线程two进行了一些操作将值变成了B 然后线程two又将V位置的数据变成A 这时候one线程进行CAS操作发现内存中仍然是A 然后线程one操作成功

解决ABA  理解原子引用 + 新增一种机制 那就是修改版本号（类似时间戳）

```java
/**
 * ABA问题的解决 java.util.concurrent.atomic.AtomicStampedReference<V>
 *
 * @author shanmingda
 * @date 2020-10-16 00:11
 */
public class ABADemo {

    static AtomicReference<Integer> atomicReference = new AtomicReference<>(100);
    static AtomicStampedReference<Integer> atomicStampedReference = new AtomicStampedReference<>(100, 1);

    public static void main(String[] args) {

        System.out.println("------以下是ABA问题的产生------");

        new Thread(() -> {
            atomicReference.compareAndSet(100, 101);
            atomicReference.compareAndSet(101, 100);
        }, "T1").start();

        new Thread(() -> {
            try {
                TimeUnit.SECONDS.sleep(1);
                System.out.println(atomicReference.compareAndSet(100, 2020) + "\t" + atomicReference.get());
            } catch (InterruptedException e) { e.printStackTrace();}
        }, "T2").start();

        try {
            TimeUnit.SECONDS.sleep(2);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("------以下是ABA问题的解决------");

        new Thread(() -> {
            // 拿到初始版本号
            int stamp = atomicStampedReference.getStamp();
            System.out.println(Thread.currentThread().getName() + "\t初始版本号" + stamp);
            // 暂停1秒T3
            try {
                TimeUnit.SECONDS.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            atomicStampedReference.compareAndSet(100, 101,
                    atomicStampedReference.getStamp(), atomicStampedReference.getStamp() + 1);
            System.out.println(Thread.currentThread().getName() + "\t第二次版本号" + atomicStampedReference.getStamp());

            atomicStampedReference.compareAndSet(101, 100,
                    atomicStampedReference.getStamp(), atomicStampedReference.getStamp() + 1);
            System.out.println(Thread.currentThread().getName() + "\t第三次版本号" + atomicStampedReference.getStamp());
        }, "T3").start();

        new Thread(() -> {
            // 拿到初始版本号
            int stamp = atomicStampedReference.getStamp();
            System.out.println(Thread.currentThread().getName() + "\t初始版本号" + stamp);
            // 暂停4秒T4 保证T3线程完成一次ABA操作
            try {
                TimeUnit.SECONDS.sleep(4);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("修改成功否 " +
                    atomicStampedReference.compareAndSet(100, 2020,
                            stamp, stamp + 1) + "\t当前最新版本号" + atomicStampedReference.getStamp());
            System.out.println(Thread.currentThread().getName() + "\t当前最新值为：" + atomicStampedReference.getReference());
        }, "T4").start();
    }
}
```

