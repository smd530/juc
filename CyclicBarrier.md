# CyclicBarrier

可循环利用的屏障

集齐七颗龙珠再召唤神龙

```java
/**
 * 集齐七颗龙珠再召唤神龙
 *
 * @author shanmingda
 * @date 2020-10-13 15:52
 */
public class CyclicBarrierDemo {

    public static void main(String[] args) {

//        CyclicBarrier(int parties, Runnable barrierAction)
        CyclicBarrier cyclicBarrier = new CyclicBarrier(7, () -> {
            System.out.println("召唤神龙");
        });

        for (int i = 0; i < 7; i++) {
            final int tempInt = i;
            new Thread(() -> {
                System.out.println(Thread.currentThread().getName() + "\t收集到第："+tempInt+"颗龙珠");
                try {
                    cyclicBarrier.await();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } catch (BrokenBarrierException e) {
                    e.printStackTrace();
                }
            }, String.valueOf(i)).start();
        }

    }
}
```