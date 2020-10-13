# BlockingQueue

**阻塞队列**

当队列是空的 从队列中获取元素的操作会被阻塞

当队列是满的 从队列中添加元素的操作会被阻塞

试图从空的队列中获取元素的线程将会被阻塞 直到其他线程往空的队列插入新的元素

试图向已满的队列中添加新元素的线程将会被阻塞 知道其他线程从队列中移除一个或者多个元素或者完全清空 使队列变得空闲起来并后续新增

在对线程领域：所谓阻塞 在某些情况下会挂起线程（即阻塞）一旦条件满足 被挂起的线程又会自动被唤起

为什么需要BlockingQueue

好处是我们不需要关心什么时候需要阻塞队列 什么时候需要唤醒线程 因为这一切BlockingQueue都一手包办了

在concurrent包发布以前 在多线程环境下 我们每个程序员都必须去自己控制这些细节 尤其还要兼顾效率和线程安全 而这会给我们的程序带来不小的复杂度

```java
// ArrayBlockingQueue 由数组结构组成的有界阻塞队列
BlockingQueue arrayblockingQueue = new ArrayBlockingQueue(2);
// LinkedBlockingDeque 由链表结构组成的有界（integer.MAX_VALUE）阻塞队列
BlockingQueue linkedblockingQueue = new LinkedBlockingDeque(2);
```

![](http://img.tomato530.com/BlockingQueue.png)

![](http://img.tomato530.com/BlockingQueue2.png)