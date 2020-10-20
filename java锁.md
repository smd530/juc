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

指的是同一线程外层函数获得锁以后 内存递归函数仍然能获取该锁的代码 在同一线程在外层方法获取锁的时候 在进入内层方法会自动获得锁 也就是说 **线程可以进入任何一个它已经拥有的锁所同步的着的代码块**

ReentrantLock 和 synchronized 都是可重入锁

**可重入锁的最大作用就是避免死锁**

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

