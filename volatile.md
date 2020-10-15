# Volatile

Volatile是JVM提供的轻量级的同步机制

***

**三大特性**

1. 保证可见性

   + 一个工作内存修改了主内存共享变量的值 其他线程要立刻知道它修改了

2. 不保证原子性

   + 什么是原子性 不可分割 完整性 也即某个线程正在作某个具体业务时 中间不可以被加塞或者分割 需要整体完整 要么同时成功 要么同事失败
   + 解决不保证原子性办法 配合原子类使用 java.util.concurrent.atomic

3. 禁止指令重排

   + 什么是指令重排 计算机在执行程序时 为了提高性能 编译器和处理期常常会对指令做重排一般分三种

     1. 编译器优化的重排
     2. 指令并行的重排
     3. 内存系统的重排
     4. 最终执行的命令

   + 单线程环境里确保程序最终执行结果和代码执行顺序一致 处理器在进行重排序时必须要考虑指令之间的**数据依赖性** 

   + 对线程环境中线程交替执行 由于编译器优化重排的存在 两个线程中使用的变量能否保证一致性是无法确定的 结果无法预测

   + 内存屏障（Memory Barrier）

     + 一是保证了特定操作的执行顺序
     + 二是保证某些变量的内存可见性

     由于编译器和处理器都能执行指令重排优化 如果在指令间插入一条Memory Barrier则会告诉CPU和编译器 不管什么指令都不能和这条Memory Barrier指令重排序 **也就是说通过插入内存屏障禁止在内存屏障前后的指令执行重排序优化** 内存屏障的另一个作用是强制刷出各种CPU的缓存数据 因此CPU上的线程都能读到这些数据的最新版本
     
     store load

***

**Volatile的使用**

```java
/**
 * 单例
 *
 * @author shanmingda
 * @date 2020-10-15 16:37
 */
public class SingletonDemo {

    private static volatile SingletonDemo instance = null;

    private SingletonDemo() {
        System.out.println(Thread.currentThread().getName()+"\tI am 构造方法");
    }

    // DCL （Double check Lock 双端检索机制）
    public static SingletonDemo getInstance() {
        if (instance == null) {

            synchronized (SingletonDemo.class) {
                if (instance == null) {
                    instance = new SingletonDemo();
                }
            }
        }
        return instance;
    }
    public static void main(String[] args) {
//        System.out.println(SingletonDemo.getInstance() == SingletonDemo.getInstance());


        for (int i = 0; i < 100; i++) {
            new Thread(() -> {
                SingletonDemo.getInstance();
            }, String.valueOf(i)).start();
        }
    }
}
```

