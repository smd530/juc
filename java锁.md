# java的锁

锁 牛逼啊......

## 公平锁 非公平锁

**公平锁**

多个线程按照申请锁的顺序来获取锁 等待队列 类似排队打饭 先来后到

**非公平锁**

是指多个线程获取锁的顺序并不是按照锁的申请顺序 有可能后申请的线程比先申请的线程优先获取锁 在高并发情况下 有可能会造成优先级反转和饥饿现象 非公平锁比较粗鲁 上来就尝试占有锁 如果尝试失败 就再采用类似公平锁那种方式 非公平锁的优点是比公平锁吞吐量大

synchronized 非公平

```java
// ReentrantLock没有参数的构造函数默认是非公平的
// 源码如下
/**
 * Creates an instance of {@code ReentrantLock}.
 * This is equivalent to using {@code ReentrantLock(false)}.
 */
public ReentrantLock() {
    sync = new NonfairSync();
}
```

```java
/**
 * Creates an instance of {@code ReentrantLock} with the
 * given fairness policy.
 *
 * @param fair {@code true} if this lock should use a fair ordering policy
 */
public ReentrantLock(boolean fair) {
    sync = fair ? new FairSync() : new NonfairSync();
}
```

## 可重入锁

又叫递归锁

是指在同一个线程在外层方法获取锁的时候 再进入该线程的内层方法会自动获取锁（前提 锁对象是同一个对象）不会因为之前已经获得过还没释放而阻塞也就是说 **线程可以进入任何一个它已经拥有的锁所同步的着的代码块**

可以 再次 进入 同步域（即同步代码块/方法或显式锁锁定的代码）

一个线程中的多个流程可以获取统一把锁 持有这把同步锁可以再次进入 自己可以获取自己的内部锁

ReentrantLock 和 synchronized 都是可重入锁

**可重入锁种类**

+ 隐式锁（即synchronized关键字使用的锁）
+ 显式锁（即Lock）

**可重入锁的最大作用就是避免死锁**

**Synchronized** 重入实现锁的机理

+ 每个锁对象拥有一个锁计数器和一个指向该锁的线程的指针
+ 当执行monitorenter的时候 如果目标锁的计数器为零 那么说明它没有被其他线程所持有 Java虚拟机会将该锁对象的持有线程设置为当前线程 并且将其计数器加一
+ 在目标锁技术器不为零的时候 如果锁对象的持有线程是当前线程 那么Java虚拟机可以将器计数器加一 否则需要等待 直到持有线程释放该锁
+ 当执行monitorexit时 Java虚拟机则需将锁对象的计数器减一 计数器为零代表锁已经释放

## 自旋锁（spinlock）

是指尝试获取锁的线程不会立即阻塞 而是**采用循环的方式去尝试获取锁** 这样的好处是减少线程上下午切换的消耗 缺点是循环会消耗CPU资源

自旋锁好处：循环比较获取直到成功为止 没有类型 wait 的阻塞

通过CAS操作完成自旋锁 A线程先进来调用mylock方法自己持有锁5秒钟 B随后进来后发现

当前线程持有锁 不是null 所以只能自旋等待 直到A释放锁后B随后抢到

## 独占锁（写锁）/共享锁（读锁）/互斥锁

独占锁：指该锁一次只能被一个线程所持有 比如 ReentrantLock 和 synchronized

共享锁：指该锁可被多个线程所持有

对ReentrantReadWriteLock其读锁是共享锁 其写锁是独占锁

读锁的共享锁可以保证并发读是非常高效的 

