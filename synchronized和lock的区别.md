#synchronized 和 lock的区别

```java
public class SyncAndReentrantLockDemo {

    public static void main(String[] args) {

      // 底层用的是 monitor对象 锁的监控器
      // synchronized用的锁是存在对象头里的 
        synchronized (new Object()) {

        }
      
      	new ReentrantLock();
      // javap -c 之后
//        public static void main(java.lang.String[]);
//            Code:
//                0: new           #2                  // class java/lang/Object
//                3: dup
//                4: invokespecial #1                  // Method java/lang/Object."<init>":()V
//                7: dup
//                8: astore_1
//                9: monitorenter  // 进来 monitor计数器+1
//                10: aload_1
//                11: monitorexit // 出来 第一次是正常退出
//                12: goto          20
//                15: astore_2
//                16: aload_1
//                17: monitorexit // 第二次是异常退出
//                18: aload_2
//                19: athrow
//                20: new           #3                  // class java/util/concurrent/locks/ReentrantLock
//                23: dup
//                24: invokespecial #4                  // Method java/util/concurrent/locks/ReentrantLock."<init>":()V
//                27: pop
//                28: return

        
    }
}
```

## 原始构成

synchronized 是关键字 属于JVM层面

+ monitorenter（底层是通过monitor对象来实现 wait/notify等方法也依赖于monitor对象 只有在同步代码或方法中才能调用wait/notify等方法）
+ monitorexit

Lock 具体类（java.util.concurrent.locks.ReentrantLock）是api层面的锁

## 使用方法

synchronized 不需要用户去手动释放锁 当synchronized代码执行完成后会自动让线程释放对锁的占用

ReentrantLock 需要用户去手动释放锁 若是没有主动释放锁 就有可能出现死锁

+ 需要用 lock() unlock() 方法来控制

## 等待是否可中断

synchronized 不可中断除非抛异常或者正常运行完成

ReentrantLock 可中断 

1. 设置超时方法 boolean tryLock(long time, TimeUnit unit)
2. lockInterruptibly()放代码块中 调用interrupt()方法可中断

## 加锁是否公平

synchronized 非公平

ReentrantLock 都可以 默认非公平 构造方法可以传入 boolean值 true为公平 false非公平

## 锁绑定多个条件Condition

synchronized 没有

ReentrantLock 用来实现分组唤醒需要唤醒的线程们 可以精确唤醒 而不是像synchronized要么随机唤醒 要么全部唤醒