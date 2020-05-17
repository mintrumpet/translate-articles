# 翻译 | 你是否有效地管理 GC？

> 原文自国外技术社区dzone，作者为  Piyush Rana，[传送门](https://dzone.com/articles/did-you-manage-your-garbage-efficiently)

> 是时候清理下垃圾了。

![](http://pic.mintrumpet.fun/blog/2019060201.png)

> 1. 虚拟监视器看到的是
>
> 2. 然而，这里有些虚拟监视器看不到的细节

在 JVM 中，如果你说到 java 内存管理，第一件事肯定是想弄懂 java 垃圾回收机制了。在使用基于 JVM 的应用时，当 JVM 自身在帮你管理 GC 时，这事情就变得非常简单。但当你因为 GC 的低效导致面临性能下降的情形时，问题就出现了。

那么，让我们先弄懂什么是 JVM GC 模型，然后我们就能清楚如何控制它并且通过分析 GC 日志来查找应用程序中发生的所有差错情况。

## 什么是自动化垃圾回收？

"自动垃圾回收是这么一个过程，通过查看堆内存，分析识别哪些是正在被使用的对象以及哪些不是，并且除去哪些没在使用的对象。"

### 自动 GC 的步骤

#### 1. 标记

第一步过程叫做标记。这是 GC 去识别哪些在用以及没在用的内存块。

如果系统中所有对象都必须被扫描到，这会是一个十分耗时的过程。

![](http://pic.mintrumpet.fun/blog/2019060203.png)

> 标记
>
> 标记之前 标记之后
>
> 存活对象 未引用对象 内存空间

#### 2. normal deletion

![](http://pic.mintrumpet.fun/blog/2019060205.png)

> normal deletion 之后
>
> 内存分配器存有空闲空间的引用的列表记录，并且当需要被分配时寻找空闲的空间

nromal deletion 会移除未被引用的对象，保留引用对象并且将其指向空闲的内存空间。

#### 3. compaction 压缩

![](http://pic.mintrumpet.fun/blog/2019060206.png)

> normal deletion with compaction 之后
>
> 内存分配器持有空闲空间的起始位置引用，并且继续分配内存

为了进一步提高性能，除了删除未被引用的对象之外，你还可以压缩留下来的引用对象。

通过将引用对象汇集到一块，可以使新的内存分配更加容易和快速。

## 为什么全自动是个问题

这是一个 **Stop the World** 事件，**Stop the World** 事件是指在应用中，当 GC 在运行时，应用程序在这段时间是没法作出相应的。所以，为了程序有效运行，GC 应该消耗最短的时间。如果这步骤花费大量时间，那么 GC 设置就会出现问题。

 在下面的 JVM 内存模型中，它被分成了两个独立的块。在基本级别中，JVM 堆内存被物理地分成了两块 — 新生代和老年代。

![](http://pic.mintrumpet.fun/blog/2019060204.png)

1. 首先，所有新对象会被分配到 eden 区。两个 survivor 区在初始化中是空的。
2. 当 eden 区被填满了，就会触发一次 minor GC。
3. 引用对象会被移动到第一个 survivor 空间中。未被引用的对象在 eden 空间被清空的时候会被删除掉。
4. 在下一个 minor GC，eden 空间会再一次重复相同的工作。未被引用的对象会被删除并且引用对象会被移动到 survivor 空间。然而，在这时，它们是被移动至第二个 survivor 区域（S1）。
5. 随着时间过去。完成 minor GC 之后，当有对象的年限达到一定阈值的时候（在本例中是8），就会将其从新生代转移至老年代当中。
6. 所以，这将近覆盖了新生代的整个过程。最终，major GC 将会在老年代中进行，在这个过程中，将会清理和压缩内存空间。

## 分析 GC 日志的方法

1. VisualVM
2. GC Easy — https://gceasy.io/
3. GC Log Viewer – https://gclogviewer.javaperformancetuning.co.za/#
4. GC Plot Tool for Linux – https://gcplot.com/

这就是目前 GC 基础知识的全部内容了。

