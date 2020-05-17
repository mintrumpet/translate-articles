# 翻译 | java 并发编程：Count Down Latches

> 原文自国外技术社区dzone，作者为 Akshansh Jain，[传送门](https://dzone.com/articles/java-concurrency-count-down-latches)

读者你们好，再次欢迎大家来到 java 并发编程系列的另一篇文章中。今天，我们打算去了解下 java 中的 `CountDownLatch` 类，以及它的用处。现在就让我们直奔主题吧。

### java 中的 CountDownLatch

有时会出现这么一种情况，仅当一组特定的任务完成后，我们需要去启动我们的应用。这些任务往往是并行运行并且在同一时间或者在不同时间段中完成。接着，我们应当如何告诉其他线程所有任务完成了呢？怎么去跟踪哪个任务是已经完成的，哪个任务还没有完成？`CountDownLatch` 就是用来解决这些问题的一个类。

我们能够在应用中将 `CountDownLatch` 定义为一个带有计算器的类。计算器的起点值是我们通知其他线程可以开始运行前需要等待的线程数。例如，如果我们在其他线程运行前需要等待5个任务完成，那 `CountDownLatch` 的起始值就为5。当一条线程完成它自身的任务，调用 **CountDownLatch's** 的 `countDown()` 方法来让程序知道任务已经完成了。`countDown()` 负责对起始值进行递减操作。因此，当值到达0的时候，Latch 知道所有线程都完成任务了，等待中的线程就可以运行了。`await()` 方法是 `CountDownLatch` 的阻塞方法，它会阻塞直到计数数值变为0，之后 `await()` 方法会立即返回。让我们看看下面的例子：

```java
import java.util.concurrent.CountDownLatch;
public class CountDownLatchDemo {
    private static final CountDownLatch COUNT_DOWN_LATCH = new CountDownLatch(5);
    public static void main(String[] args) {
        Thread run1 = new Thread(() -> {
            System.out.println("Doing some work..");
            try {
                Thread.sleep(2000);
                COUNT_DOWN_LATCH.countDown();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });
        Thread run2 = new Thread(() -> {
            System.out.println("Doing some work..");
            try {
                Thread.sleep(2000);
                COUNT_DOWN_LATCH.countDown();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });
        Thread run3 = new Thread(() -> {
            System.out.println("Doing some work..");
            try {
                Thread.sleep(2000);
                COUNT_DOWN_LATCH.countDown();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });
        Thread run4 = new Thread(() -> {
            System.out.println("Doing some work..");
            try {
                Thread.sleep(2000);
                COUNT_DOWN_LATCH.countDown();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });
        Thread run5 = new Thread(() -> {
            System.out.println("Doing some work..");
            try {
                Thread.sleep(3000);
                COUNT_DOWN_LATCH.countDown();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });
        run1.start();
        run2.start();
        run3.start();
        run4.start();
        run5.start();
        try {
            COUNT_DOWN_LATCH.await();
        } catch (InterruptedException e) {
            //Handle when a thread gets interrupted.
        }
        System.out.println("All tasks have finished..");
    }
}
```

在上述代码中，我们定义了一个起始值为5的 **COUNT_DOWN_LATCH**，意思是在执行正常的流程前需要等待5个线程完成任务。然后，在 main 方法中，我们启动了5个线程，每个都是处理相同的工作并且在工作处理完成后调用 `countDown()` 方法。

> 需要注意的是，我们并不需要担心 **COUNT_DOWN_LATCH** 的同步以及线程安全问题，因为 **CountDownLatch** 类的内部实现已经完成了同步了。

我们调用 `CountDownLatch` 的 `await()` 方法来等到计数器到达0，然后执行正常的代码片段。

这就是关于 `CountDownLatch` 的所有内容，希望对你有所帮助。