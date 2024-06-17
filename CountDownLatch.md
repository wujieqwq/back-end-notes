# CountDownLatch概述
- CountDownLatch一般用作多线程倒计时计数器，强制它们等待其他一组（CountDownLatch的初始化决定）任务执行完成。
- 有一点要说明的是CountDownLatch初始化后计数器值递减到0的时候，不能再复原的，这一点区别于Semaphore，Semaphore是可以通过release操作恢复信号量的。

# 使用原理
1. 创建CountDownLatch并设置计数器值。
2. 启动多线程并且调用CountDownLatch实例的countDown()方法。
3. 主线程调用 await() 方法，这样主线程的操作就会在这个方法上阻塞，直到其他线程完成各自的任务，count值为0，停止阻塞，主线程继续执行。

```
//使用模板
public class CountDownLatchModule {

    //线程数
    private static int N = 10;

    // 单位：min
    private static int countDownLatchTimeout = 5;

    public static void main(String[] args) {
        //创建CountDownLatch并设置计数值，该count值可以根据线程数的需要设置
        CountDownLatch countDownLatch = new CountDownLatch(N);

		//创建线程池
        ExecutorService cachedThreadPool = Executors.newCachedThreadPool();

        for (int i = 0; i < N; i++) {
            cachedThreadPool.execute(() ->{
                try {
                    System.out.println(Thread.currentThread().getName() + " do something!");
                } catch (Exception e) {
                    System.out.println("Exception: do something exception");
                } finally {
                    //该线程执行完毕-1
                    countDownLatch.countDown();
                }
            });
        }

        System.out.println("main thread do something-1");
        try {
            countDownLatch.await(countDownLatchTimeout, TimeUnit.MINUTES);
        } catch (InterruptedException e) {
            System.out.println("Exception: await interrupted exception");
        } finally {
            System.out.println("countDownLatch: " + countDownLatch.toString());
        }
        System.out.println("main thread do something-2");
        //若需要停止线程池可关闭;
//        cachedThreadPool.shutdown();

    }
```