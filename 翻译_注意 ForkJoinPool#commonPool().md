# 翻译 | 注意 ForkJoinPool#commonPool()

> 原文自国外技术社区dzone，作者为 Petr Bouda，[传送门](https://dzone.com/articles/be-aware-of-forkjoinpoolcommonpool)

> 学习更多如何在 java 中处理线程池。

今天让我们关注下 jdk 中隐藏的的功能。通常，我们会使用那些提供基于并行处理的功能的内置构造器或者框架。在大多情况下，我们可以指定我们自己的在并行处理期间使用的线程池，但有时，我们并不想指定我们的线程池并且只是使用当前库中的默认项。每一个库都有它自己的方法来定义默认的线程池。例如，Spring Framework 在大多情况使用的并非纯粹管理线程的线程池，只是为每一个任务创建一个新的线程。然而，这篇文章是介绍在 jdk 中如何处理线程池，要留意了这一定不会无聊。:)

## ForkJoinPool#commonPool 简介

我们先简单介绍一下然后直接看下一些例子。`ForkJoinPool#commonPool()` 是一个静态的线程池，在实际需要时会被懒初始化。两个使用使用 jdk 内置的 `commonPool` 的主要概念：`CompletableFuture` 和 `Parallel Streams`。这两者中有一个细小的差异点：使用 `CompletableFuture`，你能够指定自己的线程池，而不需要使用 `commonPool` 的线程，在 `Parallel Streams` 则不能。

难道我们不应该所有情况都使用 `commonPool` 呢？当我们创建一个额外的线程池的时候，不会产生开销吗？对，我们完可以样做。关于是否使用 `commonPool` 的决策关键在于我们那些传递给线程池的任务的目的。通常有两种类型的任务：计算型和阻塞型。

在计算型任务中，我们创建一个完全避免任何诸如 I/O 操作（数据库调用，同步，线程休眠等等）的阻塞的任务。诀窍在于，不需要在意你的任务运行在哪个线程，让 CPU 保持忙碌并且不要等待任何资源。然后，随意使用 `commonPool` 来执行你的任务。

然而，如果你打算使用 `commonPool` 来处理阻塞任务，那么你就需要考虑下带来的一些影响了。如果你服务器上具有超过三个可用的 CPU 的话，那么你的 `commonPool` 会自动调整为两个线程并且你能通过将线程保持在阻塞状态，来很容易地阻塞你系统中同时使用 `commonPool` 的任何操作。根据经验，我们可以创建我们自己的线程池来阻塞任务，并使系统中的其他部分保持分离和可预测。

## 直接到实例当中

让我们转到文章中更有趣的一部分 — 关于 `commonPool` 中由相同原因形成的隐藏陷阱，即计算 `commonPool` 需要使用多少个线程。这个值由 jvm 根据 CPU 核心数来自动计算确定。

``` java
public class CommonPoolTest {
  public static void main(String[] args) {
    System.out.println("CPU Core: " + Runtime.getRuntime().availableProcessors());
    System.out.println("CommonPool Parallelism: " + ForkJoinPool.commonPool().getParallelism());
    System.out.println("CommonPool Common Parallelism: " + ForkJoinPool.getCommonPoolParallelism());
    long start = System.nanoTime();
    List<CompletableFuture<Void>> futures = IntStream.range(0, 100)
      .mapToObj(i -> CompletableFuture.runAsync(CommonPoolTest::blockingOperation))
      .collect(Collectors.toUnmodifiableList());
    CompletableFuture.allOf(futures.toArray(CompletableFuture[]::new)).join();
    System.out.println("Processed in " + Duration.ofNanos(System.nanoTime() - start).toSeconds() + " sec");
  }
  private static void blockingOperation() {
    try {
      Thread.sleep(1000);
    } catch (InterruptedException e) {
      e.printStackTrace();
    }
  }
}
```

你可以注意到上面的代码有一个非常简单的阻塞调用的实现，执行100次一秒的阻塞调用。让我们看看结果：

``` shell
docker run -it --cpus 4 -v ${PWD}:/app --workdir /app adoptopenjdk/openjdk11 java CommonPoolTest.java
CPU Core: 4
CommonPool Parallelism: 3
CommonPool Common Parallelism: 3
Processed in 34 sec
```

本次使用了4个 CPU 并且34秒运行结束。我们能看到 jvm 自动检索到程序是运行在 docker 容器上并且限制 cpu 为4个和使用3条线程来执行。

``` shell
docker run -it --cpus 2 -v ${PWD}:/app --workdir /app adoptopenjdk/openjdk11 java CommonPoolTest.java
CPU Core: 2
CommonPool Parallelism: 1
CommonPool Common Parallelism: 1
Processed in 1 sec
```

在第二个例子中，我们只使用2个 cpu，我们能注意到 jvm 自动将并行限制为1。但是为什么？在这1秒钟到底发生了什么？

在 `commonPool` 中可以实现这三种模式。

- **并发线程数大于2** — jdk 为 `commonPool` 创建等于 cpu 核数的线程
- **并发线程数等于1** — jdk 为每个提交的任务创建一个新线程
- **并发线程数等于0** — 提交的任务在调用者线程中执行

如果你想重写 jdk 的自动优化（ergonomic behavior）,你可以指定这三个系统属性：

- **java.util.concurrent.ForkJoinPool.common.parallelism**
- **java.util.concurrent.ForkJoinPool.common.threadFactory**
- **java.util.concurrent.ForkJoinPool.common.exceptionHandler** 

## 在 *commonPool* 中自作自受

我发现两个例子，当在程序中使用 `commonPool` 产生错误的时候！

### 当你更改用于 容器/jvm 的资源时，请记得测试你的程序

正如你上面所看到的，我们颠覆了通常的逻辑思维，由于我们高度地阻塞代码，因此决定增加 cpu 数并且得到一个更糟糕的结果。当你有一个程序使用 http 去下载数十个文件，并且你想通过程序的不同部分来加速，最后你会非常惊讶，结果和你预想的完全不同。你使你的程序变得更慢，因为 jdk 决定使用一个真实的线程池而不是采用一个任务一个线程的策略。

### 神器的调用 --cpu-shares（一个潜在的错误）

``` shell
docker run -it --cpu-shares 1023 -v ${PWD}:/app --workdir /app adoptopenjdk/openjdk11 java CommonPoolTest.java
CPU Core: 1
CommonPool Parallelism: 1
CommonPool Common Parallelism: 1
Processed in 1 sec

docker run -it --cpu-shares 1024 -v ${PWD}:/app --workdir /app adoptopenjdk/openjdk11 java CommonPoolTest.java
CPU Core: 8
CommonPool Parallelism: 7
CommonPool Common Parallelism: 7
Processed in 15 sec

docker run -it --cpu-shares 1025 -v ${PWD}:/app --workdir /app adoptopenjdk/openjdk11 java CommonPoolTest.java
CPU Core: 2
CommonPool Parallelism: 1
CommonPool Common Parallelism: 1
Processed in 1 sec
```

**--cpu-shares 1024** 选项打破了 jvm 的 container-awareness 并且展示主机上的 cpu 核数。

这就是全部了。尽情在你的应用中使用 `commonPool`，并且我希望你能在这当中得到一些提示，以减少获得一些有趣或者不好的结果。