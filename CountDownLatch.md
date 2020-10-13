#CountDownLatch

```java
/**
 * TODO
 *
 * @author shanmingda
 * @date 2020-10-13 15:19
 */
public class CountDownLatchDemo {

    public static void main(String[] args) throws InterruptedException {

        CountDownLatch countDownLatch = new CountDownLatch(6);

        for (int i = 0; i < 6; i++) {

            new Thread(() -> {
                System.out.println(Thread.currentThread().getName() + "\t离开");
                countDownLatch.countDown();
            }, String.valueOf(i)).start();
        }
        countDownLatch.await();


        System.out.println(Thread.currentThread().getName() + "\t关门");
    }
  
//0	离开
//2	离开
//1	离开
//3	离开
//4	离开
//5	离开
//main	关门
}
```

CountDownLatch 主要有两个方法 当一个线程或多个线程调用await方法时 这些线程会阻塞

其他线程调用countDown方法会将计数器减1（调用countDown方法的线程不会阻塞）

当计数器的值变为0时 因await方法阻塞的线程会被唤醒 继续执行