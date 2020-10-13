# Callable

callable 与 runnable的区别

+ callable 有返回值
+ callable 抛出异常
+ 落地方法不同 一个是call 一个是run
+ 有泛型 <>里是什么类型 就返回什么类型

```java
class MyThread2 implements Callable<Integer> {
    @Override
    public Integer call() throws Exception {
        return null;
    }
}

/**
 * 多线程中 第三种获得多线程的方法
 *
 * @author shanmingda
 * @date 2020-10-13 13:52
 */
public class CallableDemo {

    public static void main(String[] args) throws InterruptedException, ExecutionException{
        FutureTask futureTask = new FutureTask(new MyThread2());

        new Thread(futureTask, "A").start();
				new Thread(futureTask, "B").start(); // 不执行
        System.out.println(futureTask.get()); // 1234
    }
}
```

get() 方法一般请放在最后一行 

可以理解为高考考试 把会做的做了 难的耗时的放在最后 省的阻塞其他操作

FutureTask 不论创建了几个线程 只会调用一次