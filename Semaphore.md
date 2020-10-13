# Semaphore

互斥信号量

控制多线程的并发数

```java
public static void main(String[] args) {

    //Semaphore(int permits)

    //模拟资源类 有三个空车位
    Semaphore semaphore = new Semaphore(3);

    for (int i = 0; i < 6; i++) {
        new Thread(() -> {
            try {
                 // 占用
                semaphore.acquire();
                System.out.println(Thread.currentThread().getName()+"\t抢占了车位");
                TimeUnit.SECONDS.sleep(3);
                System.out.println(Thread.currentThread().getName()+"\t离开了车位");
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                //释放
                semaphore.release();
            }

        }, String.valueOf(i)).start();
    }
}
```

在信号量上我们定义了两种操作

+ acquire（获取）当一个线程调用acquire操作时 它要么通过成功获取信号量（信号量减少1）要么一直等下去 直到有线程释放信号量 或超市
+ release（释放）实际上会将信号量的值加1 然后唤醒等待的线程

信号量主要用于两个目的 一个是用于多个线程共享资源的互斥使用 用一个用于控制线程的并发数

Semaphore semaphore = new Semaphore(1); 当传入1 可以当synchronized使用